> Khả năng hỗ trợ multi-query phụ thuộc vào DBMS, ngôn ngữ lập trình và driver/API cụ thể
# PHP
| DBMS | Driver / API | Multi-query |
|---|---|---|
| MySQL | `mysqli::query()` | Không hỗ trợ |
| MySQL | `mysqli::multi_query()` | Có |
| MySQL | `PDO_MYSQL::query()` / `PDO_MYSQL::prepare()` | Có điều kiện: hỗ trợ nếu connection không bị tạo với `Pdo\Mysql::ATTR_MULTI_STATEMENTS => false` |
| PostgreSQL | `pg_query()` | Có |
| PostgreSQL | `pg_query_params()` | Không hỗ trợ |
| PostgreSQL | `pg_prepare()` + `pg_execute()` | Không hỗ trợ |
| MSSQL | `sqlsrv_query()` | Có |
| MSSQL | `sqlsrv_prepare()` + `sqlsrv_execute()` | Có |
| Oracle | `oci_parse()` + `oci_execute()` | Không hỗ trợ |
| SQLite | `SQLite3::query()` | Không hỗ trợ |
| SQLite | `SQLite3::exec()` | Có |

--- 

# Python
| DBMS | Driver / API | Multi-query |
|---|---|---|
| SQLite | `sqlite3.Cursor.execute()` | Không hỗ trợ |
| SQLite | `sqlite3.Cursor.executescript()` | Có |
| MySQL | `mysql.connector.cursor.MySQLCursor.execute()` | Có |
| PostgreSQL | `psycopg Cursor.execute()` | Có |
| MSSQL | `pyodbc.Cursor.execute()` | Có |
| Oracle | `oracledb.Cursor.execute()` | Không hỗ trợ |

---

## Lưu ý:
với MSSQL, dấu `;` thường được khuyến nghị nhưng không bắt buộc cho đa số T-SQL statements, nên một số stacked query vẫn có thể hoạt động dù không có `;`. Tuy nhiên, không phải mọi cú pháp đều bỏ được `;`.

Ví dụ: 
- multiple SELECT statements: `SELECT 'A'SELECT 'B'SELECT 'C'`

- updating password with a stacked query: `SELECT id, username, password FROM users WHERE username = 'admin'exec('update[users]set[password]=''a''')--`


## Ví dụ nhanh

**PHP + MySQL qua `mysqli`**
- Không hỗ trợ multi-query:
  `$mysqli->query("SELECT 1; SELECT 2");`
- Hỗ trợ multi-query:
  `$mysqli->multi_query("SELECT 1; SELECT 2");`

**PHP + MySQL qua `PDO_MYSQL`**
- Có điều kiện:
  `$pdo->query("SELECT 1; SELECT 2");`
- Chỉ hoạt động nếu connection **không** được tạo với:
  `Pdo\Mysql::ATTR_MULTI_STATEMENTS => false`

**PHP + PostgreSQL qua `pg_query()`**
- Hỗ trợ multi-query:
  `pg_query($conn, "SELECT 1; SELECT 2");`

**PHP + PostgreSQL qua `pg_query_params()`**
- Không hỗ trợ multi-query:
  `pg_query_params($conn, "SELECT 1; SELECT 2", []);`

**PHP + MSSQL qua `sqlsrv_query()`**
- Hỗ trợ multi-query:
  `sqlsrv_query($conn, "SELECT 1; SELECT 2");`

**PHP + Oracle qua OCI8**
- Không hỗ trợ stacked query thông thường:
  `$stmt = oci_parse($conn, "SELECT 1; SELECT 2");`

**PHP + SQLite qua `SQLite3::query()`**
- Không hỗ trợ multi-query:
  `$db->query("SELECT 1; SELECT 2");`

**PHP + SQLite qua `SQLite3::exec()`**
- Hỗ trợ multi-query:
  `$db->exec("CREATE TABLE t(id INTEGER); INSERT INTO t VALUES (1);");`

**Python + SQLite qua `sqlite3`**
- Không hỗ trợ multi-query:
  `cur.execute("SELECT 1; SELECT 2;")`
- Hỗ trợ multi-query:
  `cur.executescript("SELECT 1; SELECT 2;")`

**Python + MySQL qua `mysql.connector`**
- Hỗ trợ multi-query:
  `cursor.execute("SELECT 1; SELECT 2")`

**Python + PostgreSQL qua `psycopg`**
- Có thể chạy:
  `cur.execute("SELECT 1; SELECT 2")`
- Nhưng không hợp lệ theo kiểu prepared statement:
  `cur.execute("SELECT %s; SELECT %s", (1, 2))`

**Python + MSSQL qua `pyodbc`**
- Hỗ trợ multi-query:
  `cursor.execute("SELECT 1; SELECT 2")`

**Python + Oracle qua `python-oracledb`**
- Không hỗ trợ stacked query thông thường:
  `cursor.execute("SELECT 1; SELECT 2")`