# Contents
* [Version function](#version-enumeration-cheatsheet)
* [Comment](#sql-comment-cheatsheet)
* [Nối chuỗi](#string-concatenation-cheatsheet)
* [Gộp row](#string-aggregation-cheatsheet)

## Version Enumeration Cheatsheet

| DBMS | Query |
|---|---|
| MySQL | `SELECT VERSION();` / `SELECT @@version;` |
| PostgreSQL | `SELECT version();` |
| MSSQL | `SELECT @@version;` |
| Oracle | `SELECT banner FROM v$version;` / `SELECT version FROM product_component_version;` |
| SQLite | `SELECT sqlite_version();` |

---

## SQL Comment Cheatsheet

| DBMS | Single-line comment | Multi-line comment |
|---|---|---|
| MySQL | `-- ` / `#`/`;%00` | `/* comment */` |
| PostgreSQL | `-- ` | `/* comment */` |
| MSSQL | `-- `/ `;%00` | `/* comment */` |
| Oracle | `-- ` | `/* comment */` |
| SQLite | `-- ` | `/* comment */` |

## String Concatenation Cheatsheet

> Nối chuỗi trong một row.

| DBMS | Toán tử | Hàm |
|---|---|---|
| MySQL | `\|\|` | `CONCAT(...)`, `CONCAT_WS(separator, ...)` |
| PostgreSQL | `\|\|` | `concat(...)`, `concat_ws(separator, ...)` |
| MSSQL | `+`, `\|\|` | `CONCAT(...)`, `CONCAT_WS(separator, ...)` |
| Oracle | `\|\|` | `CONCAT(...)` |
| SQLite | `\|\|` | `concat(...)`, `concat_ws(separator, ...)` |

### Ghi chú

| DBMS | Note |
|---|---|
| MySQL | `\|\|` chỉ hoạt động như nối chuỗi khi bật `PIPES_AS_CONCAT` |
| MSSQL | `\|\|` chỉ có ở bản mới, `+` vẫn phổ biến hơn |
| SQLite | `concat(...)`, `concat_ws(...)` chỉ có ở bản mới hơn; cách ổn định nhất là `\|\|` |

### Ví dụ
` SELECT CONCAT('admin', ':', '123'); `/`SELECT 'admin' + ':' + '123';`/`SELECT 'admin' || ':' || '123';`

---

## String Aggregation Cheatsheet

> Gộp **nhiều row thành một chuỗi**.

| DBMS | Hàm |
|---|---|
| MySQL | `GROUP_CONCAT(expr)` / `GROUP_CONCAT(expr SEPARATOR '<sep>')` |
| PostgreSQL | `string_agg(expr, '<sep>')` |
| MSSQL | `STRING_AGG(expr, '<sep>')` |
| Oracle | `LISTAGG(expr, '<sep>') WITHIN GROUP (ORDER BY <col>)` |
| SQLite | `group_concat(expr)` / `group_concat(expr, '<sep>')` / `string_agg(expr, '<sep>')` |

### Ví dụ: 
`SELECT string_agg(username, ':') FROM users;` / `SELECT LISTAGG(username, ':') WITHIN GROUP (ORDER BY 1) FROM users;`

---

## Convert Characters to Integers
| DBMS | Function | Output |
| ---- | -------- | ------ |
|MySQL| `ASCII('A')`|65|
|PostgreSQL|`ASCII('A')`|65|
|MSSQL|`UNICODE('A')`|65|
|Oracle|`ASCII('A')`|65|
|SQLite|`UNICODE('A')`|65|