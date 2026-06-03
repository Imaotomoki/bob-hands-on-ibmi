# IBM i SQL トランザクション・コミット制御・分離レベル

## コミット制御の基本

IBM i では「コミットメント制御」と呼ばれるトランザクション管理が使用される。

- トランザクション管理が必要な処理では必ず `COMMIT` / `ROLLBACK` を明示すること
- 自動コミット（AUTOCOMMIT）モードで実行する場合はその旨をコメントで記載すること
  ```sql
  -- コミットメント制御を使用するトランザクション
  SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

  UPDATE MYLIB.ORDERS
  SET STATUS = 'P'
  WHERE ORDER_ID = 12345;

  INSERT INTO MYLIB.ORDER_HISTORY (ORDER_ID, STATUS, CHANGED_AT)
  VALUES (12345, 'P', CURRENT_TIMESTAMP);

  COMMIT;
  ```

## 分離レベルの選択

IBM i の分離レベルは以下の4種類（+ コミットなし）。用途に応じて明示的に指定すること。

| SQL 分離レベル              | IBM i 相当             | 用途 |
|-----------------------------|------------------------|------|
| `SERIALIZABLE`              | 反復可能読み取り (RR)  | 最高レベルの整合性が必要な場合 |
| `REPEATABLE READ`           | 読み取り安定度 (RS)    | 同一トランザクション内で同じ行を繰り返し読む場合 |
| `READ COMMITTED`            | カーソルの固定性 (CS)  | 通常のOLTP処理（推奨デフォルト） |
| `READ UNCOMMITTED`          | コミットされていない読み取り (UR) | 参照専用・ダーティーリード許容の高速読み取り |
| `NO COMMIT` (IBM i 固有)    | コミットなし (NC)      | ジャーナルなし・単純参照のみ |

```sql
-- OLTP処理の標準（推奨）
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 参照専用の高速クエリ（ダーティーリード許容）
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- レポート等で一貫したスナップショットが必要な場合
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

- `WITH` 句を使ってステートメントレベルで分離レベルを指定することもできる
  ```sql
  SELECT ORDER_ID, STATUS
  FROM MYLIB.ORDERS
  WITH UR;  -- Uncommitted Read（IBM i 固有の短縮形）
  ```
  IBM i 固有の短縮形: `WITH RR`, `WITH RS`, `WITH CS`, `WITH UR`, `WITH NC`

## ロック管理

- `SELECT` 文に明示的ロックが必要な場合は `FOR UPDATE OF` を使用すること
  ```sql
  SELECT ORDER_ID, STATUS
  FROM MYLIB.ORDERS
  WHERE ORDER_ID = 12345
  FOR UPDATE OF STATUS;
  ```
- `LOCK TABLE` は最終手段とし、使用する場合はロック時間を最短にすること
  ```sql
  LOCK TABLE MYLIB.ORDERS IN EXCLUSIVE MODE;
  -- 処理
  COMMIT;  -- ロックはコミット/ロールバックで解放される
  ```
- ロックのタイムアウト（`LOCK WAIT`）を考慮し、長時間ロックを保持しないこと

## セーブポイント

- 長いトランザクション内で部分ロールバックが必要な場合はセーブポイントを使用すること
  ```sql
  SAVEPOINT SP1 ON ROLLBACK RETAIN CURSORS;

  UPDATE MYLIB.ORDERS SET STATUS = 'P' WHERE ORDER_ID = 12345;

  -- エラー発生時
  ROLLBACK TO SAVEPOINT SP1;

  -- 成功時
  RELEASE SAVEPOINT SP1;
  COMMIT;
  ```

## エラーハンドリング

- SQLCODE / SQLSTATE を確認してエラー処理を行うこと
- 組み込みSQL（RPGLE等）では `WHENEVER SQLERROR` / `WHENEVER SQLWARNING` を設定すること
  ```sql
  WHENEVER SQLERROR  GO TO ERROR_HANDLER;
  WHENEVER SQLWARNING CONTINUE;
  ```
