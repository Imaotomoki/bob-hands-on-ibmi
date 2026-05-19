# IBM i システム・ヘルス・サービス

これらのサービスは、システムの健全性に関する情報を提供します。

## SYSLIMITS ビュー

SYSLIMITS ビューは、システム限界に対する現在の使用状況を返します。システム・リソースの使用率が限界に近づいているかどうかを監視するために使用できます。

**権限:** 不要。

**Table 282. SYSLIMITS view**

| カラム名 | システムカラム名 | データ型 | 説明 |
| --- | --- | --- | --- |
| LAST_CHANGE_TIMESTAMP | LASTCHG | TIMESTAMP | この行が最後に変更されたタイムスタンプ。 |
| LIMIT_CATEGORY | CATEGORY | VARCHAR(15) | この限界のカテゴリー。DATABASE、JOURNAL、SECURITY、MISCELLANEOUS、WORK MANAGEMENT、FILE SYSTEM、SAVE RESTORE、CLUSTER、COMMUNICATION。 |
| LIMIT_TYPE | TYPE | VARCHAR(7) | 限界のタイプ。OBJECT、JOB、SYSTEM、ASP。 |
| SIZING_NAME | SIZING_NAM | VARCHAR(128) | サイジング ID に対応する名前。 |
| COMMENTS | COMMENTS | VARCHAR(2000) Nullable | 限界の説明。 |
| USER_NAME | CURUSER | VARCHAR(10) | この行がログ記録されたときに有効なユーザー名。 |
| CURRENT_VALUE | CURVAL | BIGINT | この限界の報告値。 |
| MAXIMUM_VALUE | MAXVAL | DECIMAL(21,0) Nullable | この限界に許可される最大値。 |
| JOB_NAME | JOB_NAME | VARCHAR(28) | この行がログ記録されたときのジョブ名。ジョブがアクティブでない場合は NULL 値を含みます。 |
| JOB_STATUS | JOB_STATUS | CHAR(10) Nullable | ジョブの状況。ジョブがアクティブでない場合は NULL 値を含みます。 |
| ACTIVE_JOB_STATUS | AJSTATUS | CHAR(4) Nullable | ジョブの初期スレッドのアクティブ状況。ジョブが遷移中またはアクティブでない場合は NULL 値を含みます。 |
| RUN_PRIORITY | RUNPRI | INTEGER Nullable | このジョブ内の任意のスレッドに許可される最高実行優先順位。ジョブがアクティブでない場合は NULL 値を含みます。 |
| SBS_NAME | SBS_NAME | CHAR(10) Nullable | ジョブが実行されているサブシステム名。ジョブがアクティブでない場合は NULL 値を含みます。 |
| CPU_USED | CPU_USED | BIGINT Nullable | このジョブが現在使用している CPU 時間（ミリ秒）。ジョブがアクティブでない場合は NULL 値を含みます。 |
| TEMP_STORAGE_USED_MB | TEMPSTG | INTEGER Nullable | このジョブに現在割り振られている補助記憶域（MB）。ジョブがアクティブでない場合は NULL 値を含みます。 |
| AUX_IO_REQUESTED | AUXIO | BIGINT Nullable | ジョブがすべてのルーティング・ステップで実行した補助 I/O 要求の数。ジョブがアクティブでない場合は NULL 値を含みます。 |
| PAGE_FAULTS | PAGEFAULT | BIGINT Nullable | 指定ジョブの現行ルーティング・ステップ中にアクティブ・プログラムが主記憶域にないアドレスを参照した回数。ジョブがアクティブでない場合は NULL 値を含みます。 |
| CLIENT_WRKSTNNAME | CLIENTWRK | CHAR(255) Nullable | SQL CLIENT_WRKSTNNAME 特殊レジスターの値。ジョブがアクティブでない場合は NULL 値を含みます。 |
| CLIENT_APPLNAME | CLIENTAPP | CHAR(255) Nullable | SQL CLIENT_APPLNAME 特殊レジスターの値。ジョブがアクティブでない場合は NULL 値を含みます。 |
| CLIENT_ACCTNG | CLIENTACT | CHAR(255) Nullable | SQL CLIENT_ACCTNG 特殊レジスターの値。ジョブがアクティブでない場合は NULL 値を含みます。 |
| CLIENT_PROGRAMID | CLIENTPGM | CHAR(255) Nullable | SQL CLIENT_PROGRAMID 特殊レジスターの値。ジョブがアクティブでない場合は NULL 値を含みます。 |
| CLIENT_USERID | CLIENTUSER | CHAR(255) Nullable | SQL CLIENT_USERID 特殊レジスターの値。ジョブがアクティブでない場合は NULL 値を含みます。 |
| SQL_STATEMENT_TEXT | SQLSTMT | VARCHAR(10000) Nullable | 最後に実行された SQL ステートメントまたは現在実行中の SQL ステートメントのテキスト。ジョブがアクティブでない場合は NULL 値を含みます。 |
| SCHEMA_NAME | OBJ_SCHEMA | VARCHAR(128) Nullable | このオブジェクトの SQL スキーマ名。スキーマ名がない場合は NULL 値を含みます。 |
| OBJECT_NAME | OBJ_NAME | VARCHAR(128) Nullable | オブジェクトの SQL 名。オブジェクト名がないか SQL 名が返せない場合は NULL 値を含みます。 |
| SYSTEM_SCHEMA_NAME | SYS_NAME | VARCHAR(10) Nullable | オブジェクトのライブラリー名。ライブラリー名がない場合は NULL 値を含みます。 |
| SYSTEM_OBJECT_NAME | SYS_ONAME | VARCHAR(30) Nullable | この行のオブジェクト名。オブジェクト名がない場合は NULL 値を含みます。 |
| SYSTEM_TABLE_MEMBER | SYS_MNAME | VARCHAR(10) Nullable | データベース・メンバーに固有のオブジェクト限界のメンバー名。メンバー限界用でない場合は NULL 値を含みます。 |
| IFS_PATH_NAME | PATHNAME | CLOB Nullable | オブジェクトの統合ファイル・システム・パス。パスがない場合は NULL 値を含みます。 |
| OBJECT_TYPE | OBJTYPE | VARCHAR(7) Nullable | SYSTEM_SCHEMA_NAME/SYSTEM_OBJECT_NAME カラムにオブジェクト名がログ記録されているときの IBM i オブジェクト・タイプ。指定なしの場合は NULL 値を含みます。 |
| SQL_OBJECT_TYPE | SQLOBJTYPE | VARCHAR(9) Nullable | オブジェクトの SQL タイプ。ALIAS、FUNCTION、INDEX、PACKAGE、PROCEDURE、ROUTINE、SEQUENCE、TABLE、TRIGGER、TYPE、VARIABLE、VIEW、XSR。SQL オブジェクトでないか指定なしの場合は NULL 値を含みます。 |
| ASP_NUMBER | ASPNUM | SMALLINT Nullable | この行に関連する ASP 番号。ASP 番号がない場合は NULL 値を含みます。 |
| SIZING_ID | SIZING_ID | INTEGER | この限界の一意の識別子。値は QSYS2.SQL_SIZING テーブルの SIZING_ID カラムで管理されます。 |

### 例

システム限界情報を表示する:

```sql
SELECT * FROM QSYS2.SYSLIMITS;
```

## SYSLIMITS_BASIC ビュー

SYSLIMITS_BASIC ビューは、SYSLIMITS ビューのサブセットで、パフォーマンスが重要な場合に使用します。

**権限:** 不要。

**Table 283. SYSLIMITS_BASIC view**

| カラム名 | システムカラム名 | データ型 | 説明 |
| --- | --- | --- | --- |
| LAST_CHANGE_TIMESTAMP | LASTCHG | TIMESTAMP | この行が最後に変更されたタイムスタンプ。 |
| LIMIT_CATEGORY | CATEGORY | VARCHAR(15) | この限界のカテゴリー。 |
| LIMIT_TYPE | TYPE | VARCHAR(7) | 限界のタイプ。OBJECT、JOB、SYSTEM、ASP。 |
| SIZING_NAME | SIZING_NAM | VARCHAR(128) | サイジング ID に対応する名前。 |
| COMMENTS | COMMENTS | VARCHAR(2000) Nullable | 限界の説明。 |
| USER_NAME | CURUSER | VARCHAR(10) | ログ記録時の有効ユーザー名。 |
| CURRENT_VALUE | CURVAL | BIGINT | この限界の報告値。 |
| MAXIMUM_VALUE | MAXVAL | DECIMAL(21,0) Nullable | この限界に許可される最大値。 |
| JOB_NAME | JOB_NAME | VARCHAR(28) | ログ記録時のジョブ名。ジョブがアクティブでない場合は NULL 値を含みます。 |
| SYSTEM_SCHEMA_NAME | SYS_NAME | VARCHAR(10) Nullable | オブジェクトのライブラリー名。ライブラリー名がない場合は NULL 値を含みます。 |
| SYSTEM_OBJECT_NAME | SYS_ONAME | VARCHAR(30) Nullable | この行のオブジェクト名。オブジェクト名がない場合は NULL 値を含みます。 |
| SYSTEM_TABLE_MEMBER | SYS_MNAME | VARCHAR(10) Nullable | メンバー限界用のメンバー名。該当しない場合は NULL 値を含みます。 |
| IFS_PATH_NAME | PATHNAME | CLOB Nullable | オブジェクトの統合ファイル・システム・パス。該当しない場合は NULL 値を含みます。 |
| OBJECT_TYPE | OBJTYPE | VARCHAR(7) Nullable | IBM i オブジェクト・タイプ。該当しない場合は NULL 値を含みます。 |
| ASP_NUMBER | ASPNUM | SMALLINT Nullable | ASP 番号。該当しない場合は NULL 値を含みます。 |

### 例

基本システム限界情報を表示する:

```sql
SELECT * FROM QSYS2.SYSLIMITS_BASIC;
```

## PROCESS_SYSTEM_LIMITS_ALERTS プロシージャー

PROCESS_SYSTEM_LIMITS_ALERTS プロシージャーは、システム限界に対するアラート処理を実行します。設定された閾値を超えるシステム限界についてアラート・メッセージを送信します。

**権限:** 不要。