# IBM i メモリープール管理サービス

このドキュメントは、IBM i のメモリープール管理に関連するシステムサービスについて説明します。

## MEMORY_POOL テーブル関数

MEMORY_POOL テーブル関数は、メモリー・プールに関する情報を返します。統計のリセット・オプションがあります。

**権限:** 不要。

```
>>-- MEMORY_POOL --(
  RESET_STATISTICS => reset-statistics
)-->>
```
スキーマは QSYS2 です。

**パラメータ:**
- `RESET_STATISTICS`: 統計をリセットするかどうか（'YES' または 'NO'）

**主要カラム:**

| カラム名 | データ型 | 説明 |
| --- | --- | --- |
| SYSTEM_POOL_ID | INTEGER | システム・ストレージ・プールのシステム関連プール識別子 |
| POOL_NAME | VARCHAR(10) | ストレージ・プールの名前（*MACHINE、*BASE、*INTERACT、*SPOOL、*SHRPOOLx など） |
| CURRENT_SIZE | DECIMAL(20,2) | プール内の主記憶域の量（メガバイト） |
| MAXIMUM_ACTIVE_THREADS | INTEGER | プール内で同時にアクティブにできるスレッドの最大数 |
| SUBSYSTEM_NAME | VARCHAR(10) | このストレージ・プールが関連付けられているサブシステム |
| ELAPSED_TIME | INTEGER | 測定開始時刻からの経過時間（秒） |
| ELAPSED_DATABASE_FAULTS | DECIMAL(10,1) | データベース・ページ・フォールトの率（ページ・フォールト/秒） |
| ELAPSED_NON_DATABASE_FAULTS | DECIMAL(10,1) | 非データベース・ページ・フォールトの率（ページ・フォールト/秒） |
| ELAPSED_TOTAL_FAULTS | DECIMAL(10,1) | 総ページ・フォールトの率（ページ・フォールト/秒） |

### 使用例

```sql
-- メモリー・プール情報を表示
SELECT * FROM TABLE(QSYS2.MEMORY_POOL());

-- 統計をリセットしてメモリー・プール情報を取得
SELECT * FROM TABLE(QSYS2.MEMORY_POOL(RESET_STATISTICS => 'YES'));

-- 特定のプールの情報を表示
SELECT POOL_NAME, CURRENT_SIZE, MAXIMUM_ACTIVE_THREADS, ELAPSED_TOTAL_FAULTS
FROM TABLE(QSYS2.MEMORY_POOL())
WHERE POOL_NAME = '*BASE';

-- ページ・フォールト率が高いプールを特定
SELECT POOL_NAME, CURRENT_SIZE, ELAPSED_TOTAL_FAULTS
FROM TABLE(QSYS2.MEMORY_POOL())
WHERE ELAPSED_TOTAL_FAULTS > 10
ORDER BY ELAPSED_TOTAL_FAULTS DESC;
```

## MEMORY_POOL_INFO ビュー

MEMORY_POOL_INFO ビューは、メモリー・プールに関する情報を返します。

**権限:** 不要。

**主要カラム:**

| カラム名 | システムカラム名 | データ型 | 説明 |
| --- | --- | --- | --- |
| SYSTEM_POOL_ID | POOL_ID | INTEGER | システム・ストレージ・プールのプール識別子 |
| POOL_NAME | POOL_NAME | VARCHAR(10) | ストレージ・プールの名前 |
| CURRENT_SIZE | CURR_SIZE | DECIMAL(20,2) | プール内の主記憶域の量（メガバイト） |
| RESERVED_SIZE | RSVD_SIZE | DECIMAL(10,2) | システム用に予約されたストレージ量（メガバイト） |
| DEFINED_SIZE | DFND_SIZE | DECIMAL(20,2) | 定義されたプールのサイズ（メガバイト） |
| MAXIMUM_ACTIVE_THREADS | MAX_THREAD | INTEGER | 同時にアクティブにできるスレッドの最大数 |
| CURRENT_THREADS | CURR_THRD | INTEGER | 現在プールを使用しているスレッド数 |
| CURRENT_INELIGIBLE_THREADS | INEL_THRD | INTEGER | プール内の非適格スレッド数 |
| PAGING_OPTION | PAGE_OPT | VARCHAR(10) | ページング特性の動的調整（*FIXED、*CALC、USRDFN） |

### 使用例

```sql
-- メモリー・プール情報を表示
SELECT * FROM QSYS2.MEMORY_POOL_INFO;

-- プールのサイズとスレッド情報を表示
SELECT POOL_NAME, CURRENT_SIZE, DEFINED_SIZE, 
       CURRENT_THREADS, MAXIMUM_ACTIVE_THREADS
FROM QSYS2.MEMORY_POOL_INFO
ORDER BY CURRENT_SIZE DESC;

-- 非適格スレッドが多いプールを特定
SELECT POOL_NAME, CURRENT_THREADS, CURRENT_INELIGIBLE_THREADS,
       DECIMAL(CURRENT_INELIGIBLE_THREADS, 5, 2) / CURRENT_THREADS * 100 AS INELIGIBLE_PCT
FROM QSYS2.MEMORY_POOL_INFO
WHERE CURRENT_THREADS > 0
ORDER BY INELIGIBLE_PCT DESC;

-- ページング設定を確認
SELECT POOL_NAME, PAGING_OPTION, CURRENT_SIZE, ELAPSED_TOTAL_FAULTS
FROM QSYS2.MEMORY_POOL_INFO
WHERE PAGING_OPTION = '*CALC';
```

## OPEN_FILES テーブル関数

OPEN_FILES テーブル関数は、ジョブで開いているファイルに関する情報を返します。

**権限:** 不要。

```
>>-- OPEN_FILES --(
  JOB_NAME => job-name
)-->>
```
スキーマは QSYS2 です。

**パラメータ:**
- `JOB_NAME`: ジョブ名（'*' で現行ジョブ、または修飾ジョブ名）

**主要カラム:**

| カラム名 | データ型 | 説明 |
| --- | --- | --- |
| LIBRARY_NAME | VARCHAR(10) | 開いているファイルを含むライブラリーの名前 |
| FILE_NAME | VARCHAR(10) | 開いているファイルの名前 |
| FILE_TYPE | VARCHAR(7) | ファイルのタイプ（PF、LF、DSPF、PRTF など） |
| MEMBER_NAME | VARCHAR(10) | データベース・メンバーの名前 |
| DEVICE_NAME | VARCHAR(10) | プログラム・デバイスの名前 |
| OPEN_OPTION | VARCHAR(6) | オープン操作のタイプ（ALL、INPUT、OUTPUT） |
| WRITE_COUNT | BIGINT | 成功した書き込み操作の数 |
| READ_COUNT | BIGINT | 成功した読み取り操作の数 |

### 使用例

```sql
-- 現行ジョブで開いているファイルを表示
SELECT * FROM TABLE(QSYS2.OPEN_FILES('*'));

-- 特定のジョブで開いているファイルを表示
SELECT * FROM TABLE(QSYS2.OPEN_FILES('123456/QUSER/QZDASOINIT'));

-- データベース・ファイルのみを表示
SELECT LIBRARY_NAME, FILE_NAME, MEMBER_NAME, OPEN_OPTION, 
       READ_COUNT, WRITE_COUNT
FROM TABLE(QSYS2.OPEN_FILES('*'))
WHERE FILE_TYPE IN ('PF', 'LF')
ORDER BY READ_COUNT + WRITE_COUNT DESC;

-- I/O 操作が多いファイルを特定
SELECT LIBRARY_NAME, FILE_NAME, 
       READ_COUNT + WRITE_COUNT AS TOTAL_IO
FROM TABLE(QSYS2.OPEN_FILES('*'))
WHERE FILE_TYPE IN ('PF', 'LF')
ORDER BY TOTAL_IO DESC
FETCH FIRST 10 ROWS ONLY;
```

## パフォーマンス監視のベストプラクティス

### メモリープールの監視

```sql
-- メモリープールの健全性チェック
SELECT POOL_NAME, 
       CURRENT_SIZE,
       MAXIMUM_ACTIVE_THREADS,
       CURRENT_THREADS,
       ELAPSED_TOTAL_FAULTS,
       CASE 
         WHEN ELAPSED_TOTAL_FAULTS > 20 THEN 'HIGH'
         WHEN ELAPSED_TOTAL_FAULTS > 10 THEN 'MEDIUM'
         ELSE 'LOW'
       END AS FAULT_LEVEL
FROM QSYS2.MEMORY_POOL_INFO
WHERE POOL_NAME NOT LIKE '*MACHINE'
ORDER BY ELAPSED_TOTAL_FAULTS DESC;
```

### プール使用率の分析

```sql
-- プールの使用率を計算
SELECT POOL_NAME,
       CURRENT_SIZE,
       DEFINED_SIZE,
       CURRENT_THREADS,
       MAXIMUM_ACTIVE_THREADS,
       DECIMAL(CURRENT_THREADS, 5, 2) / MAXIMUM_ACTIVE_THREADS * 100 AS THREAD_UTILIZATION_PCT
FROM QSYS2.MEMORY_POOL_INFO
WHERE MAXIMUM_ACTIVE_THREADS > 0
ORDER BY THREAD_UTILIZATION_PCT DESC;
```

## 関連ビュー・関数

- [`SYSTEM_STATUS`](ibmi-system-status.md) - システム全体の状況
- [`JOB_INFO`](ibmi-job-management-part1.md) - ジョブ情報
- [`OBJECT_LOCK_INFO`](ibmi-locks-records.md) - オブジェクト・ロック情報