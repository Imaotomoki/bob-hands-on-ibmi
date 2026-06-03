# IBM i SQL コメント・ドキュメント規則

## ファイルヘッダーコメント

全ての SQL ファイル先頭に以下のヘッダーを記載すること。

```sql
-- =============================================================================
-- スキーマ    : MYLIB
-- オブジェクト: ORDER_PROCESS_SP (プロシージャー)
-- 概要        : 注文処理メインプロシージャー
--               受注登録・在庫引当・確定処理を一括実行する
-- 作成者      : 山田 太郎
-- 作成日      : 2024-03-01
-- 変更履歴    :
--   2024-03-15  山田太郎  初版作成
--   2024-04-01  鈴木花子  エラーハンドリング追加
-- =============================================================================
```

## COMMENT ON 文の使用

- テーブル・カラムには必ず `COMMENT ON` でテキスト説明を付けること
- QSYS2.SYSTABLES / SYSCOLUMNS の `LONG_COMMENT` に格納される
  ```sql
  -- テーブルへのコメント
  COMMENT ON TABLE MYLIB.CUSTOMER_ORDERS
      IS '受注テーブル: 顧客からの注文ヘッダー情報を管理する';

  -- カラムへのコメント
  COMMENT ON COLUMN MYLIB.CUSTOMER_ORDERS.ORDER_ID
      IS '受注番号: システム自動採番。受注管理の主キー';

  COMMENT ON COLUMN MYLIB.CUSTOMER_ORDERS.STATUS
      IS 'ステータス: N=新規, P=処理中, C=完了, X=キャンセル';
  ```

## インラインコメント

- 複雑な条件・計算には必ずインラインコメントを付けること
  ```sql
  -- 良い例
  WHERE ORDER_DATE >= ADD_MONTHS(CURRENT_DATE, -3)  -- 過去3ヶ月以内
    AND STATUS NOT IN ('X', 'D')                    -- キャンセル・削除済みを除外
    AND AMOUNT > 0                                  -- ゼロ金額は除外

  -- 悪い例（説明なし）
  WHERE ORDER_DATE >= ADD_MONTHS(CURRENT_DATE, -3)
    AND STATUS NOT IN ('X', 'D')
    AND AMOUNT > 0
  ```

- ビジネスルールに関わる条件には必ず背景をコメントで説明すること
  ```sql
  -- 消費税計算: 2019年10月以降は軽減税率対応（食料品は8%、それ以外は10%）
  CASE
      WHEN ITEM_CATEGORY = 'FOOD' AND ORDER_DATE >= DATE('2019-10-01')
          THEN AMOUNT * 0.08
      WHEN ORDER_DATE >= DATE('2019-10-01')
          THEN AMOUNT * 0.10
      ELSE AMOUNT * 0.08
  END AS TAX_AMOUNT
  ```

## 変更管理コメント

- DDL に対して ALTER を行う場合、変更理由をコメントで残すこと
  ```sql
  -- 2024-04-01 山田太郎: GDPR対応により個人情報マスク用カラムを追加
  ALTER TABLE MYLIB.CUSTOMERS
      ADD COLUMN MASKED_EMAIL VARCHAR(100) NOT NULL DEFAULT '';
  ```

- 廃止予定のオブジェクト・カラムには `@DEPRECATED` コメントを付けること
  ```sql
  -- @DEPRECATED: 2024-12以降削除予定。代替: NEW_STATUS カラムを使用すること
  OLD_STATUS  CHAR(1) NOT NULL DEFAULT 'N',
  ```

## TODO / FIXME コメント

- 未対応事項には `TODO` / `FIXME` タグを付けてトラッキングできるようにすること
  ```sql
  -- TODO: パーティション検討 (月次データ増大に備え2024Q4以降に対応予定)
  -- FIXME: この条件は一時的な回避策。#2345 が解決したら修正すること
  ```
