# IBM i SQL 命名規則

## システム名 vs SQL 名

IBM i には **システム名** と **SQL 名** の二重命名体系がある。

- **システム名**: 最大 10 文字。物理ファイル名・ライブラリー名として OS レベルで使用される
- **SQL 名**: 最大 128 文字。SQL ステートメント内で使用される論理名

### ルール
- テーブル・カラム名を新規作成する場合、**システム名（10文字以内）とSQL名の両方を明示**すること
  ```sql
  -- 良い例
  CREATE TABLE MYSCHEMA.CUSTOMER_ORDERS
      FOR SYSTEM NAME CUSTORD (
      ORDER_ID    FOR COLUMN ORDID   INTEGER NOT NULL,
      CUSTOMER_ID FOR COLUMN CUSTID  INTEGER NOT NULL
  );

  -- 悪い例（システム名が未定義）
  CREATE TABLE MYSCHEMA.CUSTOMER_ORDERS (
      ORDER_ID INTEGER NOT NULL
  );
  ```
- システム名は **大文字英数字とアンダースコア** のみを使用し、先頭は英字にすること
- SQL 名が 10 文字を超える場合は必ず `FOR SYSTEM NAME` / `FOR COLUMN` 句で短縮名を指定すること

## スキーマ（ライブラリー）名

- スキーマ名（ライブラリー名）は常に **修飾名** で記述すること
  ```sql
  -- 良い例
  SELECT * FROM MYLIB.MYTABLE;

  -- 悪い例（非修飾）
  SELECT * FROM MYTABLE;
  ```
- スキーマ名が省略される場合は `SET SCHEMA` または `SET PATH` の設定を明示的にコメントで示すこと

## 識別子の大文字・小文字

- SQL キーワードは **大文字** で記述すること（例: `SELECT`, `FROM`, `WHERE`）
- テーブル名・カラム名は **大文字** で統一すること
- 小文字識別子を使用する場合は **二重引用符** で区切り、意図的であることをコメントで示すこと
  ```sql
  -- 大文字/小文字を区別する場合のみ二重引用符を使用
  SELECT "OrderDate" FROM MYLIB.ORDERS;
  ```

## インデックス・制約の命名

- **主キー制約**: `PK_<テーブルシステム名>` 形式（例: `PK_CUSTORD`）
- **外部キー制約**: `FK_<子テーブル>_<親テーブル>` 形式（例: `FK_ORDLIN_CUSTORD`）
- **ユニーク制約**: `UQ_<テーブル>_<カラム群>` 形式（例: `UQ_CUSTORD_CUSTID`）
- **検査制約**: `CK_<テーブル>_<内容>` 形式（例: `CK_CUSTORD_STATUS`）
- **索引**: `IX_<テーブルシステム名>_<カラム群>` 形式（例: `IX_CUSTORD_DATE`）

## ビュー・別名の命名

- ビュー名の末尾には `_V` または `_VW` サフィックスを付けること（例: `CUSTOMER_ORDERS_V`）
- 別名（ALIAS）には元テーブルとの関連が分かるよう命名すること
