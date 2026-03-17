# Blind SQLi Cheatsheet

# Contents
* [Character extraction](#character-extraction)
* [Conditional / branching](#conditional--branching)
* [Time delay](#time-delay)

---

## Character extraction

| DBMS | Primitive | Syntax khung | Giải thích |
|---|---|---|---|
| MySQL | `SUBSTR()` | `SUBSTR(<expr>, <pos>, 1)` | Cắt chuỗi con từ vị trí chỉ định |
| MySQL | `SUBSTRING()` | `SUBSTRING(<expr>, <pos>, 1)` | Tương đương `SUBSTR()` |
| MySQL | `MID()` | `MID(<expr>, <pos>, 1)` | Alias của `SUBSTRING()` trong MySQL |
| MySQL | `LEFT()` | `LEFT(<expr>, <len>)` | Lấy N ký tự từ bên trái |
| MySQL | `RIGHT()` | `RIGHT(<expr>, <len>)` | Lấy N ký tự từ bên phải |
| MSSQL | `SUBSTRING()` | `SUBSTRING(<expr>, <pos>, 1)` | Cắt chuỗi con theo vị trí |
| PostgreSQL | `SUBSTR()` | `SUBSTR(<expr>, <pos>, <len>)` | Cắt chuỗi con |
| PostgreSQL | `SUBSTRING()` | `SUBSTRING(<expr>, <pos>, <len>)` | Tương đương `SUBSTR()` |
| PostgreSQL | `SUBSTRING ... FROM ... FOR` | `SUBSTRING(<expr> FROM <pos> FOR <len>)` | Cú pháp chuẩn của PostgreSQL |
| SQLite | `SUBSTR()` | `SUBSTR(<expr>, <pos>, <len>)` | Cắt chuỗi con |
| SQLite | `SUBSTRING()` | `SUBSTRING(<expr>, <pos>, <len>)` | Cách viết khác của `SUBSTR()` |
| Oracle | `SUBSTR()` | `SUBSTR(<expr>, <pos>, <len>)` | Cắt chuỗi con |

---

## Conditional / branching

| DBMS | Primitive | Syntax khung | Giải thích |
|---|---|---|---|
| MySQL | `IF()` | `IF(<condition>, <true_expr>, <false_expr>)` | Rẽ nhánh nếu điều kiện đúng hoặc sai |
| MySQL | `CASE WHEN` | `CASE WHEN <condition> THEN <a> ELSE <b> END` | Rẽ nhánh kiểu chuẩn SQL |
| MySQL | `MAKE_SET()` | `MAKE_SET(<cond>, <value>)` | Trả về giá trị khi điều kiện phù hợp |
| MySQL | `ELT()` | `ELT(<index>, <a>, <b>, ...)` | Chọn giá trị theo vị trí chỉ số |
| MSSQL | `IF ... ELSE` | `IF <condition> <true_stmt> ELSE <false_stmt>` | Rẽ nhánh theo điều kiện |
| MSSQL | `CASE WHEN` | `CASE WHEN <condition> THEN <a> ELSE <b> END` | Rẽ nhánh dạng biểu thức |
| PostgreSQL | `CASE WHEN` | `CASE WHEN <condition> THEN <a> ELSE <b> END` | Rẽ nhánh theo điều kiện |
| Oracle | `CASE WHEN` | `CASE WHEN <condition> THEN <a> ELSE <b> END` | Rẽ nhánh theo điều kiện |

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