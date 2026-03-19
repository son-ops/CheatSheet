# Truncation

| DBMS | Truncation | Ghi chú ngắn |
|---|---|---|
| MySQL | Có điều kiện | Nếu **không bật strict mode**, giá trị vượt độ dài cột có thể bị **cắt bớt** và sinh warning; bật strict mode thì thường thành lỗi.  |
| MariaDB | Có điều kiện | Tương tự MySQL; khi **không strict** có thể tự điều chỉnh giá trị, gồm cả **truncating strings that are too long**.  |
| PostgreSQL | Thường không | Với `varchar(n)`, chuỗi dài hơn giới hạn thường **bị lỗi**; chỉ trường hợp ký tự dư toàn là khoảng trắng thì có thể bị cắt.  |
| SQLite | Không theo kiểu truyền thống | SQLite không ép chặt độ dài `VARCHAR(n)`; có thể lưu cả chuỗi rất dài mà **không lỗi, không truncate**. |

---

> Với MySQL cần tắt strict mode bằng câu lệnh:

`SET SQL_MODE="NO_AUTO_VALUE_ON_ZERO,ONLY_FULL_GROUP_BY,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION"`

---

# Loose Comparison

| DBMS | Loose comparison | Ví dụ tiêu biểu | Ghi chú ngắn |
|---|---|---|---|
| MySQL | Có | `'0' = 0` → `TRUE` ; `'0.0' = 0` → `TRUE` ; `'x6' = 0` → `TRUE` ; `7 > '6x'` → `TRUE` | MySQL tự động chuyển chuỗi sang số khi cần; nhiều chuỗi khác nhau có thể cùng bị hiểu thành một giá trị số. |
| MariaDB | Có | `'0' = 0` → `TRUE` ; `'0.0' = 0` → `TRUE` ; `'0      ' = '0'` → `TRUE` ; `0 = 'cookiehanhoan'` → `TRUE` kèm warning | Rất gần MySQL; nếu một vế là string và vế kia là integer thì MariaDB sẽ so sánh theo kiểu số thập phân. |
| PostgreSQL | Không nổi bật | `'0' = 0` → thường lỗi ; `'1'::int = 1` → `TRUE` ; `CAST('0' AS int) = 0` → `TRUE` | PostgreSQL không “loose” kiểu MySQL/MariaDB; thường phải có operator/cast phù hợp mới so sánh được. |
| SQLite | Có | `'0' = 0` ; `'500' = 500` ; so sánh còn phụ thuộc affinity của cột / biểu thức | SQLite dùng dynamic typing và type affinity, nên việc chuyển kiểu trước khi so sánh linh hoạt hơn PostgreSQL. |