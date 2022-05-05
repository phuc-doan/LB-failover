## References:

- https://blog.cloud365.vn/linux/cau-truc-file-cau-hinh-haproxy/#:~:text=C%E1%BA%A5u%20h%C3%ACnh%20c%E1%BB%A7a%20HAProxy%20th%C6%B0%E1%BB%9Dng,t%E1%BB%9Bi%20c%C3%A1c%20Backend%20ph%C3%ADa%20sau.



- https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration/


## Thông số cấu hình

```
global
    # Các thiết lập tổng quan

defaults
    # Các thiết lập mặc định

frontend
    # Thiết lập điều phối các request

backend
    # Định nghĩa các server xử lý request
```


- Các thiết lập tại `global` sẽ được sử dụng để định nghĩa cách HAProxy được khởi tạo như số lượng kết nối tối đa, đường dẫn ghi file log, số process v.v.


- Sau đó các thiết lập tại mục `defaults` sẽ được áp dụng cho tất cả mục `frontend, backend` nằm phía sau (các bạn hoàn toàn có thể định nghĩa lại các giá tri mặc định tại frontend và backend).

-  Có thể có nhiều mục `frontend, backend` được định nghĩa trong file cấu hình. 


-  Mục `frontend` được định nghĩa để điều hướng các request nhận được tới các `backend`. 


-  Mục `backend` sử dụng để định nghĩa các danh sách máy chủ dịch vụ (có Web server, Database, …) đây là nơi request được xử lý.

## Cấu hình logging Haproxy

```
 yum -y install rsyslog

 vi /etc/rsyslog.d/haproxy.conf

```
- Dán nội dung sau vào file **/etc/rsyslog.d/haproxy.conf**

```
if ($programname == 'haproxy') then -/var/log/haproxy.log

```
- Sửa file /etc/rsyslog.conf
```

# Collect log with UDP
$ModLoad imudp
$UDPServerAddress 192.168.187.139
$UDPServerRun 514
```


![image](https://user-images.githubusercontent.com/83824403/166904755-109b2416-b714-47ae-b558-32d73c139557.png)

 Chỉnh quyền cho file log
 
```
 chmod 755 /var/log/haproxy.log
 systemctl restart rsyslog
systemctl status ryslog

```

- chỉnh file config haproxy


```
global
    log         192.168.187.139:514 local0 info

```


![image](https://user-images.githubusercontent.com/83824403/166906094-b9ac0188-be49-4cf1-9bfa-2e63e7bbb3d3.png)



=> Đã get được log vào file haproxy.log


![image](https://user-images.githubusercontent.com/83824403/166905388-4426cd4b-2ba3-4ddf-8345-db0c44250f98.png)



