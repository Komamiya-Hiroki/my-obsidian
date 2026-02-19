# 遺伝性疾患プラス WordPress.com移行仕様書

## 目次

1. [プロジェクト概要](#1-プロジェクト概要)
2. [WordPress.comサイト設定](#2-wordpresscomサイト設定)
3. [データ移行計画](#3-データ移行計画)
4. [移行後の検証と最適化](#4-移行後の検証と最適化)

---

## 1. プロジェクト概要

### 1.1 移行元と移行先

- **移行元**: Nuxt.js + Directus (https://genetics.qlife.jp/)
- **移行先**: WordPress.com Business プラン
- **ソースコード**: https://github.com/QLife-Inc/genetics.qlife.jp

### 1.2 移行の目的

- WordPress.comによるコンテンツ管理の効率化
- 保守性の向上
- 記事更新・追加の容易化
- 将来的な機能拡張の基盤構築

### 1.3 プロジェクトスケジュール

1. **WordPress.comサイト設定**: 1週間
2. **データ移行**: 2週間
3. **検証とQA**: 1週間
4. **本番移行**: 1日

### 1.4 主要ステークホルダー

- プロジェクト管理者
- コンテンツ編集チーム
- 開発チーム
- WordPress.comサポート

---

## 2. WordPress.comサイト設定

### 2.1 WordPress.com Businessプラン概要

#### 2.1.1 主要機能と特徴

- **カスタマイズ機能**
  - カスタムプラグインのインストール（50,000以上のプラグインから選択可能）
  - サードパーティ製WordPressテーマのアップロード
  - JavaScriptやiframeなどのカスタムコード追加
  - 高度なSEOツールとプラグインサポート
  - 50GBのストレージ（画像やドキュメント用）

- **開発者向け機能**
  - SFTPとSSHアクセス
  - MySQLデータベースアクセス
  - PHPバージョン切り替え
  - GitHubデプロイメント
  - REST API連携
  - ステージングサイト環境

- **パフォーマンスとセキュリティ**
  - マルチデータセンターサポート
  - リアルタイムサイトレプリケーション
  - グローバルコンテンツキャッシング
  - Jetpack Scanによるセキュリティ監視
  - 日次自動バックアップ
  - DDoSおよびマルウェア保護

#### 2.1.2 サイト基本設定

- **サイト名**: 遺伝性疾患プラス
- **言語設定**: 日本語
- **タイムゾーン**: 東京 (UTC+9:00)
- **日付形式**: YYYY年MM月DD日
- **パーマリンク設定**: カスタム構造 (現URLパターンを維持)
- **ドメイン設定**: genetics.qlife.jp
- **SSL証明書**: 自動発行（Let's Encrypt）

### 2.2 必要なプラグインと設定

#### 2.2.1 コア機能プラグイン

| プラグイン名 | 用途 | 主な設定項目 |
|------------|------|------------|
| **Custom Post Type UI** | カスタム投稿タイプの作成 | カスタム投稿タイプ名、スラッグ、表示名、アーカイブ設定 |
| **Advanced Custom Fields** | カスタムフィールドの設定 | フィールドグループ、フィールドタイプ、表示条件 |
| **Redirection** | URL構造の維持とリダイレクト設定 | リダイレクトルール、404監視、インポート/エクスポート |
| **WP Glossary** | 用語集機能 | 用語自動リンク、表示スタイル、除外設定 |

#### 2.2.2 拡張機能プラグイン

| プラグイン名 | 用途 | 主な設定項目 |
|------------|------|------------|
| **SearchWP** | 高度な検索機能 | 検索対象、重み付け、カスタム投稿タイプ検索設定 |
| **EWWW Image Optimizer** | 画像最適化 | 圧縮レベル、リサイズ設定、WebP変換 |
| **Yoast SEO** | SEO対策 | メタタイトル、メタ説明、構造化データ、XMLサイトマップ |
| **WP All Import** | データインポート | インポートテンプレート、スケジュール、フィールドマッピング |
| **JWT Authentication** | REST API認証用 | 認証設定、トークン有効期限、許可IPアドレス |

#### 2.2.3 外部連携プラグイン

| プラグイン名 | 用途 | 主な設定項目 |
|------------|------|------------|
| **Google Site Kit** | Googleサービス連携 | Analytics、Search Console、AdSense、PageSpeed |
| **LINE連携プラグイン** | LINE会員限定コンテンツ | チャネルID、シークレット、コールバックURL |
| **AddToAny Share Buttons** | SNSシェア機能 | 表示位置、デザイン、対象SNS設定 |

### 2.3 カスタム投稿タイプ設定

#### 2.3.1 Custom Post Type UI設定

WordPress.com BusinessプランでCustom Post Type UIプラグインを使用して、以下のカスタム投稿タイプを設定します。

| 投稿タイプ名 | スラッグ | 表示名 | 階層 | 特記事項 |
|------------|---------|--------|------|---------|
| disease | diseases | 遺伝性疾患解説 | なし | 検索インデックス有効 |
| disease_article | disease_articles | 遺伝性疾患関連記事 | なし | 親子関係あり |
| feature | features | 特集 | なし | 検索インデックス有効 |
| feature_article | feature_articles | 特集記事 | feature | 階層型URL |
| glossary | glossary | 用語集 | なし | 検索インデックス有効 |
| news | news | ニュース | なし | 日付ベースURL |
| interview | interviews | 専門家インタビュー | なし | 目次機能必要 |
| other_article | other_articles | その他の記事 | なし | カテゴリ分類 |
| tutorial | tutorials | 学習 | なし | 検索インデックス有効 |
| tutorial_article | tutorial_articles | 学習記事 | tutorial | 階層型URL |
| limited_article | limited_release | 限定公開記事 | なし | アクセス制御必要 |
| supporter | supporters | 支援者 | なし | 検索インデックス無効 |
| interviewee | interviewees | インタビュー対象者 | なし | 検索インデックス無効 |

#### 2.3.2 各投稿タイプの詳細設定

**遺伝性疾患解説 (disease)**
- **公開ラベル**: 「疾患を公開」
- **メニュー位置**: 5（ダッシュボード上部に配置）
- **サポート機能**: タイトル、エディタ、アイキャッチ画像、抜粋、カスタムフィールド
- **リビジョン追跡**: 有効
- **REST API表示**: 有効
- **アーカイブ設定**: カスタムスラッグ「diseases」

**特集記事 (feature_article)**
- **親投稿タイプ**: feature（特集）
- **階層型**: 有効
- **メニュー表示位置**: 親投稿タイプの下に配置
- **一覧表示カラム**: タイトル、親特集、公開日、アイキャッチ

**学習記事 (tutorial_article)**
- **親投稿タイプ**: tutorial（学習）
- **階層型**: 有効
- **メニュー表示位置**: 親投稿タイプの下に配置
- **一覧表示カラム**: タイトル、親学習、公開日、アイキャッチ

**ニュース (news)**
- **一覧表示カラム**: タイトル、公開日、タグ、アイキャッチ
- **日付アーカイブ**: 有効
- **投稿数表示**: 1ページあたり10件

### 2.4 カスタムフィールド設定

#### 2.4.1 Advanced Custom Fields設定方法

WordPress.com BusinessプランでAdvanced Custom Fieldsプラグインを使用して、以下のカスタムフィールドグループを設定します。

**フィールドグループ作成手順**：
1. WordPress管理画面で「カスタムフィールド」→「新規追加」をクリック
2. フィールドグループ名を設定（例：「遺伝性疾患解説フィールド」）
3. 必要なフィールドを追加し、各フィールドのプロパティを設定
4. 表示条件を設定（特定の投稿タイプでのみ表示など）
5. グループを保存

#### 2.4.2 投稿タイプ別カスタムフィールド設定

**遺伝性疾患解説 (disease)**

| フィールド名 | フィールドタイプ | 必須 | オプション設定 |
|-------------|-----------------|------|---------------|
| first_letter | テキスト | はい | 最大長：1文字、プレースホルダ：「頭文字（A〜Z）」 |
| english_name | テキスト | はい | 最大長：200文字、プレースホルダ：「英語表記」 |
| kana_name | テキスト | いいえ | 最大長：60文字、プレースホルダ：「かな表記」 |
| factors | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |
| diagnosis | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |
| treatments | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |
| locations | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |
| patients | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |
| miscellaneous | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |
| details | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |
| references | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |
| hospital_searches | ウィジェットエリア | いいえ | エディタツールバー：フル、メディア挿入許可 |

**遺伝性疾患関連記事 (disease_article)**

| フィールド名 | フィールドタイプ | 必須 | オプション設定 |
|-------------|-----------------|------|---------------|
| disease | 投稿オブジェクト | はい | 投稿タイプ：disease、戻り値：投稿ID |
| category | セレクト | はい | 選択肢：clinical-trials/interviews/patients、複数選択：いいえ |

**特集記事 (feature_article)**

| フィールド名 | フィールドタイプ | 必須 | オプション設定 |
|-------------|-----------------|------|---------------|
| youtube_url | URL | いいえ | プレースホルダ：「https://youtu.be/XXXX」 |

**専門家インタビュー (interview)**

| フィールド名 | フィールドタイプ | 必須 | オプション設定 |
|-------------|-----------------|------|---------------|
| supporter | 投稿オブジェクト | はい | 投稿タイプ：supporter、戻り値：投稿ID |
| toc_enable | 真偽値 | いいえ | メッセージ：「目次を表示する」、デフォルト：オン |

**インタビュー対象者 (interviewee)**

| フィールド名 | フィールドタイプ | 必須 | オプション設定 |
|-------------|-----------------|------|---------------|
| position | テキスト | いいえ | 最大長：100文字、プレースホルダ：「役職・肩書き」 |
| disease_article | 投稿オブジェクト | いいえ | 投稿タイプ：disease_article、戻り値：投稿ID |
| specialist_interview | 投稿オブジェクト | いいえ | 投稿タイプ：interview、戻り値：投稿ID |
| other_article | 投稿オブジェクト | いいえ | 投稿タイプ：other_article、戻り値：投稿ID |

**ニュース記事 (news)**

| フィールド名 | フィールドタイプ | 必須 | オプション設定 |
|-------------|-----------------|------|---------------|
| points | ウィジェットエリア | いいえ | エディタツールバー：シンプル、メディア挿入許可 |

### 2.5 パーマリンク・URL構造設定

#### 2.5.1 WordPress.comでのパーマリンク設定方法

1. **基本設定**
   - 管理画面の「設定」→「パーマリンク」へアクセス
   - 「カスタム構造」を選択
   - 基本構造として `/%postname%/` を設定

2. **Custom Post Type UIでの設定**
   - カスタム投稿タイプ編集画面で「リライトスラッグ」と「with_front」オプションを設定
   - has_archive設定をONにして、アーカイブページURLを有効化

3. **Redirectionプラグインの設定**
   - 旧サイトURL構造を保持するリダイレクトルールを設定
   - 404エラー監視を有効にして、見つからないURLを自動検出

#### 2.5.2 投稿タイプ別パーマリンク設定

| 投稿タイプ | パーマリンク構造 | 設定方法 | 具体的なコード例 |
|-----------|----------------|----------|----------------|
| disease | `/diseases/%postname%/` | カスタムベース + スラッグ | `'rewrite' => array('slug' => 'diseases', 'with_front' => false)` |
| disease_article | `/diseases/%disease%/%category%/%postname%/` | カスタム構造（関連疾患+カテゴリ） | 関連フックとカスタム関数で実装 |
| feature | `/features/%postname%/` | カスタムベース + スラッグ | `'rewrite' => array('slug' => 'features', 'with_front' => false)` |
| feature_article | `/features/%feature%/%postname%/` | カスタム構造（親特集） | 親子関係のrewrite_rulesで実装 |
| glossary | `/glossary/%postname%/` | カスタムベース + スラッグ | `'rewrite' => array('slug' => 'glossary', 'with_front' => false)` |
| news | `/news/%postname%/` | カスタムベース + スラッグ | `'rewrite' => array('slug' => 'news', 'with_front' => false)` |
| interview | `/interviews/%postname%/` | カスタムベース + スラッグ | `'rewrite' => array('slug' => 'interviews', 'with_front' => false)` |
| tutorial | `/tutorials/%postname%/` | カスタムベース + スラッグ | `'rewrite' => array('slug' => 'tutorials', 'with_front' => false)` |
| tutorial_article | `/tutorials/%tutorial%/%postname%/` | カスタム構造（親学習） | 親子関係のrewrite_rulesで実装 |
| limited_article | `/limited-release/%postname%/` | カスタムベース + スラッグ | `'rewrite' => array('slug' => 'limited-release', 'with_front' => false)` |
| supporter | `/supporters/%postname%/` | カスタムベース + スラッグ | `'rewrite' => array('slug' => 'supporters', 'with_front' => false)` |

#### 2.5.3 特殊なパーマリンク構造の実装

**疾患関連記事の階層URL実装**
```php
// functions.phpに追加
add_filter('post_type_link', 'disease_article_permalink', 10, 2);
function disease_article_permalink($permalink, $post) {
  if ($post->post_type !== 'disease_article') {
    return $permalink;
  }
  
  $disease_id = get_post_meta($post->ID, 'disease', true);
  $category = get_post_meta($post->ID, 'category', true);
  
  if (!$disease_id || !$category) {
    return $permalink;
  }
  
  $disease_post = get_post($disease_id);
  if (!$disease_post) {
    return $permalink;
  }
  
  $permalink = str_replace('%disease%', $disease_post->post_name, $permalink);
  $permalink = str_replace('%category%', $category, $permalink);
  
  return $permalink;
}
```

### 2.6 分類タクソノミー設定

#### 2.6.1 標準タグ設定

**標準タグの設定**
- すべてのカスタム投稿タイプで標準タグ（post_tag）を使用可能に設定
- Custom Post Type UIの投稿タイプ設定で「標準タグを有効化」をON

**タグ表示設定**
```php
// functions.phpに追加
add_action('pre_get_posts', 'custom_tag_archives');
function custom_tag_archives($query) {
  if ($query->is_tag() && $query->is_main_query()) {
    // 標準タグアーカイブに全カスタム投稿タイプを含める
    $post_types = array('post', 'disease', 'disease_article', 'news', 'interview', 'feature', 'feature_article', 'tutorial', 'tutorial_article', 'other_article');
    $query->set('post_type', $post_types);
  }
}
```

#### 2.6.2 タグクラウドウィジェット設定

**タグクラウド表示カスタマイズ**
```php
// functions.phpに追加
add_filter('widget_tag_cloud_args', 'custom_tag_cloud_args');
function custom_tag_cloud_args($args) {
  // 最大表示数と並び順をカスタマイズ
  $args['number'] = 30;
  $args['orderby'] = 'count';
  $args['order'] = 'DESC';
  
  return $args;
}
```

---

## 3. データ移行計画

### 3.1 移行前の準備

#### 3.1.1 移行環境準備

- WordPress.com Business プランのアクティベーション
- 必要なプラグインのインストールと設定
- カスタム投稿タイプとフィールドの設定
- バックアップ環境の構築

#### 3.1.2 バックアップの取得

- Directus データベースの完全バックアップ
- メディアファイル（S3バケット）のバックアップ
- WordPress.com 初期設定のエクスポート

### 3.2 移行スクリプト開発

#### 3.2.1 API連携設定

**Directus API設定**
```javascript
// Directus API設定例（Node.js）
const axios = require('axios');

const directusConfig = {
  baseURL: 'https://api.genetics.qlife.jp/',
  headers: {
    'Authorization': 'Bearer YOUR_DIRECTUS_API_TOKEN',
    'Content-Type': 'application/json'
  },
  timeout: 30000 // 30秒タイムアウト
};

const directusAPI = axios.create(directusConfig);

// レートリミット対策としての遅延関数
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// APIコール例
async function fetchDiseases() {
  try {
    const response = await directusAPI.get('/items/diseases?limit=100&status=published');
    return response.data.data;
  } catch (error) {
    console.error('Directus API error:', error);
    throw error;
  }
}
```

**WordPress.com REST API設定**
```javascript
// WordPress.com REST API設定例（Node.js）
const wpConfig = {
  baseURL: 'https://public-api.wordpress.com/wp/v2/sites/genetics.qlife.jp',
  headers: {
    'Authorization': 'Bearer YOUR_WP_ACCESS_TOKEN',
    'Content-Type': 'application/json'
  },
  timeout: 30000 // 30秒タイムアウト
};

const wpAPI = axios.create(wpConfig);

// OAuth2認証関数
async function getWPAccessToken() {
  try {
    const tokenResponse = await axios.post('https://public-api.wordpress.com/oauth2/token', {
      client_id: 'YOUR_CLIENT_ID',
      client_secret: 'YOUR_CLIENT_SECRET',
      grant_type: 'password',
      username: 'YOUR_USERNAME',
      password: 'YOUR_PASSWORD'
    });
    
    return tokenResponse.data.access_token;
  } catch (error) {
    console.error('WP OAuth error:', error);
    throw error;
  }
}

// 投稿作成例
async function createWPPost(postData) {
  try {
    const response = await wpAPI.post('/posts', postData);
    return response.data;
  } catch (error) {
    console.error('WordPress API error:', error);
    throw error;
  }
}
```

#### 3.2.2 移行スクリプトの機能実装

**タグ・分類データの移行**
```javascript
// タグデータ移行関数
async function migrateTags() {
  // 1. Directusからタグデータ取得
  const tags = await directusAPI.get('/items/tags?limit=500');
  
  // 2. 重複チェックと正規化
  const normalizedTags = tags.data.data.map(tag => ({
    name: tag.name.trim(),
    slug: generateSlug(tag.name),
    directus_id: tag.id // 後で関連付けに使用
  }));
  
  // 3. WordPress.comにタグを登録
  const tagMapping = {}; // Directus ID → WordPress ID のマッピング
  
  for (const tag of normalizedTags) {
    await delay(500); // レートリミット対策
    
    try {
      // 既存タグの確認
      const existingTag = await wpAPI.get(`/tags?slug=${tag.slug}`);
      
      if (existingTag.data.length > 0) {
        // 既存タグが見つかった場合
        tagMapping[tag.directus_id] = existingTag.data[0].id;
        console.log(`Tag already exists: ${tag.name}`);
      } else {
        // 新規タグ作成
        const newTag = await wpAPI.post('/tags', {
          name: tag.name,
          slug: tag.slug
        });
        
        tagMapping[tag.directus_id] = newTag.data.id;
        console.log(`Created tag: ${tag.name}`);
      }
    } catch (error) {
      console.error(`Error processing tag ${tag.name}:`, error);
    }
  }
  
  return tagMapping;
}

// スラッグ生成ヘルパー関数
function generateSlug(name) {
  return name
    .toLowerCase()
    .replace(/[^a-z0-9\s]/g, '') // 英数字とスペースのみ残す
    .replace(/\s+/g, '-') // スペースをハイフンに変換
    .replace(/^-+|-+$/g, ''); // 先頭と末尾のハイフンを削除
}
```

**メディア移行関数**
```javascript
// S3からWordPress.comへのメディア移行
async function migrateMedia() {
  // 1. Directusからメディア情報取得
  const files = await directusAPI.get('/files?limit=100');
  
  // 2. S3から画像をダウンロードしてWordPress.comにアップロード
  const mediaMapping = {}; // Directus ID → WordPress ID のマッピング
  
  for (const file of files.data.data) {
    await delay(1000); // レートリミット対策
    
    try {
      // S3からファイルをダウンロード
      const fileResponse = await axios.get(file.url, { responseType: 'arraybuffer' });
      const fileBuffer = Buffer.from(fileResponse.data, 'binary');
      
      // WordPress.comにアップロード
      const formData = new FormData();
      formData.append('file', fileBuffer, file.filename_download);
      
      const wpMedia = await wpAPI.post('/media', formData, {
        headers: {
          'Content-Disposition': `attachment; filename="${file.filename_download}"`
        }
      });
      
      mediaMapping[file.id] = wpMedia.data.id;
      console.log(`Uploaded media: ${file.filename_download}`);
    } catch (error) {
      console.error(`Error processing media ${file.filename_download}:`, error);
    }
  }
  
  return mediaMapping;
}
```

**コンテンツ移行のマスター関数**
```javascript
async function migrateContent() {
  // 1. 基本データ（タグ、メディア）の移行
  const tagMapping = await migrateTags();
  const mediaMapping = await migrateMedia();
  
  // 2. 支援者データの移行
  const supporterMapping = await migrateSupporters(mediaMapping);
  
  // 3. インタビュー対象者データの移行
  const intervieweeMapping = await migrateInterviewees(mediaMapping);
  
  // 4. 主要コンテンツの移行
  await migrateDiseases(mediaMapping, tagMapping);
  await migrateGlossary();
  await migrateNews(mediaMapping, tagMapping);
  await migrateInterviews(mediaMapping, tagMapping, supporterMapping);
  
  // 5. 関連コンテンツの移行
  // ...
  
  console.log('Migration completed successfully!');
}
```

### 3.3 フェーズ別移行計画

#### 3.3.1 第1フェーズ: 基盤データ移行

1. **タグデータ**
   - 343件のタグデータをWordPress.comに移行
   - 重複チェックと正規化

2. **支援者データ**
   - 62件の支援者情報を`supporter`カスタム投稿タイプに移行
   - プロフィール画像の移行

3. **インタビュー対象者データ**
   - 102件のインタビュー対象者情報を`interviewee`カスタム投稿タイプに移行
   - プロフィール画像の移行

#### 3.3.2 第2フェーズ: メインコンテンツ移行

4. **遺伝性疾患解説**
   - 80件の疾患データを`disease`カスタム投稿タイプに移行
   - カスタムフィールドの正確な移行確認
   - 疾患タグ関連付けの再構築

5. **用語集**
   - 379件の用語データを`glossary`カスタム投稿タイプに移行
   - 検索用インデックスの構築

6. **ニュース**
   - 401件のニュース記事を`news`カスタム投稿タイプに移行
   - タグ関連付けの再構築
   - 日付ベースのパーマリンク設定確認

7. **専門家インタビュー**
   - 38件のインタビューを`interview`カスタム投稿タイプに移行
   - 支援者との関連付け再構築
   - 目次機能の実装確認

#### 3.3.3 第3フェーズ: 関連コンテンツ移行

8. **遺伝性疾患関連記事**
   - 152件の関連記事を`disease_article`カスタム投稿タイプに移行
   - 親疾患との関連付け再構築

9. **その他の記事**
   - 57件のその他の記事を`other_article`カスタム投稿タイプに移行
   - カテゴリ分類の確認

10. **特集・特集記事**
    - 13件の特集を`feature`カスタム投稿タイプに移行
    - 56件の特集記事を`feature_article`カスタム投稿タイプに移行
    - 親子関係の再構築

11. **学習・学習記事**
    - 14件の学習を`tutorial`カスタム投稿タイプに移行
    - 107件の学習記事を`tutorial_article`カスタム投稿タイプに移行
    - 親子関係の再構築

#### 3.3.4 第4フェーズ: 特殊コンテンツ移行

12. **限定公開記事**
    - 19件の限定公開記事を`limited_article`カスタム投稿タイプに移行
    - LINEログイン連携による限定アクセス制御の設定

13. **外部連携の再設定**
    - Googleカスタム検索の再設定
    - LINE会員連携の再構築
    - SNSシェアボタンの設定

### 3.4 データクリーニング

- HTML整形・正規化
- 画像参照パスの修正
- 内部リンクの更新
- 日本語URL・特殊文字のエンコード確認
- 重複コンテンツの検出と統合

---

## 4. 移行後の検証と最適化

### 4.1 管理画面設定と最適化

#### 4.1.1 WordPress.comダッシュボード設定

**サイト設定の最終確認**
```
WordPress.com管理画面 > 設定 > 一般
```
- サイトタイトル: 遺伝性疾患プラス
- キャッチフレーズ: 遺伝性疾患について正しく理解するための情報サイト
- 言語: 日本語
- タイムゾーン: UTC+9:00（東京）

**プライバシー設定**
```
WordPress.com管理画面 > 設定 > プライバシー
```
- 検索エンジンの可視性: オン
- クッキーの設定: GDPR対応
- プライバシーポリシーページの設定

**キャッシュ設定**
```
WordPress.com管理画面 > Jetpack > パフォーマンス
```
- サイト高速化: オン
- 画像最適化: オン
- 遅延読み込み: オン
- 動画の遅延読み込み: オン

#### 4.1.2 プラグイン設定最適化

**Yoast SEO設定**
```
WordPress.com管理画面 > SEO > ダッシュボード
```
- サイトマップ生成: 有効
- ソーシャルメディア連携: 設定
- カスタム投稿タイプのSEO設定: 有効
- メタデータテンプレート設定

**画像最適化設定**
```
WordPress.com管理画面 > EWWW画像最適化 > 設定
```
- WebP変換: 有効
- メディアライブラリの一括最適化を実行
- 画像サイズの自動リサイズ: 有効（最大幅2000px）

### 4.2 検証計画

#### 4.2.1 機能検証手順

**1. 全ページの表示確認**
```bash
#!/bin/bash
# ページ表示テストスクリプト

# テスト対象URLリストファイル
URL_LIST="test_urls.txt"

# 結果出力ファイル
RESULTS="page_test_results.csv"

echo "URL,Status,Load Time (s),Mobile Responsive" > $RESULTS

while read url; do
  # ステータスコード取得
  status=$(curl -o /dev/null -s -w "%{http_code}\n" "$url")
  
  # 読み込み時間計測
  load_time=$(curl -o /dev/null -s -w "%{time_total}\n" "$url")
  
  # モバイルレスポンシブチェック（簡易版）
  mobile_check=$(curl -A "Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X)" -s "$url" | grep -c "viewport")
  
  if [ "$mobile_check" -gt 0 ]; then
    responsive="Yes"
  else
    responsive="No"
  fi
  
  echo "$url,$status,$load_time,$responsive" >> $RESULTS
  
  echo "Tested: $url - Status: $status"
  
  # API制限対策の遅延
  sleep 1
done < $URL_LIST

echo "Test completed. Results saved to $RESULTS"
```

**2. 検索機能テストケース**

| 検索パターン | テスト内容 | 期待結果 |
|------------|---------|----------|
| 疾患名検索 | 「レット症候群」を検索 | 該当疾患と関連記事が表示される |
| 症状検索 | 「筋力低下」を検索 | 関連する疾患や記事が表示される |
| 英語名検索 | 「Usher Syndrome」を検索 | 該当疾患が表示される |
| 医療用語検索 | 「遺伝子変異」を検索 | 用語集と関連記事が表示される |
| カタカナ表記 | 「アッシャー症候群」を検索 | 「Usher Syndrome」の結果が表示される |

**3. 会員機能検証チェックリスト**
- LINE連携認証フロー動作確認
- 限定コンテンツの表示制御確認
- 非会員アクセス時の挙動確認
- セッション持続性確認
- ログアウト機能確認

#### 4.2.2 パフォーマンステスト

**Google PageSpeed Insightsテスト対象**
- トップページ
- 疾患一覧ページ
- 疾患詳細ページ
- ニュース一覧ページ
- 特集ページ

**パフォーマンス最適化設定**
```php
// functions.phpに追加

// 不要なスタイルとスクリプトの除去
function optimize_script_loading() {
  // WordPressのemoji関連リソースを無効化
  remove_action('wp_head', 'print_emoji_detection_script', 7);
  remove_action('admin_print_scripts', 'print_emoji_detection_script');
  remove_action('wp_print_styles', 'print_emoji_styles');
  remove_action('admin_print_styles', 'print_emoji_styles');
  
  // 不要なjQueryの依存を削除（必要に応じて）
  if (!is_admin()) {
    wp_deregister_script('jquery');
    wp_register_script('jquery', includes_url('/js/jquery/jquery.min.js'), false, NULL, true);
  }
  
  // CSSとJSの遅延読み込み（クリティカルパスの最適化）
  add_filter('style_loader_tag', 'add_defer_attribute', 10, 2);
  add_filter('script_loader_tag', 'add_defer_attribute', 10, 2);
}
add_action('init', 'optimize_script_loading');

// 遅延読み込み属性の追加
function add_defer_attribute($tag, $handle) {
  // クリティカルでないリソースに遅延読み込み属性を追加
  if (strpos($handle, 'critical') === false) {
    if (is_style_tag($tag)) {
      return str_replace(' rel=', ' rel="preload" as="style" onload="this.rel=\'stylesheet\'" ', $tag);
    } else if (is_script_tag($tag)) {
      return str_replace(' src=', ' defer src=', $tag);
    }
  }
  return $tag;
}
```

#### 4.2.3 SEO検証項目

**メタデータ検証**
```html
<!-- 各投稿タイプのメタタグテンプレート例 -->

<!-- 疾患解説ページ -->
<title>%disease_name% (遺伝性疾患) | 遺伝性疾患プラス</title>
<meta name="description" content="%disease_name%（%english_name%）は、%description_excerpt%。症状や治療法、原因について解説。">

<!-- ニュースページ -->
<title>%title% | 遺伝性疾患ニュース | 遺伝性疾患プラス</title>
<meta name="description" content="%excerpt%">

<!-- 特集ページ -->
<title>%title% | 特集 | 遺伝性疾患プラス</title>
<meta name="description" content="%description_excerpt%">
```

**構造化データ検証**

各コンテンツタイプに適したJSON-LDスキーマを実装：
- 疾患ページ: `MedicalCondition`スキーマ
- ニュース記事: `NewsArticle`スキーマ 
- 専門家インタビュー: `Interview`スキーマ

Google構造化データテストツールで検証

### 4.3 最終調整

- デザイン微調整
- テーマのカスタマイズ
- 404エラーページの設定
- リダイレクトルールの最終確認
- 編集者向けマニュアルの作成

### 4.4 本番移行

1. 最終バックアップの取得
2. DNS切り替え準備
3. WordPress.comサイトの最終確認
4. DNS設定変更（genetics.qlife.jp を WordPress.com に向ける）
5. 伝播確認とモニタリング
6. 移行完了宣言

### 4.5 移行後サポート計画

- 初期監視期間（2週間）の設定
- 編集者向けトレーニング実施
- 定期バックアップの設定
- WordPress.com 更新プロセスの確立