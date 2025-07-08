# Step1: 現状調査

## 目次

1. [データベースとコンテンツの調査](#1-データベースとコンテンツの調査)
   - 1.1 [テーブル名対応表](#11-テーブル名対応表)
   - 1.2 [データベーステーブル構造詳細](#12-データベーステーブル構造詳細)
   - 1.3 [画像・メディアファイルの分析](#13-画像メディアファイルの分析)
   - 1.4 [データ関連付けの分析](#14-データ関連付けの分析)
2. [フロントエンドとURL設計の分析](#2-フロントエンドとurl設計の分析)
   - 2.1 [コンテンツタイプ別URL構造パターン](#21-コンテンツタイプ別url構造パターン)
   - 2.2 [sitemap.xmlの分析と分布状況](#22-sitemapxmlの分析と分布状況)
3. [サイト機能と外部連携の調査](#3-サイト機能と外部連携の調査)
   - 3.1 [現存サイトの機能チェックリスト](#31-現存サイトの機能チェックリスト)
   - 3.2 [機能調査結果サマリー](#32-機能調査結果サマリー)

---

## 1. データベースとコンテンツの調査

### 1.1 テーブル名対応表

実際のMySQLテーブル名と日本語名の対応：

#### 1.1.1 主要コンテンツテーブル
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

#### 1.1.2 関連テーブル（多対多関係）
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

#### 1.1.3 データ規模の概要
- **総記事数**: 1,305件（メインコンテンツテーブル合計）
- **最大規模**: ニュース記事（401件）、用語集（379件）
- **タグ関連付け**: 2,452件（関連テーブル合計）

### 1.2 データベーステーブル構造詳細

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

#### 遺伝性疾患解説（diseases）テーブル構造

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


#### 遺伝性疾患関連記事（disease_articles）テーブル構造

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


#### 特集（features）テーブル構造

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


#### 特集記事（feature_articles）テーブル構造

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


#### 用語集（glossary）テーブル構造

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


#### 問い合わせ（inquiries）テーブル構造

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
※ LINEでの問い合わせ


#### インタビュー対象者（interviewees）テーブル構造

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


#### 限定公開記事（limited_release_articles）テーブル構造

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


#### ニュース記事（news_articles）テーブル構造

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


#### その他の記事（other_articles）テーブル構造

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


#### 専門家インタビュー（specialist_interviews）テーブル構造

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


#### 支援者（supporters）テーブル構造

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


#### タグ（tags）テーブル構造

| フィールド名 | データ型 | 必須 | 説明 | WordPress対応 |
|-------------|----------|------|------|---------------|
| id | int unsigned | ○ | 主キー | term_id |
| created_on | datetime | | 作成日時 | - |
| name | varchar(200) | ○ | タグ名（ユニーク） | name |


#### 学習（tutorials）テーブル構造

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


#### 学習記事（tutorial_articles）テーブル構造

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



### 1.3 画像・メディアファイルの分析

#### 1.3.1 メディアファイルの統計分析
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
```sql
-- S3ファイル格納構造の確認
SELECT 
    CHAR_LENGTH(filename_disk) - CHAR_LENGTH(REPLACE(filename_disk, '/', '')) as slash_count,
    CASE 
        WHEN filename_disk NOT LIKE '%/%' THEN 'ROOT_DIRECT'
        WHEN filename_disk LIKE '%/%' AND CHAR_LENGTH(filename_disk) - CHAR_LENGTH(REPLACE(filename_disk, '/', '')) = 1 THEN 'ONE_LEVEL'
        WHEN CHAR_LENGTH(filename_disk) - CHAR_LENGTH(REPLACE(filename_disk, '/', '')) = 2 THEN 'TWO_LEVEL'
        ELSE 'DEEP_NESTED'
    END as path_structure,
    COUNT(*) as file_count,
    MIN(filename_disk) as example_path,
    MAX(filename_disk) as another_example
FROM directus_files 
WHERE storage = 's3'
GROUP BY slash_count
ORDER BY slash_count;

```

#### 1.3.2 S3ファイル格納構造の詳細

【メディアファイル集計】
- 総ファイル数: 1,766 件
- 総サイズ: 500.94 MB
- 画像ファイル: 1,761 件（PNG: 1,093件、JPEG: 668件）
- 音声ファイル: 3 件（M4A）
- その他: 2 件（AI、ZIP）
- アップロード期間: 2020-12-01 ～ 2025-07-07

【S3格納構造の分析結果】
**完全にフラット構造**
- **全1,766ファイルがS3バケットの直下（ルート）に格納**
- フォルダ階層は一切使用されていない
- `slash_count = 0` が全ファイル（100%）

**ファイル命名規則**
- **filename_disk**: UUIDベースのユニークファイル名（例：`6f1c34d6-0a5b-49bc-9cf4-a846b7a7a30a.png`）
- **filename_download**: 元の分かりやすいファイル名（例：`06.png`, `ff-5years.png`）


## 3. サイト機能と外部連携の調査

### 3.1 現存サイトの機能チェックリスト

**基本機能**
- [x] **サイト内検索**: Googleカスタム検索を使用 → 結果: 外部検索エンジン利用
- [x] **ナビゲーション**: 以下のメニュー構成を確認 → 動作: 正常
  - 疾患解説を探す
  - 専門家インタビュー  
  - ニュース
  - 患者さんと支援（/tags/患者）
  - 生活のヒント（/tags/生活のヒント）
  - 用語集
  - 特集
  - 学習コンテンツ
  - サイト内検索ボックス
- [x] **レスポンシブ**

**ユーザー機能**
- [x] **LINE会員限定コンテンツ**: あり → 制御方法: LINE連携による制御

**コンテンツ機能**
- [x] **SNSシェア**:  X, Facebook, LINE
- [x] ページ内の目次の有無
	- 専門家インタビューのみ目次あり
	- ニュースでは最初に要点をまとめた「POINT」がある
- [x] 公式SNS: X, Instagram, TikTok, Facebook, YouTube, LINE
- [x] **関連記事**: 記事下に関連記事表示
	- 専門家インタビューのみ最新２記事を表示
	- その他は関連リンク

**SEO・技術機能**
- [x] **メタタグ**: ページソースで確認
- [x] **サイトマップ**: /sitemap.xml へアクセス 

**外部連携**
- [x] **Microsoft Clarity**: アクセス解析ツール → 設定: 設定により無効化されている
- [x] **Google検索**: Googleカスタム検索使用 → 設定: サイト内検索として実装
- [x] **LINE連携**: 会員限定コンテンツ制御 → 設定: トップバナーで確認
- [x] **YouTube**: 動画埋め込み → 設定: メインコンテンツに埋め込み確認
- [x] **サードパーティクッキー**: 多数の外部サービス連携 → 設定: コンソールで多数の警告確認

### 3.2 機能調査結果サマリー
```
【機能調査結果】
■ 検索機能: Googleカスタム検索使用
■ 会員機能: LINE会員限定コンテンツあり
■ SNS連携: X,Instagram,TikTok,Facebook,YouTube,LINE
■ お問い合わせ: LINE公式アカウントから行う
■ SEO設定: サイトマップ(/sitemap.xml)確認済み、メタタグ・構造化データは要詳細調査
■ 外部連携: Microsoft Clarity、Google検索、LINE、YouTube、多数のサードパーティクッキー
■ その他特記事項: 
  - 専門家インタビューのみ目次機能あり
  - ニュース記事に「POINT」要点まとめあり
  - 関連記事は専門家インタビューのみ最新2記事表示
  - レスポンシブデザイン対応済み
```

### 1.4 データ関連付けの分析

```sql
-- 関連付け情報の取得
SELECT 
    collection_many,
    field_many,
    collection_one,
    field_one,
    junction_field
FROM directus_relations
WHERE collection_many NOT LIKE 'directus_%'
   AND collection_one NOT LIKE 'directus_%'
ORDER BY collection_many;
```

**実行結果から判明した関連付け構造:**

#### 1.4.1 親子関係のデータ構造（One-to-Many）
| 親テーブル | 子テーブル | 関連フィールド | 説明 |
|-----------|-----------|---------------|------|
| diseases | disease_articles | disease | 遺伝性疾患解説 → 遺伝性疾患関連記事 |
| features | feature_articles | feature | 特集 → 特集記事 |
| tutorials | tutorial_articles | tutorial | 学習 → 学習記事 |
| supporters | specialist_interviews | supporter | 支援者 → 専門家インタビュー |
| supporters | links | supporter | 支援者 → リンク |
→ 例）1つの疾患に対して、複数の関連記事が存在する

#### 1.4.2 多対多関係（Many-to-Many）- タグ関連付け
| 中間テーブル | 対象テーブル | 説明 |
|-------------|-------------|------|
| disease_article_tags | disease_articles ↔ tags | 遺伝性疾患関連記事 ↔ タグ |
| disease_tags | diseases ↔ tags | 遺伝性疾患解説 ↔ タグ |
| feature_article_tags | feature_articles ↔ tags | 特集記事 ↔ タグ |
| feature_tags | features ↔ tags | 特集 ↔ タグ |
| news_article_tags | news_articles ↔ tags | ニュース記事 ↔ タグ |
| other_article_tags | other_articles ↔ tags | その他の記事 ↔ タグ |
| specialist_interview_tags | specialist_interviews ↔ tags | 専門家インタビュー ↔ タグ |
| tutorial_article_tags | tutorial_articles ↔ tags | 学習記事 ↔ タグ |
| tutorial_tags | tutorials ↔ tags | 学習 ↔ タグ |
→ 例）1つの記事に複数のタグを付けることができ、1つのタグも複数の記事に使える

#### 1.4.3 特殊な関連付け（インタビュー対象者）
| テーブル | 関連先 | フィールド | 説明 |
|---------|-------|-----------|------|
| interviewees | disease_articles | disease_article | インタビュー対象者 → 遺伝性疾患関連記事 |
| interviewees | specialist_interviews | specialist_interview | インタビュー対象者 → 専門家インタビュー |
| interviewees | other_articles | other_article | インタビュー対象者 → その他の記事 |
通常の一対多や多対多とは異なり、**1人のインタビュー対象者が複数の異なる記事タイプに同時に関連付けされている**構造

#### 1.4.4 データ構造分析の結論

**1. 階層構造がある関連付け**
- 特集 → 特集記事（1つの特集に複数の記事）
- 学習 → 学習記事（1つの学習コンテンツに複数の記事）
- 疾患解説 → 疾患関連記事（1つの疾患に複数の関連記事）

**2. タグによる分類**
- ほぼ全ての記事タイプにタグが付けられている
- 記事を横断的に分類・検索できる仕組み

**3. 人物と記事の関連付け**
- 支援者 → 専門家インタビュー（支援者が出演するインタビュー）
- インタビュー対象者 → 複数の記事タイプ（1人が複数の記事に登場）




## 2. フロントエンドとURL設計の分析

### 2.1 コンテンツタイプ別URL構造パターン

**確認できたURL構造パターン:**

| コンテンツタイプ | URL構造 | 例 | 備考 |
|----------------|---------|-----|------|
| 疾患解説 | `/diseases/[英語疾患名]` | `/diseases/usher-syndrome` | 英語名をケバブケースで使用 |
| 疾患関連臨床試験 | `/diseases/[英語疾患名]/clinical-trials/` | `/diseases/rett-syndrome/clinical-trials/` | 疾患に関する臨床試験情報 |
| 疾患関連インタビュー | `/diseases/[英語疾患名]/interviews/` | `/diseases/duchenne-muscular-dystrophy/interviews/` | 疾患に関するインタビュー |
| 患者情報 | `/diseases/[英語疾患名]/patients/` | `/diseases/duchenne-muscular-dystrophy/patients/` | 患者向け情報 |
| ニュース記事 | `/news/[日付コード]` | `/news/20250704` | 日付形式: YYYYMMDD |
| 特集 | `/features/[特集コード]` | `/features/genetics-music-connect` | 特定テーマの特集 |
| 特集記事 | `/features/[特集コード]/[記事ID]` | `/features/genetics-music-connect/01` | 特集内の個別記事 |
| 専門家インタビュー | `/interviews/[インタビューID]` | `/interviews/genie-system` | インタビュー記事 |
| 学習コンテンツ | `/tutorials/[学習コード]` | `/tutorials/about-genetic-diseases` | 教育的コンテンツ |
| 学習記事 | `/tutorials/[学習コード]/[記事ID]` | `/tutorials/about-genetic-diseases/01` | 学習内の個別記事 |
| タグベース | `/tags/[タグ名]` | `/tags/患者`, `/tags/生活のヒント` | タグによるコンテンツ分類 |
| 用語集 | `/glossary/[用語ID]` | `/glossary/isoform` | 医学用語解説 |

**WordPress移行時の考慮点:**
- 疾患名の英語スラッグは維持が必要（SEO観点）
- 日付ベースのニュース記事URL構造の維持
- タグによる横断的なコンテンツ分類
- 階層構造（diseases/[疾患名]/clinical-trials/, features/[特集]/[記事], tutorials/[学習]/[記事]）の維持

### 2.2 sitemap.xmlの分析と分布状況

**サイトマップの基本情報：**
- 合計URL数：約1,000件
- 最終更新日：2025年7月初旬
- 階層構造：フラットなXMLサイトマップ（個別サイトマップファイルへの分割なし）

**URL分布：**

| コンテンツタイプ | URL数 | 割合 | URL例 |
|---------------|------|-----|-------|
| 疾患解説ページ | 約100件 | 10% | https://genetics.qlife.jp/diseases/usher-syndrome |
| 疾患関連ページ（臨床試験/インタビュー/患者） | 約150件 | 15% | https://genetics.qlife.jp/diseases/rett-syndrome/clinical-trials/ |
| ニュース記事 | 約400件 | 40% | https://genetics.qlife.jp/news/20250704 |
| 特集・特集記事 | 約70件 | 7% | https://genetics.qlife.jp/features/genetics-music-connect |
| インタビュー | 約40件 | 4% | https://genetics.qlife.jp/interviews/genie-system |
| 学習コンテンツ | 約120件 | 12% | https://genetics.qlife.jp/tutorials/about-genetic-diseases |
| タグページ | 約70件 | 7% | https://genetics.qlife.jp/tags/患者 |
| 用語集 | 約50件 | 5% | https://genetics.qlife.jp/glossary/isoform |

**特記事項：**
- すべてのURLは一貫した命名規則に従っている
- 疾患関連ページは疾患名の英語表記をスラッグとして使用
- ニュース記事はYYYYMMDD形式の日付コードを使用
- 特集・学習コンテンツは階層構造のURLを採用

