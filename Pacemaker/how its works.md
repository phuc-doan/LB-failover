## <a name="pacemaker"> Pacemaker</a>

- pacemaker là một cluster quản lý các resource, nó có khả năng hoạt động với hầu hết các dịch vụ cluster bằng cách phát hiện và phục hồi từ node và resource-level bằng các sử dụng khả năng trao đổi và các mối quan hệ được cung cấp bởi kiến trúc hạ tầng ưa thích của bạn ( Corosync hoặc Heartbeat). [Xem thêm](pcmk-pacemaker-overview.md)

- Tính năng của pacemaker bao gồm:

-  Dò tìm và và khôi phục các dịch vụ lỗi
    + Không yêu cầu chia sẻ không gian lưu trữ
    
    + Dảm bảo tính toàn vẹn dữ liệu
    
    + Hỗ trợ những cluster lớn và nhỏ
    
    + Hỗ trợ hầu hết bất cứ cấu hình dự phòng nào
    
    + Tự động tạo bản sao cấu hình vì vậy có thể cập nhật từ bất kì node nào
    
    + Hỗ trợ những kiểu dịch vụ được mở rộng

    + Thống nhất, có kịch bản, những công cụ quản lý cluster.          

			
 ## <a name="corosync"> Corosync</a>
 
 -  Pacemaker sử dụng corosync để liên lạc bên trong cluster, xác định node lỗi và chuyển tài nguyên sang những node đang hoạt động.

    + Là một cơ sở hạ tầng mức độ thấp cung cấp thông tin tin cậy, thành viên và những thông tin quy định về cluster
	 
    + Trong các mức độ của các ha cluster stack hiện tại, corosync là một giải pháp mặc định. Điều này có nghĩa rằng ta nên sử dụng corosync trong mọi trường hợp. Đôi khi, trong một vài trường hợp đặc biệt, corosync sẽ không làm việc.
