# IBM i SQL DDL（テーブル・制約・索引定義）

## テーブル定義の基本構造

CREATE TABLE 文は以下の順序で記述すること。

1. カラム定義（主キー候補カラムを先頭に）
2. 主キー制約
3. ユニーク制約
4. 検査制約
5. 外部キー制約

```sql
CREATE TABLE MYLIB.ORDER_LINES
    FOR SYSTEM NAME ORDLIN (
    -- キーカラム
    ORDER_ID    FOR COLUMN ORDID   INTEGER        NOT NULL,
    LINE_NO     FOR COLUMN LINNO   SMALLINT       NOT NULL,
    -- 業務カラム
    ITEM_CODE   FOR COLUMN ITMCOD  CHAR(10)       NOT NULL DEFAULT '',
    QUANTITY    FOR COLUMN QTY     DECIMAL(9, 2)  NOT NULL DEFAULT 0,
    UNIT_PRICE  FOR COLUMN UPRC    DECIMAL(15, 2) NOT NULL DEFAULT 0,
    STATUS      FOR COLUMN STS     CHAR(1)        NOT NULL DEFAULT 'A',
    -- 管理カラム
    CREATED_AT  FOR COLUMN CREAT   TIMESTAMP(6)   NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UPDATED_AT  FOR COLUMN UPDAT   TIMESTAMP(6)   NOT NULL DEFAULT CURRENT_TIMESTAMP,
    -- 制約
    CONSTRAINT PK_ORDLIN PRIMARY KEY (ORDER_ID, LINE_NO),
    CONSTRAINT CK_ORDLIN_STATUS CHECK (STATUS IN ('A', 'C', 'X')),
    CONSTRAINT FK_ORDLIN_ORDER
        FOREIGN KEY (ORDER_ID)
        REFERENCES MYLIB.ORDERS (ORDER_ID)
        ON DELETE RESTRICT
        ON UPDATE RESTRICT
);
```

## テーブル属性

- `RCDFMT`（レコード形式名）: RPG との連携が必要な場合は明示すること
  ```sql
  CREATE TABLE MYLIB.MYTABLE
      FOR SYSTEM NAME MYTBL (
      ...
  ) RCDFMT MYTBLR;
  ```
- テーブルスペースを指定する場合は `IN` 句でライブラリーを明示すること

## 制約定義

### 主キー制約
- 全テーブルに必ず主キーを定義すること
- サロゲートキーを使用する場合は `GENERATED ALWAYS AS IDENTITY` を活用すること
  ```sql
  ID  INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY
      (START WITH 1 INCREMENT BY 1 NO CYCLE NO ORDER),
  ```

### 外部キー制約
- 参照整合性は可能な限りデータベースレベルで担保すること
- 削除・更新ルールは `ON DELETE` / `ON UPDATE` で明示すること
  - `RESTRICT`: 参照行が存在する場合に削除・更新を禁止（推奨デフォルト）
  - `CASCADE`: 連鎖削除・更新（意図的な場合のみ使用し、コメントを付けること）
  - `SET NULL`: NULL に設定（カラムが NULL 許容の場合のみ）
  - `NO ACTION`: デフォルト（明示的に記述すること）

### 検査制約
- 列挙値を持つカラムには検査制約を必ず定義すること
  ```sql
  CONSTRAINT CK_ORDERS_STATUS CHECK (STATUS IN ('N', 'P', 'C', 'X'))
  ```

## 索引定義

- 索引は `CREATE INDEX` で明示的に作成し、暗黙的な索引に頼らないこと
- **昇順/降順** を明示すること（IBM i のデフォルトは昇順）
  ```sql
  CREATE INDEX MYLIB.IX_CUSTORD_DATE
      ON MYLIB.CUSTOMER_ORDERS (ORDER_DATE DESC, CUSTOMER_ID ASC);
  ```
- **EVI（Encoded Vector Index）**: 分析クエリに有効。用途を検討して使用すること
  ```sql
  CREATE ENCODED VECTOR INDEX MYLIB.EVI_ORDERS_MONTH
      ON MYLIB.ORDERS (ORDER_MONTH);
  ```
- **ビットマップ索引**: IBM i では EVI がビットマップに相当する
- 索引の `INCLUDE` 句（カバリングインデックス）: クエリのカバー率を高める場合に使用
  ```sql
  CREATE UNIQUE INDEX MYLIB.IX_ORDERS_PK
      ON MYLIB.ORDERS (ORDER_ID)
      INCLUDE (STATUS, ORDER_DATE);
  ```

## ビュー定義

- ビューには必ず `WITH CHECK OPTION` を検討すること（更新可能ビューの場合）
- カラムリストを明示すること（`SELECT *` は禁止）
  ```sql
  CREATE VIEW MYLIB.ACTIVE_ORDERS_V (
      ORDER_ID, CUSTOMER_ID, ORDER_DATE, STATUS
  ) AS
  SELECT
      ORDER_ID,
      CUSTOMER_ID,
      ORDER_DATE,
      STATUS
  FROM MYLIB.CUSTOMER_ORDERS
  WHERE STATUS <> 'X'
  WITH LOCAL CHECK OPTION;
  ```

## ALTER TABLE

- カラム追加時は `DEFAULT` 値と `NOT NULL` を明示すること
- 既存テーブルへのカラム追加後、既存データへのデフォルト値反映を確認すること
  ```sql
  ALTER TABLE MYLIB.ORDERS
      ADD COLUMN LAST_MODIFIED TIMESTAMP(6) NOT NULL
          DEFAULT CURRENT_TIMESTAMP;
  ```

## テーブルのジャーナル設定

- IBM i では物理ファイルのジャーナリングが重要
- トランザクション管理が必要なテーブルは作成後にジャーナルを開始すること
  ```sql
  -- ジャーナル開始はSQLでは直接実行できないため、CLコマンドで実施
  -- STRJRNPF FILE(MYLIB/MYTABLE) JRN(MYLIB/MYJRN)
  -- ※ DDL実行後に必ずジャーナル設定を確認すること
  ```
