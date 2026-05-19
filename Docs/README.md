# IBM i サービスドキュメント

このディレクトリ（`Docs/`）には、IBM i のシステムヘルスとワークマネジメントサービスに関するドキュメントが含まれています。元の大きなドキュメント（`syshealth_workmgmt_services_jp.md`、1813行）を機能別に6つのファイルに分割し、IBM Bob が効率的に参照できるように整理しました。

## ファイル構成

### 1. システムヘルス関連
- **[ibmi-system-health.md](ibmi-system-health.md)**
  - SYSLIMITS ビュー - システム制限情報
  - SYSLIMITS_BASIC ビュー - 基本システム制限情報
  - PROCESS_SYSTEM_LIMITS_ALERTS プロシージャー - システム制限アラート処理

### 2. ジョブ管理関連
- **[ibmi-job-management-part1.md](ibmi-job-management-part1.md)**
  - ACTIVE_JOB_INFO ビュー - アクティブ・ジョブ情報
  - ENDED_JOB_INFO ビュー - 終了ジョブ情報
  - GET_JOB_INFO テーブル関数 - ジョブ情報取得
  - JOB_INFO テーブル関数 - 詳細ジョブ情報
  - JOB_LOCK_INFO ビュー - ジョブ・ロック情報

- **[ibmi-job-management-part2.md](ibmi-job-management-part2.md)**
  - JOB_QUEUE_ENTRIES ビュー - ジョブ待ち行列エントリー
  - JOB_QUEUE_INFO ビュー - ジョブ待ち行列情報
  - SCHEDULED_JOB_INFO ビュー - スケジュールジョブ情報
  - SUBSYSTEM_INFO ビュー - サブシステム情報

### 3. メモリープール管理
- **[ibmi-memory-pool.md](ibmi-memory-pool.md)**
  - MEMORY_POOL テーブル関数 - メモリープール情報
  - MEMORY_POOL_INFO ビュー - メモリープール詳細情報
  - OPEN_FILES テーブル関数 - 開いているファイル情報

### 4. ロック・レコード管理
- **[ibmi-locks-records.md](ibmi-locks-records.md)**
  - OBJECT_LOCK_INFO ビュー - オブジェクト・ロック情報
  - RECORD_LOCK_INFO ビュー - レコード・ロック情報
  - ロック競合の分析方法
  - デッドロック検出のSQL例

### 5. システム状況
- **[ibmi-system-status.md](ibmi-system-status.md)**
  - SYSTEM_STATUS テーブル関数 - システム状況情報
  - SYSTEM_STATUS_INFO ビュー - システム状況詳細
  - SYSTEM_STATUS_INFO_BASIC ビュー - 基本システム状況
  - SYSTEM_ACTIVITY_INFO テーブル関数 - システムアクティビティ
  - SYSTEM_VALUE_INFO ビュー - システム値情報

## 使用方法

### Bob での参照方法

Bob でSQLを記載する際に、これらのファイルを参照するには、以下のように指定します：

```
@Docs/ibmi-system-health.md を参照して、システム制限のチェックSQLを作成してください
```

または、複数のファイルを参照する場合：

```
@Docs/ibmi-job-management-part1.md と @Docs/ibmi-locks-records.md を参照して、
ジョブのロック状況を分析するSQLを作成してください
```

### ファイル選択のガイドライン

| 目的 | 参照すべきファイル |
|------|-------------------|
| システムリソースの監視 | ibmi-system-health.md, ibmi-system-status.md |
| ジョブの状態確認 | ibmi-job-management-part1.md |
| ジョブキューの管理 | ibmi-job-management-part2.md |
| メモリー使用状況の確認 | ibmi-memory-pool.md, ibmi-system-status.md |
| ロック競合の調査 | ibmi-locks-records.md |
| パフォーマンス分析 | ibmi-system-status.md, ibmi-memory-pool.md |

## 各ファイルの特徴

### システムヘルス (ibmi-system-health.md)
- システムの制限値と現在の使用状況を監視
- アラート設定の例を含む
- 容量計画に有用

### ジョブ管理パート1 (ibmi-job-management-part1.md)
- アクティブなジョブの詳細情報
- ジョブのパフォーマンス分析
- ジョブ間のロック関係

### ジョブ管理パート2 (ibmi-job-management-part2.md)
- ジョブキューの管理
- スケジュールジョブの監視
- サブシステムの負荷分析

### メモリープール (ibmi-memory-pool.md)
- メモリープールの使用状況
- ページフォールトの監視
- ファイルI/O統計

### ロック・レコード (ibmi-locks-records.md)
- オブジェクトとレコードのロック状況
- ロック競合の検出
- デッドロックの分析

### システム状況 (ibmi-system-status.md)
- CPU使用率の監視
- ストレージ使用状況
- システム全体のパフォーマンス指標

## SQL例の特徴

各ファイルには以下が含まれています：

1. **基本的な使用例** - 各ビュー/関数の基本的な使い方
2. **実用的なクエリ** - 実際の運用で使用できるSQL
3. **監視・分析例** - パフォーマンス監視やトラブルシューティング用のSQL
4. **ベストプラクティス** - 推奨される使用方法

## 注意事項

- すべてのビューと関数は QSYS2 スキーマに存在します
- 基本的に特別な権限は不要ですが、一部の情報は権限によって制限される場合があります
- パフォーマンスを考慮して、必要なカラムのみを SELECT することを推奨します
- 大量のデータを扱う場合は、適切な WHERE 句でフィルタリングしてください

## ドキュメントの配置場所

これらのドキュメントは `Docs/` フォルダに配置されています。この配置により：

- **プロジェクトドキュメントとして明確に管理**できます
- **`@Docs/ファイル名.md`** の形式で Bob から簡単に参照できます
- **チーム全体で共有**しやすくなります
- **バージョン管理**が容易になります

## 更新履歴

- 2026-05-19: 初版作成
  - 元のドキュメント（1813行）を機能別に6つのファイルに分割
  - Docsフォルダに配置し、プロジェクトドキュメントとして整理
  - 各ファイルに実用的なSQL例とベストプラクティスを追加