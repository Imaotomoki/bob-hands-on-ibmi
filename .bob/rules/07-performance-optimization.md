# IBM i SQL パフォーマンス・クエリ最適化

## インデックス活用の原則

- WHERE 句・JOIN 条件・ORDER BY のカラムにはインデックスを定義すること
- 複合インデックスはカーディナリティが高いカラムを **先頭** に配置すること
- カラムに関数を適用するとインデックスが効かなくなるため避けること
  ```sql
  -- 良い例（インデックスが効く）
  WHERE ORDER_DATE >= DATE('2024-01-01')
    AND ORDER_DATE <  DATE('2025-01-01')

  -- 悪い例（インデックスが効かない）
  WHERE YEAR(ORDER_DATE) = 2024
  ```
- LIKE 検索で前方一致 (`'ABC%'`) はインデックスが効くが、後方一致・中間一致 (`'%ABC'`, `'%ABC%'`) は **フルスキャン**になる

## SQE（SQL Query Engine）の活用

IBM i 7.5 では SQE が標準クエリエンジン。以下の点を意識すること。

- 統計情報（列統計）が最適化に使用されるため、大量データ変更後は `CALL QSYS2.ANALYZE_TABLE('MYLIB', 'MYTABLE')` を実行すること
- `OPTIMIZE FOR n ROWS` ヒントで最初のN行取得に最適化できる
  ```sql
  SELECT ORDER_ID, ORDER_DATE
  FROM MYLIB.ORDERS
  WHERE STATUS = 'N'
  OPTIMIZE FOR 10 ROWS;
  ```
- `QUERY OPTIMIZATION GOAL` の設定を把握すること（`*FIRSTIO` vs `*ALLIO`）

## JOIN の最適化

- 大きなテーブルと小さなテーブルを結合する場合、小さなテーブルを先にフィルタリングすること
- `INNER JOIN` / `LEFT JOIN` / `RIGHT JOIN` を意図通りに使い分けること
  - `CROSS JOIN` は意図的な場合のみ使用し、必ずコメントを記述すること
- 複数テーブルの JOIN では結合順序の最適化をオプティマイザーに任せる（ヒント句は最終手段）

## サブクエリ vs JOIN

- 相関サブクエリは多くの場合 JOIN より **遅い**。JOIN への書き換えを検討すること
  ```sql
  -- 良い例（JOIN）
  SELECT O.ORDER_ID, C.CUSTOMER_NAME
  FROM MYLIB.ORDERS AS O
  JOIN MYLIB.CUSTOMERS AS C ON C.CUSTOMER_ID = O.CUSTOMER_ID;

  -- 悪い例（相関サブクエリ）
  SELECT ORDER_ID,
         (SELECT CUSTOMER_NAME FROM MYLIB.CUSTOMERS
          WHERE CUSTOMER_ID = O.CUSTOMER_ID) AS CUSTOMER_NAME
  FROM MYLIB.ORDERS AS O;
  ```
- `EXISTS` / `NOT EXISTS` は `IN` / `NOT IN` より NULL 安全で、大規模データでは高速な場合が多い

## CTE（共通テーブル式）の活用

- 複雑なクエリは `WITH` 句（CTE）で可読性を高めること
  ```sql
  WITH ACTIVE_ORDERS AS (
      SELECT ORDER_ID, CUSTOMER_ID, ORDER_DATE
      FROM MYLIB.CUSTOMER_ORDERS
      WHERE STATUS = 'A'
  ),
  CUSTOMER_SUMMARY AS (
      SELECT CUSTOMER_ID, COUNT(*) AS ORDER_COUNT
      FROM ACTIVE_ORDERS
      GROUP BY CUSTOMER_ID
  )
  SELECT C.CUSTOMER_NAME, S.ORDER_COUNT
  FROM CUSTOMER_SUMMARY AS S
  JOIN MYLIB.CUSTOMERS AS C ON C.CUSTOMER_ID = S.CUSTOMER_ID
  ORDER BY S.ORDER_COUNT DESC;
  ```
- 再帰 CTE は階層データ（BOM、組織図等）の処理に有効
  ```sql
  WITH RECURSIVE BOM (PART_ID, PARENT_ID, LEVEL) AS (
      -- アンカー
      SELECT PART_ID, PARENT_ID, 0
      FROM MYLIB.PARTS
      WHERE PARENT_ID IS NULL
      UNION ALL
      -- 再帰部
      SELECT P.PART_ID, P.PARENT_ID, B.LEVEL + 1
      FROM MYLIB.PARTS AS P
      JOIN BOM AS B ON B.PART_ID = P.PARENT_ID
      WHERE B.LEVEL < 10  -- 再帰の深さ制限（無限ループ防止）
  )
  SELECT * FROM BOM;
  ```

## ウィンドウ関数（OLAP 関数）

- 集計と元データを同時に扱う場合はウィンドウ関数を活用すること
  ```sql
  SELECT
      ORDER_ID,
      ORDER_DATE,
      AMOUNT,
      SUM(AMOUNT) OVER (
          PARTITION BY CUSTOMER_ID
          ORDER BY ORDER_DATE
          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
      ) AS CUMULATIVE_AMOUNT,
      ROW_NUMBER() OVER (
          PARTITION BY CUSTOMER_ID
          ORDER BY ORDER_DATE DESC
      ) AS RECENCY_RANK
  FROM MYLIB.ORDERS;
  ```

## 大量データ処理

- 大量データ更新・削除はバッチ分割して処理すること（ロック競合・ジャーナルの肥大化防止）
  ```sql
  -- バッチ削除の例（1000件ずつ）
  DELETE FROM MYLIB.LOG_TABLE
  WHERE LOG_DATE < DATE('2023-01-01')
  FETCH FIRST 1000 ROWS ONLY;
  -- ↑ を SQLCODE = 0 かつ ROW_COUNT > 0 の間繰り返す
  ```

## Visual Explain / STRDBG の活用

- パフォーマンス問題が発生した場合は IBM i の **Visual Explain** または `STRDBG` でアクセス計画を確認すること
- `QSYS2.SYSIXSTAT` でインデックス統計を確認すること
  ```sql
  SELECT * FROM QSYS2.SYSIXSTAT
  WHERE TABLE_SCHEMA = 'MYLIB'
    AND TABLE_NAME   = 'ORDERS';
  ```
