
このドキュメントは、Docker 環境での開発時に発生した問題と、その解決方法をまとめたものです。

## 目次

- [発生した問題](#発生した問題)
- [問題の原因](#問題の原因)
- [解決方法](#解決方法)
- [推奨される開発フロー](#推奨される開発フロー)
- [トラブルシューティング](#トラブルシューティング)

---

## 発生した問題

### 症状

Docker 起動時に以下のエラーが発生：

```
Error: Cannot find module @rollup/rollup-linux-arm64-musl
または
Error: Cannot find module @rollup/rollup-linux-x64-gnu
```

開発サーバーが起動せず、`http://localhost:4321/` にアクセスできない。

### 影響範囲

- Docker環境で開発するユーザー
- **プラットフォーム固有のネイティブモジュールを使用するプロジェクト**
  - **ビルドツール**: Rollup, esbuild, SWC, Turbopack
  - **画像処理**: sharp, canvas
  - **暗号化**: bcrypt
  - **ファイル監視**: fsevents（Mac）
  - など、多くのモダンなフロントエンド/フルスタックプロジェクトで使用されるパッケージ

---

## 問題の原因

### 1. package-lock.json の環境依存

**問題の本質：**

- Mac 環境で`npm install`を実行すると、Mac 用のモジュール情報しか package-lock.json に記録されない
- Docker 内（Linux 環境）で`npm ci`を実行すると、Linux 用のモジュール情報が不足してエラーになる
- **この問題は、Rollup以外にもesbuild、sharp、bcryptなど多くのパッケージで発生する**

**なぜこれらのパッケージで起きるのか：**

- パフォーマンスのため、Rust/C++などで書かれたネイティブコードを使用
- OS/CPUアーキテクチャごとに異なるバイナリが必要
- npmは実行環境に応じて適切なバイナリを選択してインストール
- Mac環境でインストールすると、Mac用のバイナリ情報のみがpackage-lock.jsonに記録される

**具体例：**

```json
// Mac環境で生成されたpackage-lock.json
{
  "optionalDependencies": {
    "@rollup/rollup-darwin-arm64": "4.54.0", // Mac用のみ
    "@rollup/rollup-darwin-x64": "4.54.0" // Mac用のみ
    // Linux用の情報が欠けている ❌
  }
}
```

### 2. オプショナル依存関係の特性

- Rollup はプラットフォームごとに異なるネイティブモジュールを使用
- これらは`optionalDependencies`として定義されている
- インストール失敗してもエラーが出ないため、問題に気づきにくい

---

## 解決方法

### 推奨される解決策

**package-lock.json を Linux/Docker 環境で再生成**することで解決できます。

#### 実施する変更

1. **package-lock.json の再生成**

   - Docker 環境で`npm install`を実行
   - すべてのプラットフォーム用のモジュール情報を含む package-lock.json を生成

2. **依存パッケージの更新（オプション）**

   - 問題のあるパッケージ（Rollup など）を最新版に更新
   - バグ修正が含まれている可能性がある

3. **Node.js バージョンの固定（推奨）**
   - `.node-version`ファイルを追加
   - チーム全体で同じバージョンを使用

#### 期待される結果

```json
// 正しく生成されたpackage-lock.json
{
  "optionalDependencies": {
    "@rollup/rollup-linux-x64-musl": "x.x.x", // Linux Alpine用 ✅
    "@rollup/rollup-linux-x64-gnu": "x.x.x", // Linux Debian用 ✅
    "@rollup/rollup-darwin-arm64": "x.x.x", // Mac ARM用 ✅
    "@rollup/rollup-darwin-x64": "x.x.x" // Mac Intel用 ✅
  }
}
```

---

## 推奨される開発フロー

### ✅ パッケージの追加・更新（推奨方法）

新しいパッケージを追加する場合は、**必ず Docker 環境で実行**してください：

```bash
# docker-compose run を使う（推奨）
docker-compose run <service-name> npm install <package-name>

# 例: astro-siteというサービス名の場合
docker-compose run astro-site npm install <package-name>
```

### ❌ 避けるべきパターン

```bash
# ローカル（Mac）でnpm install → Docker環境で動かなくなる可能性
cd <project-directory>
npm install <package-name>  # ❌ 非推奨
git add package-lock.json
git commit
```

### 開発シナリオ別の推奨フロー

#### シナリオ 1: 新しいパッケージを追加

```bash
# 1. package.jsonを編集（手動またはnpm install経由）
vim package.json

# 2. Docker環境でインストール
docker-compose run <service-name> npm install

# 3. 動作確認
docker-compose up
# アプリケーションにアクセスして確認

# 4. コミット
git add package.json package-lock.json
git commit -m "chore: add <package-name>"
```

#### シナリオ 2: 既存の依存関係をインストール

```bash
# ローカルでもDockerでもOK
npm install

# または
docker-compose run <service-name> npm install
```

#### シナリオ 3: 依存関係を更新

```bash
# Docker環境で更新（推奨）
docker-compose run <service-name> npm update

# または特定のパッケージのみ
docker-compose run <service-name> npm install <package-name>@latest
```

---

## トラブルシューティング

### Q1: ローカルで npm install してしまった場合

```bash
# 1. package-lock.jsonを削除
rm package-lock.json

# 2. Docker環境で再生成
docker-compose run <service-name> npm install

# 3. 動作確認
docker-compose up
```

### Q2: Docker 起動時に rollup エラーが出る場合

```bash
# 完全クリーンアップして再ビルド
docker-compose down --volumes --rmi all
docker-compose up --build

# それでもダメな場合
docker-compose run <service-name> sh -c "rm -rf node_modules package-lock.json && npm install"
docker-compose up
```

### Q3: ビルドが遅い / キャッシュをクリアしたい

```bash
# Dockerイメージとボリュームを完全削除
docker-compose down --volumes --rmi all

# 再ビルド
make start
```

### Q4: package.json と package-lock.json が不整合

```bash
# package-lock.jsonを削除して再生成
docker-compose run <service-name> sh -c "rm package-lock.json && npm install"
```

---

## チーム開発のベストプラクティス

### ルール 1: package-lock.json は Docker 環境で生成

- 新しいパッケージを追加する際は、Docker 環境で npm install を実行
- これにより、すべてのプラットフォームで動作する package-lock.json が生成される

### ルール 2: Pull Request 前の動作確認

```bash
# PRを作成する前に、Docker環境で動作確認
docker-compose down --volumes
docker-compose up --build
# アプリケーションにアクセスして動作確認
```

### ルール 3: package-lock.json の変更は慎重に

- package-lock.json が変更された場合は、必ず Docker 環境で動作確認
- 不用意にローカルで npm install しない

### ルール 4: Node.js バージョンの統一

```bash
# プロジェクトのNode.jsバージョンを確認
cat .node-version
# または
cat .nvmrc

# nvm使用時
nvm use

# nodenv使用時
nodenv install
```

---

## 参考情報

### 典型的な技術スタック

- Node.js（プロジェクトによって異なる）
- npm/yarn/pnpm
- Rollup などのビルドツール（プラットフォーム固有のネイティブモジュールを使用）
- Docker（Alpine Linux または Debian 系）

### 関連ドキュメント

- [npm optional dependencies](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#optionaldependencies)
- [Rollup platform-specific binaries](https://rollupjs.org/introduction/#installation)
- [Docker Compose volumes](https://docs.docker.com/compose/compose-file/07-volumes/)

### よくある誤解

#### ❌ 「Docker 環境だけの問題」

→ **実際は**: package-lock.json の管理方法の問題

#### ❌ 「npm にバグがある」

→ **実際は**: オプショナル依存関係の仕様通りの動作

#### ❌ 「Node.js のバージョンが原因」

→ **実際は**: package-lock.json が環境に依存して生成されることが原因

---

## 更新履歴

- 2026-02-18: 初版作成（Docker 起動問題の解決方法を記録）
