# hbkhanh_network
Mô hình cơ bản của hệ thống.
<img width="940" height="353" alt="image" src="https://github.com/user-attachments/assets/dfe6fb07-82be-4ca8-ba59-e7ae91cd3cfc" />

Trong các máy ảo, hai chế độ mạng phổ biến là Host-Only và NAT. 

1. Host-Only Networking
- Cách hoạt động: Chế độ mạng Host-Only cho phép máy ảo kết nối với máy chủ vật lý (host) nhưng không có kết nối ra ngoài Internet.
- Mục đích sử dụng: Phù hợp khi bạn muốn máy ảo chỉ giao tiếp với máy chủ hoặc các máy ảo khác trong cùng mạng Host-Only mà không có nhu cầu truy cập Internet.
- Ưu điểm: Tăng cường bảo mật, vì các máy ảo không kết nối được ra ngoài hoặc bị ảnh hưởng bởi bên ngoài.
- Nhược điểm: Máy ảo không có quyền truy cập Internet, nên không thể cập nhật hoặc tải về dữ liệu từ bên ngoài.

2. NAT (Network Address Translation)
- Cách hoạt động: Chế độ NAT cho phép máy ảo sử dụng IP nội bộ để truy cập ra Internet thông qua IP của máy chủ (host). Máy chủ sẽ chuyển tiếp các gói tin của máy ảo ra ngoài mạng.
- Mục đích sử dụng: Thích hợp khi bạn cần máy ảo có kết nối Internet, nhưng không muốn chúng trực tiếp nằm trong mạng nội bộ mà host đang dùng.
- Ưu điểm: Máy ảo có thể truy cập Internet để cập nhật hoặc tải dữ liệu, nhưng không hiển thị ra ngoài mạng nội bộ, tăng tính an toàn.
- Nhược điểm: Do sử dụng IP của máy chủ để chuyển tiếp gói tin, máy ảo có thể bị hạn chế về một số cổng hoặc dịch vụ tùy thuộc vào cài đặt NAT.

Tóm lại:
- Host-Only phù hợp cho môi trường kín, chỉ có kết nối nội bộ.
- NAT thích hợp khi máy ảo cần Internet, nhưng không muốn tiếp xúc với các máy khác trong mạng của máy chủ vật lý.

<img width="942" height="827" alt="image" src="https://github.com/user-attachments/assets/3196ad1c-0c52-4d05-8699-fb6d22e58d83" />


Cấu hình trên Router
<img width="975" height="578" alt="image" src="https://github.com/user-attachments/assets/8c48dc0d-05d8-4330-b250-9b2aea238f9d" />


Dùng 2 net đại diện cho Internet bên ngoài và mạng nội bộ bên trong vì để giảm độ trễ nếu đặt hết tất cả các thiết bị PC ảo và cài đặt giám sát môi trường ảo trên Lab EVE sẽ gây ra độ trễ rất lớn

Câu lệnh config trên Router:

# Với interface trên Router
Router(config)#interface e0/1
Router(config-if)#ip address dhcp
# Kiểm tra địa chỉ ip được cấp
Router#show ip interface brief
# Đặt ip gateway ra ngoài Internet
Router(config-if)#ip nat outside
Router(config-if)#no shutdown



# Đặt ip gateway nội bộ
Router(config)#interface e0/0
Router(config-if)#ip address 192.168.205.1 255.255.255.0
Router(config-if)#ip nat inside
Router(config-if)#no shutdown
# Định tuyến ra gateway card mạng NAT của VMWare
Router(config)#ip route 0.0.0.0 0.0.0.0 192.168.100.96
# Tạo NAT Overload qua interface
Router(config)#access-list 10 permit any
Router(config)#ip nat inside source list 10 interface e0/1 overload
# Kích hoạt SNMP bằng lệnh sau:
Router(config)#snmp-server community ro

#Kiểm tra lại trên router
<img width="975" height="230" alt="image" src="https://github.com/user-attachments/assets/c682bab1-bbc6-45f4-bde1-7283e3d0ce05" />

Ta cài đặt PRTG và được giao diện như sau
 <img width="975" height="407" alt="image" src="https://github.com/user-attachments/assets/c283dfbc-7c4a-485d-b5c3-95c0e0498532" />

Vào Device -> Network Discovery -> Chọn các cổng e0/1, e0/0 và thêm cái sensor chúng ta muốn giám sát vào.

Thiết lập giám sát địa chỉ Router trên PRTG:
<img width="975" height="449" alt="image" src="https://github.com/user-attachments/assets/c92219cf-857b-4b21-be69-f733ea9c6825" />
Dùng Sensor sau:
SNMP Traffic 64bit
<img width="975" height="435" alt="image" src="https://github.com/user-attachments/assets/46c11f72-bb7e-4629-a20e-c3e8ff15b988" />
Kết quả thu được:
<img width="975" height="213" alt="image" src="https://github.com/user-attachments/assets/a84dfba6-f74d-4db1-9453-4f00196cc931" />
=> Tuy nhiên thì ta thấy đồ thị vẽ không được đẹp biểu hiện không rõ.

#LẤY DỮ LIỆU TỪ PRTG VÀ LƯU VÀO THƯ MỤC CHIA SẺ

Phần này sẽ mô tả cách thức để máy PC ảo tự động gửi yêu cầu (request) đến API của PRTG nhằm lấy dữ liệu từ các sensor, sau đó lưu dữ liệu này dưới dạng file CSV vào thư mục chia sẻ (Shared Folder) giữa PC ảo và PC thật. Đây là quá trình quan trọng để đảm bảo rằng dữ liệu mạng được cập nhật liên tục, sẵn sàng để PC thật truy cập và lưu trữ vào cơ sở dữ liệu MySQL để phục vụ cho các công cụ giám sát như Grafana.
- Để truy cập dữ liệu của PRTG qua API, trước tiên chúng ta cần tạo một API token nhằm cấp quyền truy cập bảo mật cho các yêu cầu (requests)
<img width="940" height="248" alt="image" src="https://github.com/user-attachments/assets/3130935a-6820-4453-80db-9fc7208c67b7" />

- Request API của PRTG dưới dạng dữ liệu lịch sử (Historic Data) : PRTG cho phép truy cập dữ liệu lịch sử của sensor qua API bằng cách gọi đến endpoint /api/historicdata.csv. Dữ liệu này giúp người dùng lấy thông tin đã lưu trữ từ trước, với tùy chọn thời gian cụ thể để phân tích.

<img width="940" height="235" alt="image" src="https://github.com/user-attachments/assets/82a6fa91-fe34-41d9-91cc-f530c9007538" />

=> Ví dụ, với sensor ID là 2088, thời gian từ sdate=2023-10-01-10-00-00 đến edate=2023-10-01-10-02-00, ta có thể xây dựng URL như sau: http://192.168.205.100:8080/api/historicdata.csv?id=2088&avg=0&sdate=2023-10-01-10-00-00&edate=2023-10-01-10-02-00&apitoken=UL65L4A5G55G6U7EKO7QNXFDJNHBZKIFZ3BQUY5G4M======

- Viết chương trình gửi request API để tải file CSV
Mục tiêu: Tải dữ liệu từ các sensor của PRTG, lưu dưới dạng file CSV vào thư mục chia sẻ giữa hai máy tính. Dữ liệu này sẽ được sử dụng để giám sát lưu lượng mạng.
Phương pháp: Sử dụng script Python để gọi API của PRTG theo chu kỳ, tải dữ liệu mới nhất từ các sensor.

