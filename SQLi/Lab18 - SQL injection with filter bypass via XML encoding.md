![Tổng quan về Lab](images/Pasted%20image%2020260624103911.png)
# Tổng quan
Lab có lỗ hổng SQLi ở tính năng stock check

Kết quả được trả về

Ta có thể sử dụng UNION attack để lấy ra thông tin `username` và `password`

Mục tiêu của ta là đăng nhập bằng tài khoản admin
# Khai thác

Đầu tiên,ta tương tác với web bình thường bấm vào `View Details`:
![Trang sản phẩm và nút View Details](images/Pasted%20image%2020260624113727.png)
Bấm vào `check stock` một nơi có vẻ sẽ xuất hiện lỗ hổng SQLi và quay lại Burp ta check phần `Request` của `POST /product/stock` :
![Request POST product stock trong Burp Suite](images/Pasted%20image%2020260624113931.png)
Ta tiến hành `Send To Repeater`,để check xem SQLi nằm ở đâu ta thử payload `'--` với thẻ storeId với productID:
![Kiểm tra SQLi trên tham số XML](images/Pasted%20image%2020260624114525.png)

Với thẻ `<storeId>` ta có thể thấy web chặn không cho ta query không hợp lệ vào server

Nhưng nếu ta sử dụng dec_entities thì sao?

Ta sử dụng Extension `Hackvector` chọn thẻ xml dec_entities ta thử cho query `OR 1=1` vào:
![Bypass bằng XML dec_entities và OR 1=1](images/Pasted%20image%2020260624115108.png)
Và nó hiện ra hết các số lượng unit => SQLi thành công

Việc còn lại của ta là ta sẽ dùng `UNION attack` để lấy ra tài khoản,mật khẩu admin,ta tiến hành xem có bao nhiêu cột:
```SQL
ORDER BY 1 --346 unit
ORDER BY 2 --0 unit
```
=> câu truy vấn trước có 1 cột

Tiếp theo ta tiến hành `UNION attack` với 1 cột:
```SQL
UNION SELECT username||'~'||password FROM users
```
![UNION attack lấy username và password](images/Pasted%20image%2020260624115646.png)
Và ta đã thành công leak ra được username và password của tài khoản admin:
`administrator~f9nkcqv6ek4m99cg75av`

Việc còn lại của ta là đăng nhập và hoàn thành lab:
![Đăng nhập administrator thành công](images/Pasted%20image%2020260624115809.png)

=> solved!
