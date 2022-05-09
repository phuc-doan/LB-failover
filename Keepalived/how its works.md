  ## 1. Keepalive
  
  
  
  
  ![image](https://user-images.githubusercontent.com/83824403/167387458-f2a292e7-fbce-496d-a0eb-40d951aa6329.png)






  
- Keepalived cung cấp các bộ thư viện (framework) cho 2 chức năng chính là: LB và failover với VRRP

- Tính năng LB sử dụng Linux Virtual Server (IPVS) module kernel trên Linux.

- Tính năng heath checking của các server backend linh động giúp duy trì được pool server dịch vụ nào còn sống để cân bằng tải tốt.

- Tính sẵn sàng cao (HA). Keepalived sử dụng VRRP (Virtual Redundancy Routing Protocol). VRRP được ứng dụng nhiều trong mô hình router failover (cisco vrrp)

## 2. Keepalived Failover IP hoạt động như thế nào ?

- Keepalived sẽ gom nhóm các máy chủ dịch vụ nào tham gia cụm HA, khởi tạo một Virtual Server đại diện cho một nhóm thiết bị đó với một Virtual IP (VIP) và một địa chỉ MAC vật lý của máy chủ dịch vụ đang giữ Virtual IP đó.


-  Vào mỗi thời điểm nhất định, chỉ có một server dịch vụ dùng địa chỉ MAC này tương ứng Virtual IP. Khi có ARP request gởi tới virtual IP thì server dịch vụ đó sẽ trả về địa chỉ MAC này.

- Các máy chủ dịch vụ sử dụng chung VIP phải liên lạc với nhau bằng địa chỉ multicast 224.0.0.18 bằng giao thức VRRP. 

- Các máy chủ sẽ có độ (priority) trong khoảng từ 1 – 254, và máy chủ nào có độ ưu tiên cao nhất sẽ thành Master, các máy chủ còn lại sẽ thành các Slave/Backup, hoạt động ở chế độ chờ.


=> Nếu vì một sự cố gì đó mà các server BACKUP không nhận được các gói tin quảng bá từ MASTER trong một khoảng thời gian nhất định thì cả nhóm sẽ bầu ra một MASTER mới.

=> MASTER mới này sẽ tiếp quản địa chỉ VIP của nhóm và gởi các gói tin ARP báo là nó đang giữ địa chỉ VIP này. 

=> Khi MASTER cũ hoạt động bình thường trở lại thì server này có thể lại trở thành MASTER hoặc trở thành BACKUP tùy theo cấu hình độ ưu tiên của các router.

## Khi keepalive chạy trên linux

**Hai tiến trình:**

- Một tiến trình cha có tên gọi là ‘watchdog‘, sản sinh ra 2 tiến trình con kế tiếp. Tiến trình cha sẽ quản lý theo dõi hoạt động của tiến trình con.


- Hai tiến trình con, một chịu trách nhiệm cho VRRP framework và một chịu trách nhiệm cho health checking (kiểm tra tình trạng sức khoẻ).

*Watchdog sẽ giao tiếp với các tiến trình con qua unix domain socket trên Linux để quản lý các tiến trình con.*
