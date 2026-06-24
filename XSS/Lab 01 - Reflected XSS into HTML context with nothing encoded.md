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
Quay lại với bài lab,ta thử tìm kiếm với chuỗi `Hello World!` với thẻ `<br>` có gì khác nhau

Với chuỗi `Hello World!`:

<img width="1231" height="510" alt="image" src="https://github.com/user-attachments/assets/0b25cb56-5761-44a3-a2e3-d27716265761" />

Với thẻ `<br>` ở giữa chuỗi:

<img width="1224" height="619" alt="image" src="https://github.com/user-attachments/assets/c7057d45-eb16-451c-b21d-6c5939502cd6" />

Ta có thể thấy rằng web đang render thẻ `<br>`.Vậy nếu ta thử với thẻ `<script>` thì sao?

```Javascript
<script>alert(1)</script>
```
<img width="1193" height="654" alt="image" src="https://github.com/user-attachments/assets/768ef48b-3150-4452-aba2-517617d5640f" />

Bấm tìm kiếm:

<img width="1696" height="968" alt="image" src="https://github.com/user-attachments/assets/c5b005af-0132-432b-b3e2-f5dfa573292c" />

=> Ta đã thành công gọi hàm `alert(1)`

<img width="1673" height="891" alt="image" src="https://github.com/user-attachments/assets/826b0e08-f26a-4c20-9e05-0f1c2527b834" />

=> SOLVED!
