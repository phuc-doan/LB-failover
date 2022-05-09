# Cấu hình Keepalived + Haproxy cho 2 web server nginx

## Mô hình:
 
 
 ![image](https://user-images.githubusercontent.com/83824403/167338530-c626e988-1338-4595-b729-5ac15c52d8f5.png)


## List

- 192.168.187.147: HA1 + Keepalived

- 192.168.187.148: HA2 + Keepalived

- 192.168.187.149: APP1 (nginx)

- 192.168.187.150: APP2 (nginx)

- 192.168.187.200: VIP


## Let's get started!

## Bước 1: Cấu hình nginx trên APP1 và APP2

- APP1+ APP2:

```
sudo yum install epel-release -y
sudo yum install nginx
sudo systemctl start nginx
sudo systemctl status nginx
systemctl enable nginx
```

- Sửa file index.html của nginx

```
vi /usr/share/nginx/html/index.html
- web nginx1 : với node 1
- web nginx2 : với node 2
Mục đích là tí phân biệt HAproxy LB cho dễ dàng
```



- Mở port cho http hoạt động ( Nếu ngại trên mô hình lab này có thể tắt sạch firewalld đi )

```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

- chỉnh file host trên cả 4 node ( HA1, HA2, APP1, APP2)

``` 
vi /etc/hosts


192.168.187.147: HA1 
192.168.187.148: HA2 
192.168.187.149: APP1 
192.168.187.150: APP2 
```

- Cấu hình hostname bằng hostnamectl cho từng host tương ứng với từng server. 
- VD trên node HA1:

``` 
hostnamectl set-hostname HA1
```

### - Test việc cài cắm nginx:

```
curl APP1  # trên node APP1
curl APP2   # Trên node  APP2
```

- Như vậy là tạm OK

![image](https://user-images.githubusercontent.com/83824403/167339702-74f2abc7-0d7c-4d24-8509-a5d8f0b0a678.png)

## Step 2: Cài Haproxy trên HA1 và HA2

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



## Step 3: Cấu hình keepalive

- Trên 2 node HA1, HA2 chúng ta cài thêm Keepalived service


```
yum -y install keepalived
systemctl start keepalived
systemctl enable keepalived
systemctl status keepalived
```

- Cấu hình keepalive trên cả 2 node HA1, HA2


```

vrrp_script chk_haproxy {
  script "killall -0 haproxy" # check the haproxy process
  interval 2 # every 2 seconds
  weight 2 # add 2 points if OK
}

vrrp_instance VI_1 {
  interface ens33 # interface to monitor
  state MASTER # MASTER on haproxy1, BACKUP on haproxy2
  virtual_router_id 51
  priority 101 # 101 on haproxy1, 100 on haproxy2
  virtual_ipaddress {
    192.168.187.200 # virtual ip address
  }
  track_script {
    chk_haproxy
  }
}

```


- Restart lại service cho ăn cấu hình ( trên cả 2 node )

```
systemctl restart keepalived
```


### - Test

=> Case 1: request liên tục vào VIP của Keepalive 

```
curl 192.168.187.200
curl 192.168.187.200
...
```
- Kết quả như sau:

![image](https://user-images.githubusercontent.com/83824403/167340989-b20b5072-0f22-4908-bde3-2b643ad810c1.png)


### vì conf của haproxy đang đặt là roundrobin nên nó cứ call tuần tự web 1 rồi web 2 và tiếp tục

=> Case 2:  Down 1 con Haproxy đi ( ở đây mình down node HA1)

- Vẫn request liên tục vào **192.168.187.200** có thể dùng **chrome** hay **cli** cũng được



- Status thông thường trên node HA2 của keepalived

![image](https://user-images.githubusercontent.com/83824403/167341220-906232e5-a8b0-4c8f-a0f7-b2b39f74aa42.png)


- Sau khi down node HA1 thì đây là status của keepalived:

![image](https://user-images.githubusercontent.com/83824403/167341267-d49e4cec-21da-46fd-9988-bf565ba96574.png)



- Request liên tục vào **192.168.187.200** có thể dùng **chrome** hay **cli** 


![image](https://user-images.githubusercontent.com/83824403/167341448-fb1cf93f-a592-40e7-9a2f-ee0ac805107f.png)

 => Như vậy mô hình **`Keepalive + Haproxy`** đang *`failover`* dựa trên Keepalive và *`LB`* trên Haproxy rất tốt
 
 ![image](https://user-images.githubusercontent.com/83824403/167341651-dcb926f0-c4ea-47c5-b67f-744ea8b1ff2d.png)
 
 - 
 
 ![image](https://user-images.githubusercontent.com/83824403/167341676-75688069-7b3f-4d86-8539-4f5ee1bea23f.png)



## The end!
