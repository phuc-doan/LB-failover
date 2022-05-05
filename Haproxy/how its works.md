## Theo cách đơn giản nhất
-  về cơ bản nó lấy các tham số mà đã được chỉ định trong call request (giả sử, yêu cầu HTTP), ví dụ: "q", "row", v.v. và nối chúng với các server bạn đã định cấu hình HAProxy để hoạt động.


# Các thuật ngữ trong HAProxy

## Access Control List (ACL)

Trong cân bằng tải, ACL được dùng để kiểm tra điều kiện và thực hiện một hành động (ví dụ như lựa chọn một server hay chặn một request) dựa trên kết quả của việc kiểm tra đó.


Ví dụ của một ACL:


```
acl url_blog        src         /something
```
- ACL này sử dụng cho các request có chứa /something (vd như: localhost/something/1/) và các request này được redirect đến src

## Backend

Backend là một tập các server mà HAProxy có thể forward các request tới. Backend được cấu hình trong mục backend trong file configuration của HAProxy. Ở dạng đơn giản nhất, một backend có thể được cài đặt bằng cách

- Đặt ra thuật toán để căn bằng tải (round-robin, least-connection,...)
- Một danh sách các máy chủ và port của chúng có thể dùng để nhận request từ HAProxy

Một backend có thể chứa một hoặc nhiều server


- Một ví dụ cấu hình của backend trong file cconfiguration của HAProxy:
```
backend web-backend
    balance leastconn
    mode http
    server backend-1 web-backend-1.example.com check
    server backend-2 web-backend-2.example.com check
    server backend-3 backup-backend.example.com check backup
    
backend forum
    balance leastconn
    server forum-1 forum-1.example.com check
    server forum-2 forum-2.example.com check
    server forum-3 backup-forum.example.com check backup
```

- Dòng balance leaseconn chỉ ra rằng thuật toán để cân bằng tải là chọn những server có ít kết nối tới nhất.

- Dòng mode http chỉ ra rằng proxy sẽ chỉ cân bằng cho các kết nối tại tầng 7 của Internet Layer (Tầng ứng dụng)

## Frontend

Frontend được dùng để định nghĩa cách mà các request được điều hướng cho backend. 


Frontend được định nghĩa trong mục frontend của HAProxy configuration. Các cấu hình cho frontend gồm:

*Một bộ địa chỉ IP và port (ví dụ: 10.0.0.1:8080, *:443,...)*
*Các ACL do người dùng định nghĩa*
*Backend được dùng để nhận request*
*Một ví dụ cấu hình của frontend trong file configuration của HAProxy:*


```
frontend web
  bind 0.0.0.0
  default_backend web-backend

frontend forum
  bind 0.0.0.0:8080
  default_backend forum
```

Các loại cân bằng tải


- no-load-balancing

Với mô hình này, người dùng sẽ kết nối trực tiếp với web server tại yourdomain.com và không có cân bằng tải nào được sử dụng. Nếu webserver gặp trục trặc, người dùng sẽ không thể kết nối đến ứng dụng web được nữa.

### Cân bằng tải tại tầng 4 (tầng truyền vận)

Cách đơn giản nhất để có thể cân bằng tải tới nhiều server là sử dụng cân bằng tải trên tầng 4. Theo hướng này thì các request sẽ được điều hướng dựa trên khoảng địa chỉ IP và cổng (ví dụ một request tới địa chỉ http:://www.example.com/something sẽ được điều hướng tới backend được dùng để điều hướng cho domain example.com với cổng 80)

### layer-4-load-balancing

- Cân bằng tải tại tầng 7 (tầng ứng dụng)
- Cân bằng tải tại tầng 7 là cách cân bằng tải phức tạp nhất và cũng là cách cân bằng tải có nhiều tùy biến nhất. Sử dụng cân bằng tải tại tầng 7, ta có thể điều hướng request dựa trên nội dụng của request đó. Với kiểu câng bằng tải này, nhiều backend có thể được sử dụng cho dùng một domain và port.

### layer-7-load-balancing

-  Ví dụ, một người dùng request tới example.com/something, request đó sẽ được điều hướng đến một backend chuyên dụng cho something.

```
frontend web
    bind *:80
    mode http
    
    acl something_url   path    /something
    use_backend something-server if something_url
    
    default_backend web-backend
```

### Các thuật toán cân bằng tải


- roundrobin: các request sẽ được chuyển đến server theo lượt. Đây là thuật toán mặc định được sử dụng cho HAProxy
- leastconn: các request sẽ được chuyển đến server nào có ít kết nối đến nó nhất
- source: các request được chuyển đến server bằng các hash của IP người dùng. Phương pháp này giúp người dùng đảm bảo luôn kết nối tới một server


## Tính sẵn sàng cao


- Cân bằng tải trên tầng 4 và tầng 7 được miêu tả ở trên đều được sử dụng để cân bằng tải để chuyển các request tới các backend server.
-  Tuy nhiên, chính câng bằng tải của bạn là một điểm lỗi (single point of failure), vì nếu cân bằng tải gặp sự cố nhưng các server đều chạy bình thường thì người dùng cũng không thể kết nối đến được ứng dụng web trong khi server vẫn đang chạy bình thường.

- Một cài đặt cấu hình cao sẽ không có một điểm lỗi nào. Nó ngăn chặn việc nếu một máy chủ không hoạt động làm ảnh hưởng tới toàn bộ hệ thống.haproxy

=>> Với mô hình có 2 cân bằng tải như trên, nếu một trong 2 cân bằng tải không hoạt động, ứng dụng vẫn có thể hoạt động bình thường. Hay nếu một trong hai server không hoạt động, server còn lại sẽ có thể chịu tải thay cho nó.


