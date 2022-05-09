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



- Chỉnh quyền cho file log
 
 
 
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


## Cấu hình haproxy 2 webserver

- Cài đặt Haproxy Trên cả 2 node HA1 và HA2

```
yum -y install haproxy
systemctl restart haproxy
systemctl enable haproxy
systemctl status haproxy
```



- Cấu hình Haproxy trên cả 2 node HA1, HA2



```
global
        log 127.0.0.1 local0 notice
        user haproxy
        group haproxy
        maxconn 256
        daemon
        stats socket /var/lib/haproxy/stats

    defaults
        log global
        retries 2
        timeout connect 5000ms
        timeout client  50000ms
        timeout server  50000ms

listen stats
    bind :8080
    mode http
    stats enable
    stats uri /stats
    stats realm HAProxy\ Statistics

    listen HA1
        bind :80
        mode http

        balance roundrobin
            server APP1 192.168.187.149:80 check
            server APP2 192.168.187.150:80 check
```


- Restart lại cho ăn cấu hình 


```
systemctl restart haproxy
```

### - Test: request liên tục vào HA1 thì nó ra như này, và nếu là HA2 thì nó cũng tương tự là tạm thành công



![image](https://user-images.githubusercontent.com/83824403/167340427-249fa642-d70f-4c41-85a0-ac9e148f90b1.png)


