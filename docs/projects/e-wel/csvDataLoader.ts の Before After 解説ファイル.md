

## 何のためのファイル？

LPページのデータをCSVファイルから読み込むためのファイルです。

  

---

  

## Before（変更前）の実装


```typescript

// 1. グローバル変数でキャッシュを保持

let csvDataCache: Map<string, LPData> | null = null;

let allSlugsCache: string[] | null = null;

  

// 2. キャッシュに保存する関数（戻り値なし）

function loadCSVData(): void {

// CSVファイルを読み込む

const csvContent = fs.readFileSync(csvPath, "utf8");

// データを変換

const rows = parseCSV(cleanContent);

  

// キャッシュ変数に保存

csvDataCache = new Map();

allSlugsCache = [];

rows.forEach((row) => {

csvDataCache!.set(slug, lpData);

allSlugsCache!.push(slug);

});

}

  

// 3. 外部から呼ばれる関数

export function loadLPDataFromCSV(slug: string): LPData | null {

loadCSVData(); // 毎回呼んでいる！

return csvDataCache!.get(slug) || null;

}

```

  

### 問題点

1. **キャッシュが機能していない**: `loadCSVData()` を毎回呼んでいるので、毎回CSVを読み込んでいる

2. **コードが複雑**: グローバル変数を使っているので追いにくい

3. **`!` 演算子が必要**: `csvDataCache!` のように、nullでないことを強制している

  

**例え話**:

図書館で本を探す時に、「一度見つけた本の場所をメモ帳に書いておこう」と思ったけど、探すたびにメモ帳を破り捨てて新しく書き直している状態です。

  

---

  

## After（変更後）の実装

  

```typescript

// 1. グローバル変数を削除（シンプル！）

  

// 2. Mapを返す関数

function loadAndParseCSV(): Map<string, LPData> {

// CSVファイルを読み込む

const csvContent = fs.readFileSync(csvPath, "utf8");

// データを変換

const rows = parseCSV(cleanContent);

  

// 関数内でMapを作成して返す

const dataMap = new Map<string, LPData>();

rows.forEach((row) => {

dataMap.set(slug, lpData);

});

  

return dataMap; // 戻り値として返す

}

  

// 3. 外部から呼ばれる関数

export function loadLPDataFromCSV(slug: string): LPData | null {

const dataMap = loadAndParseCSV(); // データを受け取る

return dataMap.get(slug) || null;

}

```

  

### 改善点

1. **シンプル**: グローバル変数がなく、関数の戻り値で受け渡し

2. **わかりやすい**: データの流れが明確（読み込む → 返す → 使う）

3. **安全**: `!` 演算子が不要

  

**例え話**:

図書館で本を探す時、メモ帳は使わず、探すたびに一から探す。本の数が少ないので十分速いし、コードもシンプル。

  

---

  

## 主な変更点まとめ

| 項目      | Before          | After                 |
| ------- | --------------- | --------------------- |
| キャッシュ変数 | あり（使われていない）     | なし                    |
| データの持ち方 | グローバル変数         | 関数の戻り値                |
| 関数名     | `loadCSVData()` | `loadAndParseCSV()`   |
| 戻り値     | void            | `Map<string, LPData>` |
| 読み込み頻度  | 毎回（実質）          | 毎回（明示的）               |


---

  

## なぜこの変更をしたの？

  

### 理由1: LP数が少ない

- 数十件程度なら、毎回読み込んでも速い

- キャッシュの複雑さが不要

  

### 理由2: 実質的に毎回読み込んでいた

- Beforeでも、`loadCSVData()` を毎回呼んでいた

- キャッシュ変数はあるけど、毎回上書きされていた

- だったら、素直に「毎回読み込む」実装にした方が明確

  

### 理由3: コードの保守性

- グローバル変数が減る = バグが減る

- 関数の役割が明確 = 読みやすい

  

---

  

## 実行の流れ

  

### Before

```

ページ生成 → loadLPDataFromCSV("example")

↓

loadCSVData() (CSVを読んでグローバル変数に保存)

↓

csvDataCache.get("example") を返す

```

  

### After

```

ページ生成 → loadLPDataFromCSV("example")

↓

loadAndParseCSV() (CSVを読んでMapを返す)

↓

dataMap.get("example") を返す

```

  

流れはほぼ同じですが、Afterの方がデータの流れが明確です。