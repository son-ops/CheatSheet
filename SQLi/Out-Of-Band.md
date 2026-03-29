## Out-of-Band

>**Cơ chế tổng quát** của OOB SQLi: thay vì dữ liệu xuất hiện trong HTTP response, DB/server sẽ bị ép tạo kết nối ra ngoài qua SMB / DNS / HTTP.

| DBMS | Hàm / kỹ thuật | Syntax khung | Giải thích |
|---|---|---|---|
| MySQL | `INTO OUTFILE`| `SELECT <data> INTO OUTFILE '<network_path>'` | Ghi dữ liệu ra một đường dẫn mạng. Nếu DB server cố truy cập hoặc ghi tới tài nguyên mạng, có thể tạo callback SMB/UNC. |
| MSSQL | File-reading functions | `<function_name>('<network_path_with_data>', ...)` | Một số hàm đọc trace/audit/file có thể bị lạm dụng để buộc SQL Server truy cập UNC path chứa dữ liệu cần exfil. |
| Oracle | `XML` / `XXE-based primitive` | `SELECT <xml_function>(<xml_payload_with_external_entity>) FROM dual` | Lợi dụng XML parser / external entity để ép Oracle gửi request HTTP hoặc DNS tới server ngoài, trong đó hostname/URL có chứa dữ liệu. |

