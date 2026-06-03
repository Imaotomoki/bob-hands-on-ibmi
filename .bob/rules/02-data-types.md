# IBM i SQL データ型

## 数値型の選択

IBM i Db2 for i には複数の数値型があり、用途に応じて適切な型を選択すること。

- **金額・精度が必要な数値**: `DECIMAL(p, s)` または `NUMERIC(p, s)` を使用
  - 精度(p)と位取り(s)を必ず明示すること（省略時の動作はバージョンにより異なる）
  - 例: `DECIMAL(15, 2)` で 小数点以下2桁の金額
- **整数**: カラムサイズに応じて `SMALLINT`（-32768〜32767）、`INTEGER`、`BIGINT` を使い分けること
- **浮動小数点** (`FLOAT`, `REAL`, `DOUBLE`): 金額・数量など精度が重要なカラムには **使用禁止**。科学計算など近似値で良い場合のみ使用すること
- **10進浮動小数点**: `DECFLOAT(16)` または `DECFLOAT(34)` は IEEE 754R 準拠。金融計算での使用は設計者と確認すること

```sql
-- 良い例
AMOUNT      DECIMAL(15, 2)   NOT NULL DEFAULT 0,
QUANTITY    INTEGER          NOT NULL DEFAULT 0,
RATE        DECIMAL(7, 4)    NOT NULL DEFAULT 0,

-- 悪い例（浮動小数点を金額に使用）
AMOUNT      FLOAT            NOT NULL DEFAULT 0,
```

## 文字型の選択と CCSID

IBM i は文字コードに **CCSID (Coded Character Set Identifier)** を使用する。

- **固定長文字列**: `CHAR(n)` — 長さが決まっている場合（コード値など）
- **可変長文字列**: `VARCHAR(n)` — 可変長テキスト。最大長を適切に設定すること
- **Unicode対応**: 日本語・多言語を扱う場合は `NCHAR`, `NVARCHAR` または `CCSID 1208`（UTF-8）を明示すること
  ```sql
  -- 日本語カラムの例（EBCDIC日本語 CCSID 939 環境での明示）
  NAME_JP   VARCHAR(100) CCSID 1208 NOT NULL DEFAULT '',

  -- システムデフォルトCCSIDを使用する場合はコメントで明示
  NAME      VARCHAR(100) NOT NULL DEFAULT '',  -- CCSID: システムデフォルト(939想定)
  ```
- `CHAR FOR BIT DATA` および `VARCHAR FOR BIT DATA`: バイナリデータにはこれらを使用し、文字コード変換を回避すること
- 文字列定数のデフォルト CCSID はジョブの CCSID に依存するため、移植性が必要な場合は `X'...'` 16進定数を活用すること

## 日付・時刻型

- **日付**: `DATE` 型を使用。文字列で日付を持つ設計は **禁止**
- **時刻**: `TIME` 型を使用
- **タイムスタンプ**: `TIMESTAMP` 型を使用。精度が必要な場合は `TIMESTAMP(6)` など明示すること
  ```sql
  -- 良い例
  ORDER_DATE    DATE         NOT NULL,
  ENTRY_TIME    TIME         NOT NULL,
  LAST_UPDATED  TIMESTAMP(6) NOT NULL,

  -- 悪い例（文字列で日付を管理）
  ORDER_DATE    CHAR(8)      NOT NULL,  -- yyyymmdd 形式は禁止
  ```
- 日付定数は ISO 形式 `'YYYY-MM-DD'` を使用すること（IBM i はデフォルト日付形式が `*ISO` 以外になる場合がある）
  ```sql
  WHERE ORDER_DATE >= DATE('2024-01-01')
  ```

## LOB 型 (Large Object)

- `CLOB` (Character Large Object): 長大な文字データ（最大 2GB）
- `BLOB` (Binary Large Object): バイナリデータ（最大 2GB）
- `DBCLOB`: ダブルバイト文字の LOB
- LOB カラムを定義する場合は **最大サイズを明示**し、不必要に大きなサイズを避けること
  ```sql
  DOCUMENT   CLOB(1M)  NOT NULL DEFAULT ''
  ```
- LOB カラムに対するインデックスは作成できないため、検索用カラムを別途設けること

## NULL 可否

- 全カラムに `NOT NULL` または `DEFAULT` 句を明示すること
- `NOT NULL` にする場合は `DEFAULT` 値もセットで定義することを推奨
  ```sql
  STATUS   CHAR(1)      NOT NULL DEFAULT 'A',
  REMARKS  VARCHAR(200) NOT NULL DEFAULT '',
  ```
- NULL を許容する場合は、その理由をコメントで記述すること

## ブール型

- `BOOLEAN` 型（IBM i 7.4以降）を使用可能
- 旧来の `CHAR(1)` による `'Y'/'N'` フラグの代わりに `BOOLEAN` の使用を検討すること
  ```sql
  IS_ACTIVE  BOOLEAN NOT NULL DEFAULT TRUE
  ```
