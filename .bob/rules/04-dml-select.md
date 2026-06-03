# IBM i SQL DML（SELECT・INSERT・UPDATE・DELETE）

## SELECT 文の基本規則

### カラム指定
- `SELECT *` は **禁止**。必要なカラムを明示的に列挙すること
  ```sql
  -- 良い例
  SELECT
      ORDER_ID,
      CUSTOMER_ID,
      ORDER_DATE,
      STATUS
  FROM MYLIB.CUSTOMER_ORDERS;

  -- 悪い例
  SELECT * FROM MYLIB.CUSTOMER_ORDERS;
  ```

### テーブル修飾
- テーブル名は必ずスキーマ名（ライブラリー名）で修飾すること
- テーブル別名（エイリアス）を使用する場合は意味のある短縮名を付けること
  ```sql
  SELECT
      O.ORDER_ID,
      O.ORDER_DATE,
      C.CUSTOMER_NAME
  FROM MYLIB.CUSTOMER_ORDERS AS O
  JOIN MYLIB.CUSTOMERS       AS C ON C.CUSTOMER_ID = O.CUSTOMER_ID
  WHERE O.STATUS = 'A';
  ```

### フォーマット
- 各句（`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`）は改行して記述すること
- `SELECT` リストの各カラムは1行に1カラムを基本とすること
- `JOIN` 条件は `ON` 句に記述し、`WHERE` 句の暗黙的結合は **禁止**
  ```sql
  -- 良い例（明示的JOIN）
  FROM MYLIB.ORDERS AS O
  JOIN MYLIB.ORDER_LINES AS L ON L.ORDER_ID = O.ORDER_ID

  -- 悪い例（暗黙的結合）
  FROM MYLIB.ORDERS O, MYLIB.ORDER_LINES L
  WHERE O.ORDER_ID = L.ORDER_ID
  ```

## WHERE 句

- `WHERE 1=1` のような無意味な条件は **禁止**
- NULL 比較は `IS NULL` / `IS NOT NULL` を使用すること（`= NULL` は **禁止**）
  ```sql
  -- 良い例
  WHERE DELETED_AT IS NULL

  -- 悪い例
  WHERE DELETED_AT = NULL
  ```
- `NOT IN` に NULL が含まれる可能性がある場合は `NOT EXISTS` を使用すること
  ```sql
  -- 良い例（NULL安全）
  WHERE NOT EXISTS (
      SELECT 1 FROM MYLIB.BLACKLIST AS B
      WHERE B.CUSTOMER_ID = C.CUSTOMER_ID
  )

  -- 悪い例（NULLが含まれると全行が除外される）
  WHERE CUSTOMER_ID NOT IN (SELECT CUSTOMER_ID FROM MYLIB.BLACKLIST)
  ```
- インデックス効率を考慮し、カラム側に関数を適用しないこと
  ```sql
  -- 良い例
  WHERE ORDER_DATE >= DATE('2024-01-01')
    AND ORDER_DATE <  DATE('2024-04-01')

  -- 悪い例（インデックスが効かない）
  WHERE YEAR(ORDER_DATE) = 2024 AND MONTH(ORDER_DATE) = 1
  ```

## INSERT 文

- カラムリストを必ず明示すること
  ```sql
  -- 良い例
  INSERT INTO MYLIB.ORDERS (ORDER_ID, CUSTOMER_ID, ORDER_DATE, STATUS)
  VALUES (NEXT VALUE FOR MYLIB.SEQ_ORDER_ID, 1001, CURRENT_DATE, 'N');

  -- 悪い例（カラムリストなし）
  INSERT INTO MYLIB.ORDERS VALUES (1, 1001, CURRENT_DATE, 'N');
  ```
- 複数行挿入には `INSERT ... VALUES (row1), (row2)` または `INSERT ... SELECT` を使用すること

## UPDATE 文

- `WHERE` 句なしの `UPDATE` は **禁止**（全件更新が意図的な場合はコメントで理由を明記）
- 更新するカラムのみを `SET` 句に列挙すること
  ```sql
  UPDATE MYLIB.ORDERS
  SET
      STATUS     = 'C',
      UPDATED_AT = CURRENT_TIMESTAMP
  WHERE ORDER_ID = 12345;
  ```

## DELETE 文

- `WHERE` 句なしの `DELETE` は **禁止**（全件削除が意図的な場合は `TRUNCATE TABLE` を使用し、コメントで理由を明記）
  ```sql
  -- 論理削除を推奨（物理削除は慎重に）
  UPDATE MYLIB.ORDERS
  SET    DELETED_AT = CURRENT_TIMESTAMP
  WHERE  ORDER_ID = 12345;

  -- 物理削除する場合
  DELETE FROM MYLIB.ORDERS
  WHERE ORDER_ID = 12345;
  ```

## FETCH FIRST / LIMIT

- 大量データ取得を防ぐため、行数制限が不要な場合を除き `FETCH FIRST n ROWS ONLY` を付けること
  ```sql
  SELECT ORDER_ID, ORDER_DATE
  FROM MYLIB.ORDERS
  ORDER BY ORDER_DATE DESC
  FETCH FIRST 100 ROWS ONLY;
  ```

## MERGE 文

- UPSERT 処理には `MERGE` 文を活用すること
  ```sql
  MERGE INTO MYLIB.CUSTOMER_TOTALS AS T
  USING (VALUES (1001, 50000.00)) AS S (CUSTOMER_ID, AMOUNT)
  ON T.CUSTOMER_ID = S.CUSTOMER_ID
  WHEN MATCHED THEN
      UPDATE SET TOTAL_AMOUNT = T.TOTAL_AMOUNT + S.AMOUNT,
                 UPDATED_AT   = CURRENT_TIMESTAMP
  WHEN NOT MATCHED THEN
      INSERT (CUSTOMER_ID, TOTAL_AMOUNT, CREATED_AT, UPDATED_AT)
      VALUES (S.CUSTOMER_ID, S.AMOUNT, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP);
  ```

## ORDER BY

- ページング・先頭N件取得の場合は必ず `ORDER BY` を付けること
- カラム番号による `ORDER BY 1, 2` は **禁止**。カラム名で明示すること
  ```sql
  -- 良い例
  ORDER BY ORDER_DATE DESC, ORDER_ID ASC

  -- 悪い例
  ORDER BY 1 DESC, 2 ASC
  ```
