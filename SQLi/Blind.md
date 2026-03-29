# Blind SQLi Cheatsheet

# Contents
* [Character extraction](#character-extraction)
* [Conditional / branching](#conditional--branching)
* [Time delay](#time-delay)

---

## Character extraction

| DBMS | Primitive | Syntax khung |
|---|---|---|
| MySQL | `SUBSTR()` | `SUBSTR(<expr>, <pos>, <len>)` |
| MySQL | `SUBSTRING()` | `SUBSTRING(<expr>, <pos>, <len>)` |
| MySQL | `MID()` | `MID(<expr>, <pos>, <len>)` |
| MySQL | `LEFT()` | `LEFT(<expr>, <len>)`  (Lấy N ký tự từ bên trái) | 
| MySQL | `RIGHT()` | `RIGHT(<expr>, <len>)` (Lấy N ký tự từ bên phải)|
| MSSQL | `SUBSTRING()` | `SUBSTRING(<expr>, <pos>, <len>)` | 
| PostgreSQL | `SUBSTR()` | `SUBSTR(<expr>, <pos>, <len>)` | 
| PostgreSQL | `SUBSTRING()` | `SUBSTRING(<expr>, <pos>, <len>)` | 
| PostgreSQL | `SUBSTRING ... FROM ... FOR` | `SUBSTRING(<expr> FROM <pos> FOR <len>)` |
| SQLite | `SUBSTR()` | `SUBSTR(<expr>, <pos>, <len>)` |
| SQLite | `SUBSTRING()` | `SUBSTRING(<expr>, <pos>, <len>)` | 
| Oracle | `SUBSTR()` | `SUBSTR(<expr>, <pos>, <len>)` | 

---

## Conditional / branching

| DBMS | Primitive | Syntax khung |
|---|---|---|
| MySQL | `IF()` | `IF(<condition>, <true_expr>, <false_expr>)` | 
| MySQL | `CASE WHEN` | `CASE WHEN <condition> THEN <a> ELSE <b> END` |
| MySQL | `MAKE_SET()` | `MAKE_SET(<cond>, <value>)` | 
| MSSQL | `IF ... ELSE` | `IF <condition> <true_stmt> ELSE <false_stmt>` | 
| MSSQL | `CASE WHEN` | `CASE WHEN <condition> THEN <a> ELSE <b> END` | 
| PostgreSQL | `CASE WHEN` | `CASE WHEN <condition> THEN <a> ELSE <b> END` | 
| Oracle | `CASE WHEN` | `CASE WHEN <condition> THEN <a> ELSE <b> END` |

---

## Time delay

| DBMS | Primitive | Syntax khung | Giải thích |
|---|---|---|---|
| MySQL | `SLEEP()` | `SLEEP(<sec>)` | Delay trực tiếp theo số giây |
| MySQL | `BENCHMARK()` | `BENCHMARK(<count>, <expr>)` | Tạo chậm bằng lặp tính toán nhiều lần |
| MSSQL | `WAITFOR DELAY` | `WAITFOR DELAY '0:0:<sec>'` | Delay trực tiếp |
| PostgreSQL | `pg_sleep()` | `pg_sleep(<sec>)` | Delay trực tiếp |
| PostgreSQL | `generate_series()` | `SELECT COUNT(*) FROM generate_series(1, <big_number>)` | Làm nặng truy vấn để tạo chậm |
| Oracle | `DBMS_PIPE.RECEIVE_MESSAGE()` | `DBMS_PIPE.RECEIVE_MESSAGE('<name>', <sec>)` | Delay trực tiếp theo số giây |
| SQLite | `RANDOMBLOB()` | `HEX(RANDOMBLOB(<big_number>))` | Tạo biểu thức nặng để làm chậm xử lý |