## Symlink

Đây là một file đặc biệt, nó không chứa nội dung mà nó trỏ tới một file hoặc thư mục khác trên hệ thống. Các thao tác với symlink thực chất là thao tác với file hay thư mục mà nó trỏ đến. 

Chính vì thế ta có thể tận dụng symlink để lừa server thao tác lên file hoặc folder khác với `quyền của process đang chạy`.
- Với symlink trỏ tới file, các thao tác như đọc, ghi hoặc thực thi sẽ tác động lên file đích (xóa hoặc di chuyển chính symlink chỉ ảnh hưởng tới bản thân symlink)
- Với symlink trỏ tới thư mục, nếu ứng dụng ghi file mới thông qua đường dẫn đó (symlink) thì dữ liệu thực chất sẽ được ghi vào thư mục đích

Tuy nhiên symlink không phải lúc nào cũng được giữ nguyên khi truyền qua mạng. Nhiều cơ chế upload/truyền file thông thường chỉ gửi nội dung file chứ không giữ metadata về loại entry trong filesystem. Vì vậy, nếu muốn phía nhận khôi phục được đúng một symlink, thường phải dùng định dạng hoặc công cụ có hỗ trợ lưu thông tin này, chẳng hạn như tar hoặc zip có hỗ trợ symlink.

- Câu lệnh tạo symlink: `ln -s index.php symindex.txt`
- Nén symlink với zip: `zip -y test.zip symindex.txt`
- Nén symlink với tar: `tar -cf test.tar symindex.txt`

### Cách phòng chống

- Không cho phép follow symlink nếu không cần thiết
- Nếu vẫn cần thì có thể resolve symlink và check xem file nó trỏ tới có nằm trong thư mục được phép không