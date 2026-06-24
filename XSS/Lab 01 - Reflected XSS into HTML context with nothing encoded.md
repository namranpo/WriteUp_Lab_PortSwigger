<img width="1005" height="500" alt="image" src="https://github.com/user-attachments/assets/db9931e8-e685-494e-ad52-d0212cf1b083" />
# Tổng quan

Lab chứa lỗ hổng Reflected XSS ở chức năng tìm kiếm

Mục tiêu của ta là sử dụng tấn công XSS để gọi hàm alert()

# Khai thác
Ta biết được XSS là lỗ hổng cho phép người dùng viết mã Javascript bằng cách chèn thêm thẻ script:

```Javascript
<script>
//code JS
</script>
```
Quay lại với bài lab,ta thử với thẻ `<br>`:
```Javascript
Hello<br>World!
```
