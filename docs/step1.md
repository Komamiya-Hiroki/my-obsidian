# Step1: 現状調査 - 具体的手順書

## 概要
遺伝性疾患プラス（https://genetics.qlife.jp/）の現状を詳細に調査し、WordPress移行のための基礎データを収集します。

---

## 1. コンテンツの棚卸し

### 1.1 テーブル名対応表

実際のMySQLテーブル名と日本語名の対応：

#### メインテーブル
| 日本語名      | MySQLテーブル名                 | 行数  | 説明             | WordPress投稿タイプ候補   |
| --------- | -------------------------- | --- | -------------- | ------------------ |
| 遺伝性疾患解説   | `diseases`                 | 80  | 疾患に関する詳細解説記事   | `disease`          |
| 遺伝性疾患関連記事 | `disease_articles`         | 152 | 関連する記事コンテンツ    | `disease_article`  |
| 特集        | `features`                 | 13  | 特集企画のメタ情報      | `feature`          |
| 特集記事      | `feature_articles`         | 56  | 特集に含まれる個別記事    | `feature_article`  |
| 用語集       | `glossary`                 | 379 | 医療用語の解説        | `glossary`         |
| 問い合わせ     | `inquiries`                | 98  | お問い合わせフォームデータ  | -                  |
| インタビュー対象者 | `interviewees`             | 102 | インタビュー記事の対象者情報 | `interviewee`      |
| 限定公開記事    | `limited_release_articles` | 19  | 会員限定コンテンツ      | `limited_article`  |
| ニュース      | `news_articles`            | 401 | ニュース記事         | `news`             |
| その他の記事    | `other_articles`           | 57  | 分類されていない記事     | `other_article`    |
| 専門家インタビュー | `specialist_interviews`    | 38  | 専門家へのインタビュー記事  | `interview`        |
| 支援者       | `supporters`               | 62  | サイト支援者情報       | `supporter`        |
| タグ        | `tags`                     | 343 | 記事分類用タグ        | WordPressタグ        |
| 学習        | `tutorials`                | 14  | 学習コンテンツのメタ情報   | `tutorial`         |
| 学習記事      | `tutorial_articles`        | 107 | 学習用記事          | `tutorial_article` |
| リンク       | `links`                    | 22  | リンク集           | `link`             |

#### 関連テーブル（多対多関係）
| テーブル名 | 行数 | 説明 |
|-----------|------|------|
| `disease_article_tags` | 466 | 遺伝性疾患関連記事 ↔ タグ |
| `disease_tags` | 177 | 遺伝性疾患解説 ↔ タグ |
| `feature_article_tags` | 115 | 特集記事 ↔ タグ |
| `feature_tags` | 21 | 特集 ↔ タグ |
| `news_article_tags` | 1,211 | ニュース記事 ↔ タグ |
| `other_article_tags` | 136 | その他の記事 ↔ タグ |
| `specialist_interview_tags` | 93 | 専門家インタビュー ↔ タグ |
| `tutorial_article_tags` | 216 | 学習記事 ↔ タグ |
| `tutorial_tags` | 17 | 学習 ↔ タグ |

#### データ規模サマリー
- **総記事数**: 1,305件（メインコンテンツテーブル合計）
- **最大規模**: ニュース記事（401件）、用語集（379件）
- **タグ関連付け**: 2,452件（関連テーブル合計）

### 1.2 データ数の把握

#### 各コレクションのデータ数を記録
   
| 日本語名 | テーブル名 | 総数 | 公開済み | 下書き | 最終更新日 |
|---------|-----------|------|----------|--------|------------|
| 遺伝性疾患解説 | diseases |  |  |  |  |
| 遺伝性疾患関連記事 | disease_articles |  |  |  |  |
| 特集 | features |  |  |  |  |
| 特集記事 | feature_articles |  |  |  |  |
| 用語集 | glossary |  |  |  |  |
| 問い合わせ | inquiries |  |  |  |  |
| インタビュー対象者 | interviewees |  |  |  |  |
| 限定公開記事 | limited_release_articles |  |  |  |  |
| ニュース | news_articles |  |  |  |  |
| その他の記事 | other_articles |  |  |  |  |
| 専門家インタビュー | specialist_interviews |  |  |  |  |
| 支援者 | supporters |  |  |  |  |
| タグ | tags |  |  |  |  |
| 学習 | tutorials |  |  |  |  |
| 学習記事 | tutorial_articles |  |  |  |  |

#### SQLクエリでの詳細確認

**テーブル構造を確認**
```sql
-- 全テーブルの基本情報を一覧表示
SELECT 
    TABLE_NAME,
    TABLE_ROWS
FROM 
    information_schema.TABLES 
WHERE 
    TABLE_SCHEMA = 'directus_genetics_prd'
    AND TABLE_NAME NOT LIKE 'directus_%'
ORDER BY TABLE_NAME;

-- 各テーブルの構造を確認（実際のフィールド名を把握）
DESCRIBE diseases;
DESCRIBE disease_articles;
DESCRIBE features;
DESCRIBE feature_articles;
DESCRIBE glossary;
DESCRIBE inquiries;
DESCRIBE interviewees;
DESCRIBE limited_release_articles;
DESCRIBE news_articles;
DESCRIBE other_articles;
DESCRIBE specialist_interviews;
DESCRIBE supporters;
DESCRIBE tags;
DESCRIBE tutorials;
DESCRIBE tutorial_articles;
```

**遺伝性疾患解説（diseases）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| name | varchar(60) | ○ | 疾患名 | post_title |
| first_letter | varchar(255) | ○ | 頭文字（検索用） | カスタムフィールド |
| code | varchar(60) | ○ | 疾患コード（ユニーク） | post_name (slug) |
| publish_date | date | | 公開日 | カスタムフィールド |
| description | varchar(200) | | 概要 | post_excerpt |
| english_name | varchar(200) | ○ | 英語名 | カスタムフィールド |
| explanations | text | | 説明 | post_content |
| factors | text | | 要因・原因 | カスタムフィールド |
| diagnosis | text | | 診断 | カスタムフィールド |
| treatments | text | | 治療法 | カスタムフィールド |
| locations | text | | 部位・場所 | カスタムフィールド |
| patients | text | | 患者情報 | カスタムフィールド |
| miscellaneous | text | | その他 | カスタムフィールド |
| main_image | int unsigned | | メイン画像ID | featured_image |
| details | text | | 詳細情報 | カスタムフィールド |
| references | text | | 参考文献 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| kana_name | varchar(60) | | かな名 | カスタムフィールド |
| hospital_searches | text | | 病院検索情報 | カスタムフィールド |


**遺伝性疾患関連記事（disease_articles）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| disease | int unsigned | ○ | 関連疾患ID | カスタムフィールド（関連付け） |
| category | varchar(100) | ○ | カテゴリ | カスタムフィールド |
| filename | varchar(200) | ○ | ファイル名（ユニーク） | post_name (slug) |
| publish_date | date | | 公開日 | カスタムフィールド |
| title | varchar(100) | ○ | タイトル | post_title |
| description | varchar(200) | | 概要 | post_excerpt |
| content | text | | 本文 | post_content |
| main_image | int unsigned | | メイン画像ID | featured_image |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |


**特集（features）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| filename | varchar(200) | ○ | ファイル名（ユニーク） | post_name (slug) |
| title | varchar(200) | ○ | 特集タイトル | post_title |
| description | text | | 特集説明 | post_content |
| supplement | text | | 補足情報 | カスタムフィールド |
| main_image | int unsigned | | メイン画像ID | featured_image |
| publish_date | date | | 公開日 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |


**特集記事（feature_articles）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| sort | int unsigned | | 表示順序 | menu_order |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| filename | varchar(200) | ○ | ファイル名（ユニーク） | post_name (slug) |
| title | varchar(200) | ○ | 記事タイトル | post_title |
| publish_date | date | | 公開日 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |
| feature | int unsigned | ○ | 親特集ID | post_parent |
| youtube_url | varchar(200) | | YouTube URL | カスタムフィールド |
| content | text | | 記事本文 | post_content |
| main_image | int unsigned | | メイン画像ID | featured_image |
| description | varchar(200) | | 記事概要 | post_excerpt |


**用語集（glossary）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| name | varchar(60) | ○ | 用語名（ユニーク） | post_title |
| first_letter | varchar(255) | ○ | 頭文字（検索用） | カスタムフィールド |
| content | text | ○ | 用語説明 | post_content |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| kana_name | varchar(60) | | かな名 | カスタムフィールド |


**問い合わせ（inquiries）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | - |
| created_on | datetime | | 作成日時 | - |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | - |
| email | varchar(255) | ○ | メールアドレス | - |
| name | varchar(200) | | 氏名 | - |
| disease | varchar(200) | | 関連疾患名 | - |
| inquiry_type | varchar(100) | ○ | 問い合わせ種別 | - |
| contents | text | ○ | 問い合わせ内容 | - |
| message | text | | メッセージ | - |
| answers | text | | 回答内容 | - |
| status | varchar(20) | ○ | ステータス（received等） | - |
| memo | text | | 管理用メモ | - |

※問い合わせデータは通常WordPressには移行せず、管理システムで別途管理

**インタビュー対象者（interviewees）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| sort | int unsigned | | 表示順序 | menu_order |
| image | int unsigned | | プロフィール画像ID | featured_image |
| name | varchar(200) | ○ | 氏名 | post_title |
| description | text | | プロフィール説明 | post_content |
| position | varchar(100) | | 役職・肩書き | カスタムフィールド |
| disease_article | int unsigned | | 関連疾患記事ID | カスタムフィールド（関連付け） |
| specialist_interview | int unsigned | | 関連専門家インタビューID | カスタムフィールド（関連付け） |
| other_article | int unsigned | | 関連その他記事ID | カスタムフィールド（関連付け） |


**限定公開記事（limited_release_articles）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| filename | varchar(200) | ○ | ファイル名（ユニーク） | post_name (slug) |
| title | varchar(200) | ○ | 記事タイトル | post_title |
| content | text | | 記事本文 | post_content |
| main_image | int unsigned | | メイン画像ID | featured_image |
| publish_date | date | | 公開日 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |
| description | varchar(200) | | 記事概要 | post_excerpt |


**ニュース記事（news_articles）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| filename | varchar(200) | ○ | ファイル名（ユニーク） | post_name (slug) |
| publish_date | date | | 公開日 | カスタムフィールド |
| title | varchar(100) | ○ | 記事タイトル | post_title |
| description | varchar(200) | | 記事概要 | post_excerpt |
| content | text | | 記事本文 | post_content |
| points | text | | ポイント・要点 | カスタムフィールド |
| main_image | int unsigned | | メイン画像ID | featured_image |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |


**その他の記事（other_articles）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| filename | varchar(200) | ○ | ファイル名（ユニーク） | post_name (slug) |
| title | varchar(100) | ○ | 記事タイトル | post_title |
| description | varchar(200) | | 記事概要 | post_excerpt |
| content | text | | 記事本文 | post_content |
| main_image | int unsigned | | メイン画像ID | featured_image |
| publish_date | date | | 公開日 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |
| category | varchar(100) | ○ | カテゴリ | カスタムフィールド |


**専門家インタビュー（specialist_interviews）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| filename | varchar(200) | ○ | ファイル名（ユニーク） | post_name (slug) |
| publish_date | date | | 公開日 | カスタムフィールド |
| title | varchar(100) | ○ | インタビュータイトル | post_title |
| description | varchar(200) | | インタビュー概要 | post_excerpt |
| content | text | | インタビュー本文 | post_content |
| main_image | int unsigned | | メイン画像ID | featured_image |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |
| supporter | int unsigned | | 関連支援者ID | カスタムフィールド（関連付け） |


**支援者（supporters）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| name | varchar(200) | ○ | 支援者名 | post_title |
| title_affiliation | varchar(200) | | 肩書き・所属 | カスタムフィールド |
| kana | varchar(200) | ○ | かな名 | カスタムフィールド |
| message | text | | メッセージ | post_content |
| main_image | int unsigned | | プロフィール画像ID | featured_image |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| publish_date | date | | 公開日 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |


**タグ（tags）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | term_id |
| created_on | datetime | | 作成日時 | - |
| name | varchar(200) | ○ | タグ名（ユニーク） | name |


**学習（tutorials）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| title | varchar(60) | ○ | 学習タイトル | post_title |
| code | varchar(60) | ○ | 学習コード（ユニーク） | post_name (slug) |
| main_image | int unsigned | | メイン画像ID | featured_image |
| publish_date | date | | 公開日 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |
| description | varchar(200) | | 学習説明 | post_excerpt |


**学習記事（tutorial_articles）テーブル構造**

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | post_id |
| status | varchar(20) | ○ | 公開状態（draft等） | post_status |
| sort | int unsigned | | 表示順序 | menu_order |
| owner | int unsigned | | 作成者ID | post_author |
| created_on | datetime | | 作成日時 | post_date |
| modified_by | int unsigned | | 更新者ID | - |
| modified_on | datetime | | 更新日時 | post_modified |
| published_at | datetime | | 公開日時 | カスタムフィールド |
| filename | varchar(200) | ○ | ファイル名（ユニーク） | post_name (slug) |
| title | varchar(200) | ○ | 記事タイトル | post_title |
| content | text | | 記事本文 | post_content |
| main_image | int unsigned | | メイン画像ID | featured_image |
| tutorial | int unsigned | ○ | 親学習ID | post_parent |
| publish_date | date | | 公開日 | カスタムフィールド |
| publish_time | time | | 公開時刻 | カスタムフィールド |
| description | varchar(2000) | | 記事概要（長文） | post_excerpt |



### 1.2 メディアファイルの把握

#### 画像・ファイル数の確認
```sql
-- 全体のサマリー
SELECT 
    COUNT(*) as total_files,
    ROUND(SUM(filesize)/1024/1024, 2) as total_size_mb,
    MIN(uploaded_on) as oldest_file,
    MAX(uploaded_on) as newest_file
FROM directus_files;

-- メディアファイルの総数と種類別集計
SELECT 
    type,
    COUNT(*) as file_count,
    ROUND(SUM(filesize)/1024/1024, 2) as total_size_mb
FROM directus_files 
WHERE type IS NOT NULL
GROUP BY type
ORDER BY file_count DESC;

-- ファイル拡張子別の集計
SELECT 
    SUBSTRING_INDEX(filename_download, '.', -1) as extension,
    COUNT(*) as file_count,
    ROUND(SUM(filesize)/1024/1024, 2) as total_size_mb
FROM directus_files 
WHERE filename_download IS NOT NULL
GROUP BY SUBSTRING_INDEX(filename_download, '.', -1)
ORDER BY file_count DESC;

-- ファイル格納場所の確認
SELECT 
    storage,
    COUNT(*) as file_count,
    ROUND(SUM(filesize)/1024/1024, 2) as total_size_mb
FROM directus_files 
GROUP BY storage
ORDER BY file_count DESC;

-- フォルダ別の集計（階層構造の把握）
SELECT 
    COALESCE(folder, 0) as folder_id,
    COUNT(*) as file_count,
    ROUND(SUM(filesize)/1024/1024, 2) as total_size_mb
FROM directus_files 
GROUP BY folder
ORDER BY file_count DESC;
```

```
【メディアファイル集計】
- 総ファイル数: 1,766 件
- 総サイズ: 500.94 MB
- 画像ファイル: 1,761 件（PNG: 1,093件、JPEG: 668件）
- 音声ファイル: 3 件（M4A）
- その他: 2 件（AI、ZIP）
- アップロード期間: 2020-12-01 ～ 2025-07-07
```

### 1.3 使用機能の洗い出し

#### 確認すべき機能リスト
- [ ] **検索機能**
  - 全文検索の有無
  - フィルタリング機能
  - 検索対象フィールド
- [ ] **会員機能**
  - ユーザー登録・ログイン
  - 権限レベル
  - 限定コンテンツの制御方法
- [ ] **お問い合わせ機能**
  - フォーム項目
  - 送信先メールアドレス
  - 自動返信機能
- [ ] **コメント機能**
- [ ] **SNS連携**
- [ ] **RSS/Atom配信**
- [ ] **多言語対応**
- [ ] **SEO機能**
  - メタタグ設定
  - 構造化データ
  - サイトマップ

### 1.4 関連付けの確認

```sql
-- リレーション情報の取得
SELECT 
    many_collection,
    many_field,
    one_collection,
    one_field,
    junction_field
FROM directus_relations
WHERE many_collection NOT LIKE 'directus_%'
   OR one_collection NOT LIKE 'directus_%'
ORDER BY many_collection;
```

**主要な関連付け**
1. 遺伝性疾患解説 ← 遺伝性疾患関連記事
2. 特集 ← 特集記事
3. 学習 ← 学習記事
4. 専門家インタビュー → 支援者
5. 専門家インタビュー → インタビュー対象者
6. 全記事 → タグ（多対多）

---

## 2. URL構造の確認

### 2.1 実際のサイトでのURL調査

#### 手動でのURL確認手順
1. **トップページから各セクションへアクセス**
   - https://genetics.qlife.jp/
   - 主要ナビゲーションをクリックして各ページのURLを記録

2. **URL パターンの記録**
   
   ```
   【URL構造調査結果】
   
   ■ 遺伝性疾患解説
   - パターン: /diseases/[疾患名]
   - 例: /diseases/huntington-disease
   
   ■ ニュース
   - パターン: /news/[記事ID or スラッグ]
   - 例: /news/2024-01-15-new-research
   
   ■ 専門家インタビュー
   - パターン: /interviews/[インタビューID]
   - 例: /interviews/dr-tanaka-genetics
   
   ■ 特集
   - パターン: /features/[特集名]/[記事名]
   - 例: /features/genetic-counseling/basic-knowledge
   
   ■ 学習コンテンツ
   - パターン: /learning/[カテゴリ]/[記事名]
   - 例: /learning/basics/dna-structure
   
   ■ 用語集
   - パターン: /glossary/[用語名]
   - 例: /glossary/gene-therapy
   
   ■ 限定公開記事
   - パターン: /members/[記事名]
   - 例: /members/advanced-research-2024
   ```

### 2.2 内部リンク構造の調査

#### リンク調査用スクリプト（参考）
```javascript
// ブラウザのコンソールで実行
// 内部リンクの一覧を取得
const internalLinks = Array.from(document.querySelectorAll('a[href]'))
  .map(link => link.href)
  .filter(href => href.includes('genetics.qlife.jp'))
  .filter((href, index, array) => array.indexOf(href) === index)
  .sort();

console.log('内部リンク一覧:', internalLinks);
```

#### 特殊なURL・パラメータの確認
- [ ] **検索URL**: `/search?q=keyword&category=disease`
- [ ] **フィルタリングURL**: `/diseases?category=single-gene&sort=name`
- [ ] **ページネーション**: `/news?page=2`
- [ ] **日本語URL**: `/diseases/ハンチントン病`
- [ ] **アンカーリンク**: `/diseases/huntington#symptoms`

---

## 4. SEO・メタデータの調査

### 4.1 現在のSEO設定確認

#### 各ページのメタデータ確認
```javascript
// ブラウザコンソールで実行
// 現在のページのメタデータを取得
const metaData = {
  title: document.title,
  description: document.querySelector('meta[name="description"]')?.content,
  keywords: document.querySelector('meta[name="keywords"]')?.content,
  ogTitle: document.querySelector('meta[property="og:title"]')?.content,
  ogDescription: document.querySelector('meta[property="og:description"]')?.content,
  ogImage: document.querySelector('meta[property="og:image"]')?.content,
  canonical: document.querySelector('link[rel="canonical"]')?.href,
  structuredData: Array.from(document.querySelectorAll('script[type="application/ld+json"]'))
    .map(script => script.textContent)
};

console.log('メタデータ:', metaData);
```

### 4.2 構造化データの確認

#### 使用されている構造化データの種類
- [ ] **Article** - 記事コンテンツ
- [ ] **MedicalWebPage** - 医療関連ページ
- [ ] **FAQPage** - よくある質問
- [ ] **BreadcrumbList** - パンくずリスト
- [ ] **Organization** - 組織情報
- [ ] **WebSite** - サイト情報

---

## 5. アクセス解析データの確認

### 5.1 Google Analytics データ

#### 確認すべき指標
- [ ] **月間PV数**
- [ ] **月間UU数**
- [ ] **人気ページランキング（上位20位）**
- [ ] **検索流入キーワード（上位50位）**
- [ ] **デバイス別アクセス比率**
- [ ] **直帰率**
- [ ] **平均セッション時間**

### 5.2 Search Console データ

#### 確認すべき項目
- [ ] **検索パフォーマンス**
  - 表示回数の多いクエリ
  - クリック率の高いページ
  - 平均掲載順位
- [ ] **インデックス状況**
  - インデックス済みページ数
  - エラーページの有無
- [ ] **モバイルユーザビリティ**
- [ ] **Core Web Vitals**

---

## 6. 調査結果のまとめ

### 6.1 調査結果レポート作成

#### レポートテンプレート
```markdown
# 遺伝性疾患プラス 現状調査レポート

## サマリー
- 総記事数: _____ 件
- 総メディアファイル数: _____ 件（_____ MB）
- 主要コンテンツタイプ: _____ 種類
- 月間PV: _____ 
- 主要な技術的課題: _____

## 詳細データ
### コンテンツ統計
[1.1で作成した表を挿入]

### URL構造
[3.1で作成したURL一覧を挿入]

### 技術的要件
- 必要なWordPressプラグイン: _____
- カスタム投稿タイプ数: _____
- 複雑な関連付け: _____

## 移行時の注意点
1. _____
2. _____
3. _____

## 次のステップ
- Step2での優先対応項目
- 想定される課題と対策
```

### 6.2 移行計画への反映

#### Step2準備のためのチェックリスト
- [ ] **必要なカスタム投稿タイプが明確になった**
- [ ] **必要なカスタムフィールドが特定できた**
- [ ] **URL構造の設計方針が決まった**
- [ ] **移行データ量が把握できた**
- [ ] **技術的制約が明確になった**

---

## 7. 作業時間の目安

| 作業項目 | 想定時間 | 担当者 |
|----------|----------|--------|
| データ数調査 | 2-3時間 | エンジニア |
| DB構造調査 | 4-6時間 | エンジニア |
| URL構造調査 | 2-3時間 | エンジニア/ディレクター |
| SEO調査 | 2-3時間 | マーケター |
| 解析データ確認 | 1-2時間 | マーケター |
| レポート作成 | 2-3時間 | ディレクター |
| **合計** | **13-20時間** | |

---

## 8. 必要なアクセス権限・ツール

### 8.1 必要なアクセス権限
- [ ] **Directus管理画面** - 管理者権限
- [ ] **データベース直接アクセス** - 読み取り権限
- [ ] **Google Analytics** - 閲覧権限
- [ ] **Google Search Console** - 閲覧権限
- [ ] **サーバー** - ファイル閲覧権限

### 8.2 使用ツール
- [ ] **ブラウザ** - Chrome/Firefox（開発者ツール使用）
- [ ] **データベースクライアント** - phpMyAdmin/MySQL Workbench
- [ ] **スプレッドシート** - Excel/Google Sheets（データ整理用）
- [ ] **テキストエディタ** - VSCode/Sublime Text

---

## 完了チェック

Step1完了の判断基準：
- [ ] 全コレクションのデータ数が把握できた
- [ ] 主要なフィールド構造が文書化された
- [ ] URL構造パターンが明確になった
- [ ] 移行に必要な技術要件が特定できた
- [ ] 調査結果レポートが作成された
- [ ] Step2で必要な情報が整理された

**Step1完了後、Step2の具体的計画を立案できる状態になります。**
