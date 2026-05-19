# IBM i ワーク・マネジメント・サービス - ジョブ管理（パート1）

これらのサービスは、ジョブ、サブシステム、メモリー・プール、システム状況に関する情報を提供します。

## ACTIVE_JOB_INFO テーブル関数

ACTIVE_JOB_INFO テーブル関数は、アクティブ・ジョブに関する包括的な情報を返します。

返される情報は、活動ジョブの処理（WRKACTJOB）CLコマンドで表示できる情報と類似しています。

**権限:** 呼び出し元のユーザー・プロファイルが情報を返すジョブのジョブ・ユーザーIDと同じ場合は不要。それ以外の場合は *JOBCTL 特殊権限が必要です。

```
>>-- ACTIVE_JOB_INFO --(
  JOB_NAME_FILTER => job-name-filter,
  SUBSYSTEM_LIST_FILTER => subsystem-list-filter,
  CURRENT_USER_LIST_FILTER => current-user-list-filter,
  DETAILED_INFO => detailed-info,
  RESET_STATISTICS => reset-statistics
)-->>
```
スキーマは QSYS2 です。

**Table 284. ACTIVE_JOB_INFO table function**

DETAILED_INFO カラムは、カラムが返される条件（NONE/WORK/QTEMP/FULL/ALL）を示します。

主要なカラム（一部抜粋）:

| カラム名 | データ型 | DETAILED_INFO | 説明 |
| --- | --- | --- | --- |
| JOB_NAME | VARCHAR(28) | NONE WORK QTEMP FULL ALL | 修飾ジョブ名。 |
| JOB_USER | VARCHAR(10) | NONE WORK QTEMP FULL ALL | ジョブを開始したユーザー・プロファイル。 |
| JOB_NUMBER | VARCHAR(6) | NONE WORK QTEMP FULL ALL | ジョブのジョブ番号。 |
| SUBSYSTEM | VARCHAR(10) | NONE WORK QTEMP FULL ALL | ジョブが実行されているサブシステム名。 |
| JOB_TYPE | VARCHAR(3) | NONE WORK QTEMP FULL ALL | アクティブ・ジョブのタイプ。ASJ、BCH、BCI、EVK、INT、M36、MRT、PDJ、PJ、RDR、SBS、SYS、WTR。 |
| JOB_STATUS | VARCHAR(4) | NONE WORK QTEMP FULL ALL | ジョブの初期スレッドの状況。CMNW、CNDW、DEQW、DLYW、DSPW、END、EOJ、EVTW、HLD、JVAW、LCKW、LSPW、MSGW、MTXW、PSRW、RUN、SEMW、THDW など。 |
| CPU_TIME | DECIMAL(20,0) | NONE WORK QTEMP FULL ALL | ジョブが使用した処理装置時間の合計（ミリ秒）。 |
| TEMPORARY_STORAGE | INTEGER | NONE WORK QTEMP FULL ALL | このジョブに現在割り振られている一時ストレージの量（MB）。 |
| TOTAL_DISK_IO_COUNT | DECIMAL(20,0) | NONE WORK QTEMP FULL ALL | ジョブがすべてのルーティング・ステップで実行したディスク I/O 操作の合計数。 |
| SQL_STATEMENT_TEXT | VARCHAR(10000) | FULL ALL | 最後に実行された SQL ステートメントまたは現在実行中の SQL ステートメントのテキスト。 |
| SQL_STATEMENT_STATUS | VARCHAR(8) | FULL ALL | このジョブ内の SQL の状況。ACTIVE、COMPLETE。 |

### 例

すべてのアクティブ・ジョブを表示する:

```sql
SELECT * FROM TABLE(QSYS2.ACTIVE_JOB_INFO());
```

特定のサブシステムのジョブを表示する:

```sql
SELECT * FROM TABLE(QSYS2.ACTIVE_JOB_INFO(
    SUBSYSTEM_LIST_FILTER => 'QINTER'));
```

CPU使用率が高いジョブを表示する:

```sql
SELECT JOB_NAME, JOB_USER, CPU_TIME, TEMPORARY_STORAGE, SQL_STATEMENT_TEXT
FROM TABLE(QSYS2.ACTIVE_JOB_INFO(DETAILED_INFO => 'FULL'))
WHERE CPU_TIME > 10000
ORDER BY CPU_TIME DESC;
```

## END_JOBS プロシージャー

END_JOBS プロシージャーは、1つ以上のジョブを終了します。

**権限:** 不要。

```
>>-- END_JOBS --(
  JOB_NAME => job-name,
  OPTION => option,
  DELAY_TIME => delay-time,
  DUPLICATE_JOB_OPTION => duplicate-job-option
)-->>
```
スキーマは QSYS2 です。

## ENDED_JOB_INFO テーブル関数

ENDED_JOB_INFO テーブル関数は、終了したジョブに関する情報を返します。

**権限:** 不要。

```
>>-- ENDED_JOB_INFO --(
  JOB_NAME_FILTER => job-name-filter,
  IGNORE_ERRORS => ignore-errors
)-->>
```
スキーマは QSYS2 です。

**Table 287. ENDED_JOB_INFO table function**

| カラム名 | データ型 | 説明 |
| --- | --- | --- |
| MESSAGE_TIMESTAMP | TIMESTAMP | CPF1164 メッセージが送信されたタイムスタンプ。 |
| FROM_JOB | VARCHAR(28) | CPF1164 メッセージが送信された修飾ジョブ名。 |
| JOB_INTERFACE | VARCHAR(22) | ジョブが使用したインターフェース。 |
| CPU_TIME | DECIMAL(15,3) | ジョブが使用した処理装置時間の合計（秒）。 |
| JOB_END_CODE | SMALLINT | ジョブ終了コード。0（正常完了）、10（制御終了中の正常完了）、20（ENDSEV ジョブ属性超過）、30（異常終了）など。 |
| JOB_TYPE | VARCHAR(11) | ジョブのタイプ。AUTOSTART、BATCH、INTERACTIVE、MONITOR、READER、SCPF、SYSTEM、WRITER。 |

### 例

最近終了したジョブを表示する:

```sql
SELECT * FROM TABLE(QSYS2.ENDED_JOB_INFO());
```

異常終了したジョブを表示する:

```sql
SELECT FROM_JOB, JOB_END_CODE, JOB_END_DETAIL, CPU_TIME
FROM TABLE(QSYS2.ENDED_JOB_INFO())
WHERE JOB_END_CODE >= 30
ORDER BY MESSAGE_TIMESTAMP DESC;
```

## GET_JOB_INFO テーブル関数

GET_JOB_INFO テーブル関数は、特定のジョブに関する情報を返します。

**権限:** 不要。

```
>>-- GET_JOB_INFO --(
  JOB_NAME => job-name
)-->>
```
スキーマは QSYS2 です。

### 例

現行ジョブの情報を取得する:

```sql
SELECT * FROM TABLE(QSYS2.GET_JOB_INFO('*'));
```

特定のジョブの情報を取得する:

```sql
SELECT * FROM TABLE(QSYS2.GET_JOB_INFO('123456/QUSER/QZDASOINIT'));
```

## JOB_INFO テーブル関数

JOB_INFO テーブル関数は、ジョブに関する情報を返します。WRKJOBと同様の機能を提供します。

**権限:** 不要。

```
>>-- JOB_INFO --(
  JOB_STATUS_FILTER => job-status-filter,
  JOB_TYPE_FILTER => job-type-filter,
  JOB_SUBSYSTEM_FILTER => job-subsystem-filter,
  JOB_USER_FILTER => job-user-filter
)-->>
```
スキーマは QSYS2 です。

主要なカラム:

| カラム名 | データ型 | 説明 |
| --- | --- | --- |
| JOB_NAME | VARCHAR(28) | 修飾ジョブ名。 |
| JOB_STATUS | VARCHAR(6) | ジョブの状況。ACTIVE、JOBQ、OUTQ。 |
| JOB_TYPE | VARCHAR(3) | ジョブのタイプ。ASJ、BCH、BCI、EVK、INT、M36、MRT、PDJ、PJ、RDR、SBS、SYS、WTR。 |
| JOB_SUBSYSTEM | VARCHAR(10) | ジョブのサブシステム名。 |
| JOB_ENTERED_SYSTEM_TIME | TIMESTAMP(0) | ジョブがシステムに入れられたタイムスタンプ。 |
| JOB_ACTIVE_TIME | TIMESTAMP(0) | ジョブがシステム上で実行を開始した時刻。 |

### 例

すべてのバッチ・ジョブを表示する:

```sql
SELECT * FROM TABLE(QSYS2.JOB_INFO(
    JOB_TYPE_FILTER => '*BATCH'));
```

特定ユーザーのアクティブジョブを表示する:

```sql
SELECT JOB_NAME, JOB_STATUS, JOB_TYPE, JOB_SUBSYSTEM
FROM TABLE(QSYS2.JOB_INFO(
    JOB_STATUS_FILTER => 'ACTIVE',
    JOB_USER_FILTER => 'QUSER'));
```

## JOB_LOCK_INFO テーブル関数

JOB_LOCK_INFO テーブル関数は、ジョブが保持しているロックに関する情報を返します。

**権限:** 不要。

```
>>-- JOB_LOCK_INFO --(
  JOB_NAME => job-name
)-->>
```
スキーマは QSYS2 です。

### 例

現行ジョブのロック情報を表示する:

```sql
SELECT * FROM TABLE(QSYS2.JOB_LOCK_INFO('*'));
```

特定オブジェクトのロック状況を確認する:

```sql
SELECT JOB_NAME, LOCK_STATE, LOCK_STATUS, OBJECT_LIBRARY, OBJECT_NAME
FROM TABLE(QSYS2.JOB_LOCK_INFO('*'))
WHERE OBJECT_NAME = 'MYFILE';