# IBM Bob ハンズオン - IBM i Notebook 作成

このリポジトリは、IBM Bob を使用して IBM i 用の .inb Notebook を作成するハンズオン資料です。

## 概要

IBM Bob の Code モードを使用して、IBM i Db2 のシステム監視やジョブ管理のための Notebook を効率的に作成する方法を学びます。

## ディレクトリ構造

```
bob-hands-on-ibmi/
├── README.md                    # このファイル
├── .bob/                        # Bob の設定ディレクトリ
│   └── rules/                   # Bob のカスタムルール
│       ├── ibmi-notebook-builder.md  # IBM i Notebook 作成専用ルール
│       ├── 01-naming-conventions.md  # 命名規則（システム名・SQL名）
│       ├── 02-data-types.md          # データ型の選択指針
│       ├── 03-ddl-table-index.md     # テーブル・インデックス定義
│       ├── 04-dml-select.md          # SELECT・DML の記述ルール
│       ├── 05-transaction-isolation.md  # トランザクション分離レベル
│       ├── 06-procedures-functions.md   # プロシージャー・関数の実装
│       ├── 07-performance-optimization.md  # パフォーマンス最適化
│       ├── 08-ibmi-specific.md       # IBM i 固有機能・注意点
│       └── 09-comments-documentation.md  # コメント・ドキュメント規則
└── Docs/                        # IBM i サービスの参照資料
    ├── ibmi-system-health.md    # システムヘルス監視（SYSLIMITS等）
    ├── ibmi-job-management-part1.md  # ジョブ管理Part1（ACTIVE_JOB_INFO等）
    ├── ibmi-job-management-part2.md  # ジョブ管理Part2（JOB_QUEUE_INFO等）
    ├── ibmi-memory-pool.md      # メモリープール管理（MEMORY_POOL等）
    ├── ibmi-locks-records.md    # ロック・レコード管理（OBJECT_LOCK_INFO等）
    └── ibmi-system-status.md    # システム状況（SYSTEM_STATUS等）
```

### 各ディレクトリの役割

#### `.bob/rules/`
Bob の動作をカスタマイズするルールファイルを格納します。

- **ibmi-notebook-builder.md**: IBM i Notebook 作成時の専門ガイドライン
  - .inb ファイルの構造定義
  - SQL の記述ルール
  - グラフ化の方法
  - IBM i Services の使用方法

- **01-naming-conventions.md**: テーブル・カラムのシステム名（10文字以内）と SQL 名の命名ルール
- **02-data-types.md**: DECIMAL / INTEGER / VARCHAR 等、用途別データ型の選択指針
- **03-ddl-table-index.md**: CREATE TABLE / CREATE INDEX の記述規則
- **04-dml-select.md**: SELECT・INSERT・UPDATE・DELETE の記述スタイル
- **05-transaction-isolation.md**: コミットメント制御と分離レベルの使い分け
- **06-procedures-functions.md**: ストアードプロシージャー・UDF の実装ガイドライン
- **07-performance-optimization.md**: インデックス活用・実行計画の改善手法
- **08-ibmi-specific.md**: IBM i 固有の機能（ジャーナル、*LIBL 等）と注意点
- **09-comments-documentation.md**: ファイルヘッダー・インラインコメントの記述規則

これらのルールにより、Bob は IBM i に特化した高品質な SQL・Notebook を自動生成できます。

#### `Docs/`
IBM i の各種サービス（ビュー・テーブル関数）の詳細な参照資料を格納します。
Bob が Notebook を作成する際に、これらのドキュメントを参照して適切な SQL を生成します。

## 参照資料

`Docs/` ディレクトリには、IBM i サービスに関する詳細なドキュメントが含まれています：

### システムヘルス関連
- **[ibmi-system-health.md](Docs/ibmi-system-health.md)** - システム制限情報の監視

### ジョブ管理関連
- **[ibmi-job-management-part1.md](Docs/ibmi-job-management-part1.md)** - アクティブ・ジョブと終了ジョブの情報
- **[ibmi-job-management-part2.md](Docs/ibmi-job-management-part2.md)** - ジョブキューとサブシステム情報

### メモリープール管理
- **[ibmi-memory-pool.md](Docs/ibmi-memory-pool.md)** - メモリープールと開いているファイルの情報

### ロック・レコード管理
- **[ibmi-locks-records.md](Docs/ibmi-locks-records.md)** - ロック競合とデッドロックの分析

### システム状況
- **[ibmi-system-status.md](Docs/ibmi-system-status.md)** - システム状況とパフォーマンス指標

## Bob での参照方法

Bob に Notebook 作成を依頼する際、これらのドキュメントを参照させることができます：

```
@Docs/ibmi-system-health.md を参照して、システム制限のチェック Notebook を作成してください
```

複数のファイルを参照する場合：

```
@Docs/ibmi-job-management-part1.md と @Docs/ibmi-locks-records.md を参照して、
ジョブのロック状況を分析する Notebook を作成してください
```

## Notebook 作成例

### 例1: システムヘルスチェック
```
システムの制限値と使用状況を監視する Notebook を作成してください。
CPU使用率、メモリー使用率、ストレージ使用率をグラフで表示してください。
```

### 例2: ジョブ監視
```
直近1時間のアクティブジョブと終了ジョブを分析する Notebook を作成してください。
ジョブタイプ別の実行時間をグラフで表示してください。
```

### 例3: ロック競合分析
```
現在のロック状況を確認し、ロック競合が発生しているジョブを特定する Notebook を作成してください。
```

## ベストプラクティス

1. **明確な依頼**
   - 何を監視・分析したいかを明確に伝える
   - グラフの種類（棒グラフ、折れ線グラフなど）を指定する

2. **段階的な作成**
   - まず基本的な Notebook を作成
   - 必要に応じて追加のセクションを依頼

3. **実行と確認**
   - 作成された Notebook を実際に実行して動作確認
   - エラーがあれば Bob に修正を依頼

4. **ドキュメント参照**
   - 適切な IBM i サービスを使用するため、Docs 内の資料を参照させる

## トラブルシューティング

### SQL エラーが発生する場合
- Bob に「SQL を修正してください」と依頼
- エラーメッセージを共有して原因を特定

### グラフが表示されない場合
- データ構造（LABEL 列と VALUE 列）を確認
- Bob に「グラフ表示用のコメントタグを追加してください」と依頼

### パフォーマンスが遅い場合
- FETCH FIRST N ROWS ONLY で件数を制限
- WHERE 句で適切にフィルタリング

## 注意事項

- すべての SQL は IBM i Db2 の構文に従います
- 識別子は ASCII（英大文字 + アンダースコア）で記述
- 日本語は SQL 内に含めず、マークダウンセルで使用
- グラフ化する際は LABEL 列と VALUE 列を明示的に指定

## サポート

質問や問題がある場合は、Bob に直接質問してください。Bob は IBM i Notebook の作成に特化した専門知識を持っています。

## 更新履歴

- **2026-06-03**: IBM i SQL コーディングルール（01〜09）を `.bob/rules/` に追加
- **初版**: IBM Bob ハンズオン資料、Docs/ ディレクトリおよび Notebook 作成ルール追加
