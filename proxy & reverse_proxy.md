## 1. Proxy 
## 2. Reverse Proxy 


# M
![image](https://user-images.githubusercontent.com/83824403/166862953-175eeb42-2839-435d-9100-c867caa239d7.png)


### 1. Proxy (Forward proxy) là gì 
- Đặt ở phía client
- Proxy là một server có nghiệm vụ chuyển tiếp và kiểm soát thông tin giữa client và server phía backend. 
- Proxy gồm 1 địa chỉ IP và một port để truy cập cố định


![image](https://user-images.githubusercontent.com/83824403/166863147-a1510384-1677-41d7-a7a3-853f502b5add.png)

#### Khả năng:

- Ẩn IP client khi truy cập webserver
- Block web ko mong muốn
- Caching

### 2. Reverse Proxy là gì
- Đặt ở khía server
- Nếu forward proxy nói đến việc forward request (requests go out from client to server) thì reverse proxy nói đến việc receive request (requests come in from client to server)

![image](https://user-images.githubusercontent.com/83824403/166863427-2f16ae79-f719-4bc8-a756-e82010849811.png)


#### Khả năng:
- LB
- Logging: quản lý log của access tới từng server và endpoint sẽ dễ dàng hơn rất nhiều so với việc kiểm tra trên từng server một.
- Ẩn IP của backend
- Encrypt connection: Mã hóa kết nối client và reverse proxy ( TLS ). User sẽ được hưởng lợi từ việc bảo mật (https)
