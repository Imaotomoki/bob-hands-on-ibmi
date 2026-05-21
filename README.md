# IBM Bob ハンズオン - IBM i Notebook 作成

このリポジトリは、IBM Bob を使用して IBM i 用の .inb Notebook を作成するハンズオン資料です。

## 概要

IBM Bob の Code モードを使用して、IBM i Db2 のシステム監視やジョブ管理のための Notebook を効率的に作成する方法を学びます。


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

