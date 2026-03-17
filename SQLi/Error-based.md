# Table of content
* [MySQL](#MySQL)
* [PostgreSQL](#PostgreSQL)
* [MSSQL](#MSSQL)
* [Oracle](#Oracle)
* [Sqlite](#Sqlite)

# MySQL
Ta nên sử dụng payload `CONCAT('~', data, '~')` ở vị trí `subquery` để thêm marker cho dễ đọc.
### 1. EXTRACTVALUE(xml, xpath)
- **xml**: dữ liệu XML đầu vào, trong payload thường không quan trọng
- **xpath**: biểu thức XPath, là chỗ bị lợi dụng để gây lỗi
- **Cơ chế khai thác:** làm `xpath` sai cú pháp để error message hiện ra dữ liệu
```sql
EXTRACTVALUE(1, (<subquery>))
```
### 2. UPDATEXML(xml_target, xpath_expr, new_xml)
- **xml_target**: dữ liệu XML đầu vào, thường không quan trọng trong payload
- **xpath_expr**: biểu thức XPath, là chỗ bị lợi dụng để gây lỗi
- **new_xml**: dữ liệu XML mới để thay thế, trong payload thường chỉ truyền đại
- **Cơ chế khai thác:** làm `xpath_expr` sai cú pháp để MySQL trả lỗi chứa dữ liệu
```sql
UPDATEXML(1, (<subquery>), 1)
```
### 3. GTID_SUBSET(gtid_set1, gtid_set2)
- **gtid_set1**: tập GTID thứ nhất
- **gtid_set2**: tập GTID thứ hai
- **Cơ chế khai thác:** truyền GTID format sai để hàm parse lỗi và hiện dữ liệu trong thông báo lỗi
```sql
GTID_SUBSET((<subquery>), 1)
```
### 4. JSON_KEYS(json_doc)
- **json_doc**: tài liệu JSON đầu vào
- **Cơ chế khai thác:** đưa dữ liệu không phải JSON hợp lệ vào để gây lỗi parse JSON và làm lộ dữ liệu
```sql
JSON_KEYS((<subquery>))
```
### 5. NAME_CONST(name, value)
- **name**: tên hằng / tên cột được tạo ra
- **value**: giá trị của hằng
- **Cơ chế khai thác:** tạo 2 cột trùng tên, trong đó `name` chứa dữ liệu cần leak, khiến MySQL báo lỗi duplicate column
```sql
SELECT NAME_CONST((<subquery>),1), NAME_CONST((<subquery>),1)
```
### 6. FLOOR(RAND()) + GROUP BY
- **RAND(0)**: sinh giá trị giả ngẫu nhiên nhưng có tính lặp
- **FLOOR(...*2)**: ép về 0 hoặc 1 để dễ tạo va chạm
- **GROUP BY**: nhóm theo giá trị có chứa dữ liệu cần leak
- **Cơ chế khai thác:** tạo lỗi duplicate entry, dữ liệu xuất hiện trong giá trị bị trùng
```sql
SELECT COUNT(*), CONCAT('~', (<subquery>), '~', FLOOR(RAND(0)*2)) AS x
FROM (SELECT 1 UNION SELECT 2) a
GROUP BY x
```
### 7. EXP(expr)
- **expr**: biểu thức số học đầu vào
- **Cơ chế khai thác:** ép dữ liệu chuỗi vào ngữ cảnh số học để gây lỗi ép kiểu hoặc overflow
```sql
EXP(~(<subquery>))
```
### 8. UUID_TO_BIN(string_uuid)
- **string_uuid**: chuỗi UUID đầu vào
- **Cơ chế khai thác:** truyền dữ liệu không đúng format UUID để gây lỗi parse và làm lộ dữ liệu
```sql
UUID_TO_BIN((<subquery>))
```
...

# PostgreSQL
Ta nên sử dụng marker như `'~'||data||'~'` ở vị trí `subquery` để thêm marker cho dễ đọc.

### 1. CAST(expr AS type) / expr::type
- **expr**: giá trị cần ép kiểu
- **type**: kiểu dữ liệu đích như `INT`, `NUMERIC`
- **Cơ chế khai thác:** ép một chuỗi không thể chuyển sang số sang kiểu số để PostgreSQL báo lỗi và hiện dữ liệu trong thông báo lỗi
```sql
CAST((<subquery>) AS NUMERIC) hoặc (<subquery>)::NUMERIC
```

# MSSQL

Ta nên sử dụng marker như `'~'+data+'~'` ở vị trí `subquery` để thêm marker cho dễ đọc.

### 1. CONVERT(data_type, expression)
- **data_type**: kiểu dữ liệu đích như `INT`
- **expression**: giá trị cần chuyển kiểu
- **Cơ chế khai thác:** ép một chuỗi không thể chuyển sang số sang kiểu số để SQL Server báo lỗi và hiện dữ liệu trong thông báo lỗi
```sql
CONVERT(INT, (<subquery>))
```
### 2. CAST(expression AS data_type)
- **expression**: giá trị cần ép kiểu
- **data_type**: kiểu dữ liệu đích như `INT`
- **Cơ chế khai thác:** tương tự `CONVERT`, ép chuỗi không thể chuyển sang số sang kiểu số để sinh lỗi conversion

```sql
CAST((<subquery>) AS INT)
```
### 3. IN (subquery / value list)
- **vế trái**: giá trị đem đi so sánh, ví dụ `1337`
- **vế phải**: tập giá trị hoặc subquery
- **Cơ chế khai thác:** bản thân `IN` không gây lỗi; lỗi xuất hiện khi SQL Server phải so sánh số với chuỗi không thể convert, từ đó phát sinh lỗi chuyển kiểu và làm lộ dữ liệu

```sql
1337 IN (SELECT (<subquery>))
```
### 4. EQUAL / so sánh bằng (`=`)
- **vế trái**: giá trị dùng để so sánh, ví dụ số `1337`
- **vế phải**: biểu thức chứa dữ liệu cần leak
- **Cơ chế khai thác:** bản thân toán tử `=` không gây lỗi; lỗi xuất hiện khi SQL Server cố chuyển chuỗi ở một vế sang kiểu số của vế còn lại để thực hiện so sánh

```sql
1337 = (<subquery>)
```

# Oracle

Ta nên sử dụng marker như `'||'~'||data||'~'||'` hoặc ghép bằng `CHR(126)` để thêm marker cho dễ đọc.
### 1. UTL_INADDR.GET_HOST_NAME(ip_address)
- **ip_address**: địa chỉ IP hoặc chuỗi đầu vào để resolve hostname
- **Cơ chế khai thác:** truyền vào dữ liệu không hợp lệ để Oracle báo lỗi network / host lookup và làm lộ dữ liệu trong thông báo lỗi

```sql
UTL_INADDR.GET_HOST_NAME((<subquery>))
```
### 2. CTXSYS.DRITHSX.SN(owner, text)
- **owner**: user/schema
- **text**: chuỗi đầu vào
- **Cơ chế khai thác:** truyền dữ liệu đặc biệt vào hàm của Oracle Text để sinh lỗi và làm lộ dữ liệu trong thông báo lỗi

```sql
CTXSYS.DRITHSX.SN(USER, (<subquery>))
```
### 3. ORDSYS.ORD_DICOM.GETMAPPINGXPATH(input, arg1, arg2)
- **input**: dữ liệu đầu vào
- **arg1 / arg2**: tham số phụ
- **Cơ chế khai thác:** truyền dữ liệu không hợp lệ vào ngữ cảnh XPath để Oracle báo lỗi XPath và làm lộ dữ liệu

```sql
ORDSYS.ORD_DICOM.GETMAPPINGXPATH((<subquery>), USER, USER)
```
### 4. CAST(expr AS type)
- **expr**: giá trị cần ép kiểu
- **type**: kiểu dữ liệu đích như `NUMBER`
- **Cơ chế khai thác:** ép một chuỗi không thể chuyển sang số sang kiểu số để Oracle báo lỗi conversion

```sql
CAST((<subquery>) AS NUMBER)
```

### 5. XDBURITYPE(value).getBlob()
- **value**: URI / chuỗi đầu vào
- **getBlob()**: lấy nội dung dạng BLOB
- **Cơ chế khai thác:** truyền dữ liệu không hợp lệ vào ngữ cảnh URI/XML DB để sinh lỗi và làm lộ dữ liệu

```sql
XDBURITYPE((<subquery>)).getBlob()
```
### 6. XDBURITYPE(value).getClob()
- **value**: URI / chuỗi đầu vào
- **getClob()**: lấy nội dung dạng CLOB
- **Cơ chế khai thác:** tương tự `getBlob()`, lợi dụng lỗi xử lý URI/XML DB để làm lộ dữ liệu

```sql
XDBURITYPE((<subquery>)).getClob()
```
### 7. XMLType(expr)
- **expr**: chuỗi/XML đầu vào
- **Chức năng gốc:** tạo một đối tượng XMLType từ chuỗi XML
- **Cơ chế khai thác:** tạo XML không hợp lệ để Oracle báo lỗi parse XML và làm lộ dữ liệu trong thông báo lỗi

```sql
XMLType((<subquery>))
```
### 8. DBMS_UTILITY.SQLID_TO_SQLHASH(value)
- **value**: SQL ID đầu vào
- **Cơ chế khai thác:** truyền dữ liệu không đúng format SQL ID để Oracle báo lỗi parse/validation và làm lộ dữ liệu

```sql
DBMS_UTILITY.SQLID_TO_SQLHASH((<subquery>))
```
# SQLite

### 1. load_extension(path)
- **Chức năng gốc:** nạp shared library extension vào SQLite
- **Cơ chế khai thác:** gọi với đối số không hợp lệ hoặc khi extension loading bị cấm để tạo lỗi
- **Vai trò trong SQLi:** thường dùng trong `CASE WHEN` để phân biệt đúng/sai qua việc có lỗi hay không

```sql
CASE WHEN (<boolean_query>) THEN 1 ELSE load_extension(1) END
```

