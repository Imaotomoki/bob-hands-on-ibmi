# IBM i SQL 固有機能・IBM i 特有の考慮事項

## 命名方式（SQL 命名 vs システム命名）

IBM i には2種類の命名方式がある。プロジェクト内で統一すること。

| 項目              | SQL 命名方式                     | システム命名方式              |
|-------------------|----------------------------------|-------------------------------|
| 区切り文字        | ピリオド (`.`)                   | スラッシュ (`/`)              |
| 例                | `MYLIB.MYTABLE`                  | `MYLIB/MYTABLE`               |
| デフォルトスキーマ | `SET SCHEMA` で指定              | ライブラリーリストで解決      |
| 設定方法          | `NAMING = *SQL`                  | `NAMING = *SYS`               |

- SQLファイル内では **SQL命名方式 (`MYLIB.MYTABLE`)** を統一使用すること
- RPGLE/CLから呼び出すSQLでも SQL命名方式を推奨
- システム命名方式を使用する場合はファイル先頭にコメントで明示すること
  ```sql
  -- 命名方式: システム命名 (*SYS) / ライブラリーリスト使用
  ```

## 特殊レジスター

IBM i 特有の特殊レジスターを積極的に活用すること。

```sql
-- よく使う特殊レジスター
CURRENT DATE          -- 現在日付
CURRENT TIME          -- 現在時刻
CURRENT TIMESTAMP     -- 現在タイムスタンプ
CURRENT USER          -- 現在のユーザーID（認証名）
SESSION_USER          -- セッションユーザー
CURRENT SCHEMA        -- 現在のデフォルトスキーマ
CURRENT PATH          -- 現在の SQL パス
CURRENT SERVER        -- 現在のサーバー名（分散DB）
CURRENT ISOLATION     -- 現在の分離レベル
```

```sql
-- 使用例
INSERT INTO MYLIB.AUDIT_LOG (USER_ID, ACTION, LOG_TIME)
VALUES (CURRENT USER, 'UPDATE', CURRENT TIMESTAMP);
```

## QSYS2 サービステーブル・ビューの活用

IBM i 提供のシステムカタログビューを活用してメタデータを参照すること。

```sql
-- テーブル一覧
SELECT TABLE_NAME, TABLE_TEXT, COLUMN_COUNT
FROM QSYS2.SYSTABLES
WHERE TABLE_SCHEMA = 'MYLIB'
ORDER BY TABLE_NAME;

-- カラム一覧
SELECT COLUMN_NAME, DATA_TYPE, LENGTH, NUMERIC_SCALE,
       IS_NULLABLE, COLUMN_DEFAULT, COLUMN_TEXT
FROM QSYS2.SYSCOLUMNS
WHERE TABLE_SCHEMA = 'MYLIB'
  AND TABLE_NAME   = 'ORDERS'
ORDER BY ORDINAL_POSITION;

-- インデックス情報
SELECT INDEX_NAME, COLUMN_NAMES, IS_UNIQUE
FROM QSYS2.SYSINDEXSTAT
WHERE TABLE_SCHEMA = 'MYLIB'
  AND TABLE_NAME   = 'ORDERS';

-- ジョブ情報
SELECT JOB_NAME, AUTHORIZATION_NAME, JOB_STATUS
FROM QSYS2.JOB_INFO
WHERE JOB_STATUS = '*ACTIVE';
```

## シーケンスオブジェクト

- 連番生成には `IDENTITY` カラムまたは `CREATE SEQUENCE` を使用すること
  ```sql
  -- シーケンス定義
  CREATE OR REPLACE SEQUENCE MYLIB.SEQ_ORDER_ID
      AS INTEGER
      START WITH 1
      INCREMENT BY 1
      NO MAXVALUE
      NO CYCLE
      ORDER;

  -- 使用方法
  NEXT VALUE FOR MYLIB.SEQ_ORDER_ID   -- 次の値を取得
  PREVIOUS VALUE FOR MYLIB.SEQ_ORDER_ID -- 直前に生成した値
  ```
- RPGのような連番管理テーブルは **廃止し** シーケンスに移行すること

## グローバル変数

- セッション間でデータを共有する場合は `CREATE VARIABLE` を活用すること
  ```sql
  CREATE OR REPLACE VARIABLE MYLIB.GBL_PROCESS_DATE DATE
      DEFAULT CURRENT_DATE;

  -- 設定
  SET MYLIB.GBL_PROCESS_DATE = DATE('2024-03-31');

  -- 参照
  SELECT * FROM MYLIB.ORDERS
  WHERE ORDER_DATE = MYLIB.GBL_PROCESS_DATE;
  ```

## 配列型とROW型

IBM i 7.x では配列型とROW型をサポートしている。

```sql
-- 配列型の定義
CREATE OR REPLACE TYPE MYLIB.INT_ARRAY AS INTEGER ARRAY[100];

-- ROW型
DECLARE V_ORDER ROW (ORDER_ID INTEGER, STATUS CHAR(1));
SET V_ORDER.ORDER_ID = 12345;
SET V_ORDER.STATUS   = 'A';
```

## データ・リンク（DATALINK）

- IBM i IFS（統合ファイルシステム）のファイルへのリンクには `DATALINK` 型を使用できる
- 外部ファイルパスをデータベース管理下に置く場合に有効

## FOR XML 句

- クエリ結果を XML 形式で取得する場合は `FOR XML` 句を使用すること
  ```sql
  SELECT ORDER_ID, CUSTOMER_ID, ORDER_DATE
  FROM MYLIB.ORDERS
  WHERE STATUS = 'A'
  FOR XML AUTO;
  ```

## PIPE 関数（テーブルパイプ）

- 結果セットをパイプ的に処理する場合は IBM i 固有の `PIPE` 関数を活用すること

## IBM i 固有の注意事項

- **ライブラリーリスト**: システム命名方式を使用している既存コードと混在する場合、ライブラリーリストの解決順序に注意すること
- **ジャーナル**: DDL 実行後、新しいテーブルへのジャーナル設定を忘れないこと（コミットメント制御に必須）
- **物理ファイル vs SQLテーブル**: DDS で作成された物理ファイルをSQL で操作する場合、フィールド定義の互換性を確認すること
- **CCSID 混在**: 異なるCCSIDのテーブル間でJOINする場合、暗黙的変換によるパフォーマンス低下に注意すること
- **EBCDIC**: IBM i は標準で EBCDIC を使用。ASCII システムとのデータ連携時はCCSID変換を明示的に行うこと
