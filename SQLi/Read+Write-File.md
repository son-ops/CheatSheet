# Contents
* [Read File](#read-file)
* [Write File](#write-file)

## Read File

| DBMS | Hàm / Primitive | Cách dùng | Quyền / điều kiện |
|---|---|---|---|
| MySQL | `LOAD_FILE()` | `SELECT LOAD_FILE('/etc/passwd');` | Cần quyền `FILE`; tiến trình DB phải có quyền đọc file; có thể bị giới hạn bởi `secure_file_priv` |
| MSSQL | `OPENROWSET(BULK ...)` | `SELECT * FROM OPENROWSET(BULK 'C:\Windows\win.ini', SINGLE_CLOB) AS x;` | Cần `ADMINISTER BULK OPERATIONS` hoặc `ADMINISTER DATABASE BULK OPERATIONS` |
| PostgreSQL | `pg_read_file()` | `SELECT pg_read_file('/etc/passwd', 0, 200);` | Thường cần superuser hoặc role đọc file server |
| PostgreSQL | `pg_ls_dir()` | `SELECT pg_ls_dir('/tmp');` | Thường cần superuser hoặc role phù hợp |
| PostgreSQL | `COPY FROM` | `COPY temp FROM '/etc/passwd';` | Cần quyền đọc file phía server; thường quyền cao |
| PostgreSQL | `lo_import()` | `SELECT lo_import('/etc/passwd');` | Thường cần quyền cao / superuser |
| Oracle | `UTL_FILE.FOPEN` + `UTL_FILE.GET_LINE` | `SELECT utl_file.get_line(utl_file.fopen('DIR_OBJ','file.txt','R'), 32767) FROM dual;` | Cần quyền dùng `UTL_FILE` và quyền trên `DIRECTORY` object |
| SQLite | N/A | `N/A` | Không hỗ trợ mặc định |

### Xem quyền / kiểm tra

#### MySQL

```sql
SHOW GRANTS FOR CURRENT_USER();
SHOW VARIABLES LIKE 'secure_file_priv';
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

## Write File

| DBMS | Hàm / Primitive | Cách dùng | Quyền / điều kiện |
|---|---|---|---|
| MySQL | `INTO OUTFILE` | `SELECT 'test' INTO OUTFILE '/tmp/test.txt';` | Cần quyền `FILE`; tiến trình DB phải có quyền ghi; thường không ghi đè file có sẵn |
| MySQL | `INTO DUMPFILE` | `SELECT 'test' INTO DUMPFILE '/tmp/test.bin';` | Cần quyền `FILE`; tiến trình DB phải có quyền ghi |
| MSSQL | Stored procedure ghi file | `EXEC spWriteStringToFile 'contents', 'C:\path\to\', 'file.txt';` | Procedure phải tồn tại và user phải có quyền `EXECUTE` |
| PostgreSQL | `COPY TO` | `COPY (SELECT 'test') TO '/tmp/test.txt';` | Thường cần quyền cao / superuser |
| PostgreSQL | `lo_export()` | `SELECT lo_export(16420, '/tmp/testexport');` | Thường cần quyền cao / superuser |
| Oracle | `UTL_FILE.PUT_LINE` | `BEGIN f := utl_file.fopen('DIR_OBJ','file.txt','W'); utl_file.put_line(f,'test'); utl_file.fclose(f); END;` | Cần quyền `UTL_FILE` và quyền ghi vào `DIRECTORY` object |
| SQLite | `writefile()` | `SELECT writefile('/tmp/test.txt', 'data');` | Chỉ có khi build/extension hỗ trợ |
| SQLite | `ATTACH DATABASE` | `ATTACH DATABASE '/tmp/test.db' AS testdb;` | Ứng dụng phải cho phép ghi vào path đó |

### Xem quyền / kiểm tra

#### MySQL

```sql
SHOW GRANTS FOR CURRENT_USER();
SHOW VARIABLES LIKE 'secure_file_priv';
```

#### MSSQL

```sql
SELECT * FROM fn_my_permissions(NULL, 'SERVER');
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');
SELECT OBJECT_ID('spWriteStringToFile');
```

#### PostgreSQL

```sql
SELECT rolsuper, rolname
FROM pg_roles
WHERE rolname = current_user;

SELECT pg_has_role(current_user, 'pg_write_server_files', 'member');
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
PRAGMA database_list;
```