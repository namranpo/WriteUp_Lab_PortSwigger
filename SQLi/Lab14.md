![Giao diện lab ban đầu](images/Pasted%20image%2020260613011939.png)
# Tổng quan
-Lab có blind SQLi vuln.Lab sử dụng tracking cookie để truy vấn SQL

-Mục tiêu của ta là khiến cho trang web bị delay 10 giây

# Khai thác
-Theo như description của lab,thì ta biết được lỗ hổng nằm ở `TrackingId` và mục tiêu của ta là làm cho web bị delay 10 giây.Ban đầu ta thấy ở phần Response thì time là 232 milis 

![Response ban đầu khoảng 232ms](images/Pasted%20image%2020260613012427.png)

Vậy nếu ta sử dụng Payload:
```SQL
;SELECT pg_sleep(10)--;
```


![Payload SELECT pg_sleep(10) không thành công](images/Pasted%20image%2020260613013055.png)
Nhưng kết quả lại không có gì thay đổi đáng kể?? có vẻ như web đang chặn stack queries ta có thể dùng cách khác là nối chuỗi kết quả của SQL query của `TrackingId` với `pg_sleep(10)`:
```SQL
'||pg_sleep(10);
```

![Payload nối chuỗi với pg_sleep(10) thành công](images/Pasted%20image%2020260613013800.png)
Vậy là ta thành công khiến web bị delay 10s
![Solve bài lab](images/Pasted%20image%2020260613013836.png)
=>solved!
