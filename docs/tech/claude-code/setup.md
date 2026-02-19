# Claude Code + aws-vault セットアップガイド

## 問題点

複数のプロジェクトで Claude Code を使用する際、毎回 `aws-vault exec dev --json --duration=16h` でトークンを取得し、各プロジェクトごとに `setup-claude-code` コマンドで認証情報を入力する必要があり、非効率的でした。

## 解決方法

**1日1回の認証で、すべてのプロジェクトで Claude Code を使用可能にする**

aws-vault で取得した認証情報をファイルに保存し、新しいターミナル/プロジェクトで自動的に読み込むことで、プロジェクトごとの認証設定を不要にします。

## セットアップ手順

### 1. `.zshrc` に関数を追加

`~/.zshrc` に以下を追加してください：

```bash
# aws-vault で取得した認証情報を全プロジェクトで共有
setup-aws-vault-once() {
    echo "🔐 aws-vault で認証情報を取得中..."

    # aws-vault で認証情報を取得
    local creds=$(aws-vault exec dev --json --duration=16h)

    if [ $? -eq 0 ]; then
        # JSON から値を抽出して環境変数に設定
        export AWS_ACCESS_KEY_ID=$(echo $creds | jq -r '.AccessKeyId')
        export AWS_SECRET_ACCESS_KEY=$(echo $creds | jq -r '.SecretAccessKey')
        export AWS_SESSION_TOKEN=$(echo $creds | jq -r '.SessionToken')
        export AWS_REGION=${AWS_REGION:-us-east-1}
        export CLAUDE_CODE_USE_BEDROCK=1
        export ANTHROPIC_MODEL=${ANTHROPIC_MODEL:-us.anthropic.claude-sonnet-4-5-20250929-v1:0}

        # 認証情報をファイルに保存（他のターミナルでも使えるように）
        cat > ~/.aws-vault-session <<EOF
export AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY_ID"
export AWS_SECRET_ACCESS_KEY="$AWS_SECRET_ACCESS_KEY"
export AWS_SESSION_TOKEN="$AWS_SESSION_TOKEN"
export AWS_REGION="$AWS_REGION"
export CLAUDE_CODE_USE_BEDROCK=1
export ANTHROPIC_MODEL="$ANTHROPIC_MODEL"
EOF

        # パーミッション設定
        chmod 600 ~/.aws-vault-session

        echo "✅ 認証情報を設定しました（16時間有効）"
        echo "   🔑 AWS_ACCESS_KEY_ID: ${AWS_ACCESS_KEY_ID:0:10}..."
        echo "   🌍 AWS_REGION: $AWS_REGION"
        echo "   🤖 CLAUDE_CODE_USE_BEDROCK: 1"
        echo ""
        echo "🎉 すべてのプロジェクトで Claude Code が利用可能です！"
    else
        echo "❌ 認証に失敗しました"
    fi
}

# 新しいターミナルを開いたときに自動的に認証情報を読み込む
if [ -f ~/.aws-vault-session ]; then
    source ~/.aws-vault-session

    # 有効期限をチェック（オプション）
    if ! aws sts get-caller-identity &>/dev/null; then
        echo "⚠️  AWS認証の有効期限が切れています"
        echo "💡 'setup-aws-vault-once' を実行して再認証してください"
    fi
fi

# エイリアス
alias aws-auth='setup-aws-vault-once'
```

### 2. 設定を反映

```bash
source ~/.zshrc
```

## 使い方

### 初回認証（1日1回、または16時間ごとに1回）

```bash
setup-aws-vault-once
# または短縮コマンド
aws-auth
```

このコマンドを実行すると：
1. aws-vault の2段階認証が実行される
2. 認証情報が `~/.aws-vault-session` に保存される
3. 現在のターミナルで環境変数が設定される

### その後の使用

**新しいターミナルを開いても自動的に認証済み：**

```bash
# プロジェクト1
cd ~/project1
claude  # すぐ使える！

# 別のターミナルでプロジェクト2
cd ~/project2
claude  # すぐ使える！

# さらに別のターミナルでプロジェクト3
cd ~/project3
claude  # すぐ使える！
```

**setup-claude-code の実行は不要です！**

## メリット

- ✅ 1日1回の認証だけで済む（16時間有効）
- ✅ プロジェクトごとに `setup-claude-code` を実行する必要がない
- ✅ 新しいターミナルでも自動的に認証済み
- ✅ 複数プロジェクトを同時に開いても問題なし
- ✅ VS Code のターミナルでも自動的に使える

## トラブルシューティング

### 認証情報の有効期限が切れた場合

新しいターミナルを開いたときに以下のメッセージが表示されます：

```
⚠️  AWS認証の有効期限が切れています
💡 'setup-aws-vault-once' を実行して再認証してください
```

この場合は、再度 `setup-aws-vault-once` または `aws-auth` を実行してください。

### jq がインストールされていない場合

`jq` が必要です。インストールしてください：

```bash
brew install jq
```

### 認証情報を手動で削除したい場合

```bash
rm ~/.aws-vault-session
```

### 有効期間を変更したい場合

`.zshrc` の `--duration=16h` を変更してください：

```bash
local creds=$(aws-vault exec dev --json --duration=12h)  # 12時間に変更
```

## セキュリティに関する注意

- 認証情報は `~/.aws-vault-session` に保存されます
- ファイルのパーミッションは自動的に `600` (所有者のみ読み書き可能) に設定されます
- 16時間後に自動的に期限切れになります
- `~/.aws-vault-session` は `.gitignore` に含める必要はありません（ホームディレクトリにあるため）

## 代替案：aws-vault を直接使う方法

毎回 aws-vault 経由で Claude Code を起動したい場合：

```bash
# .zshrc に追加
alias claude-dev='aws-vault exec dev --duration=16h -- claude'

# 使い方
claude-dev  # aws-vault 経由で claude を起動
```

この方法だと、認証情報をファイルに保存せずに使用できますが、毎回 aws-vault の認証が必要になります。

## まとめ

この設定により、**1日1回の認証だけで、すべてのプロジェクトで Claude Code をストレスなく使用**できるようになります。
