# network_monitoring
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

Kiểm tra lại trên router
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

# LẤY DỮ LIỆU TỪ PRTG VÀ LƯU VÀO THƯ MỤC CHIA SẺ

Phần này sẽ mô tả cách thức để máy PC ảo tự động gửi yêu cầu (request) đến API của PRTG nhằm lấy dữ liệu từ các sensor, sau đó lưu dữ liệu này dưới dạng file CSV vào thư mục chia sẻ (Shared Folder) giữa PC ảo và PC thật. Đây là quá trình quan trọng để đảm bảo rằng dữ liệu mạng được cập nhật liên tục, sẵn sàng để PC thật truy cập và lưu trữ vào cơ sở dữ liệu MySQL để phục vụ cho các công cụ giám sát như Grafana.
- Để truy cập dữ liệu của PRTG qua API, trước tiên chúng ta cần tạo một API token nhằm cấp quyền truy cập bảo mật cho các yêu cầu (requests)
<img width="940" height="248" alt="image" src="https://github.com/user-attachments/assets/3130935a-6820-4453-80db-9fc7208c67b7" />

- Request API của PRTG dưới dạng dữ liệu lịch sử (Historic Data) : PRTG cho phép truy cập dữ liệu lịch sử của sensor qua API bằng cách gọi đến endpoint /api/historicdata.csv. Dữ liệu này giúp người dùng lấy thông tin đã lưu trữ từ trước, với tùy chọn thời gian cụ thể để phân tích.

<img width="940" height="235" alt="image" src="https://github.com/user-attachments/assets/82a6fa91-fe34-41d9-91cc-f530c9007538" />

=> Ví dụ, với sensor ID là 2088, thời gian từ sdate=2023-10-01-10-00-00 đến edate=2023-10-01-10-02-00, ta có thể xây dựng URL như sau: http://192.168.205.100:8080/api/historicdata.csv?id=2088&avg=0&sdate=2023-10-01-10-00-00&edate=2023-10-01-10-02-00&apitoken=UL65L4A5G55G6U7EKO7QNXFDJNHBZKIFZ3BQUY5G4M======

- Viết chương trình gửi request API để tải file CSV
Mục tiêu: Tải dữ liệu từ các sensor của PRTG, lưu dưới dạng file CSV vào thư mục chia sẻ giữa hai máy tính. Dữ liệu này sẽ được sử dụng để giám sát lưu lượng mạng.
Phương pháp: Sử dụng script Python để gọi API của PRTG theo chu kỳ, tải dữ liệu mới nhất từ các sensor.
=> Hệ thống tải dữ liệu từ các sensor của PRTG một cách tự động và lưu trữ vào thư mục chia sẻ giữa PC ảo và PC thật. Đoạn mã sử dụng vòng lặp vô hạn để liên tục tải dữ liệu sau mỗi 2 phút, đảm bảo rằng dữ liệu luôn được cập nhật kịp thời. Các file CSV sau đó sẽ được PC thật truy cập, lưu trữ vào cơ sở dữ liệu MySQL và phục vụ cho quá trình trực quan hóa dữ liệu trên Grafana.
# ĐỌC VÀ LƯU TRỮ DỮ LIỆU ĐÃ XỬ LÝ VÀO MYSQL
Trong phần này, PC thật sẽ truy cập vào thư mục chia sẻ (Shared Folder) để đọc các file CSV do PC ảo tải về từ API của PRTG. Sau đó, PC thật sẽ xử lý dữ liệu, phân tích các giá trị cần thiết, và lưu vào cơ sở dữ liệu MySQL để phục vụ cho mục đích giám sát mạng.
Source code ở mục Code.zip
Phần này đã thiết lập quy trình tự động cho phép PC thật:
⮚	Truy cập vào các thư mục chia sẻ để lấy dữ liệu từ các file CSV.
⮚	Phân tích và xử lý dữ liệu, chuyển đổi định dạng thời gian, và lấy các giá trị cần thiết.
⮚	Lưu dữ liệu đã xử lý vào cơ sở dữ liệu MySQL để có thể dễ dàng theo dõi và giám sát.
Với việc sử dụng threading, hệ thống có khả năng xử lý song song dữ liệu "Traffic In" và "Traffic Out", đảm bảo cập nhật liên tục và nhanh chóng vào cơ sở dữ liệu, phục vụ cho mục đích giám sát mạng lưới của hệ thống.

# SỬ DỤNG GRAFANA ĐÃ KẾT NỐI MYSQL ĐỂ VẼ BIỂU ĐỒ

Sau khi dữ liệu lưu lượng mạng đã được lưu vào cơ sở dữ liệu MySQL, bước tiếp theo là sử dụng Grafana kết nối với MySQL, lấy dữ liệu và tạo các biểu đồ theo dõi lưu lượng. Grafana là công cụ phổ biến để trực quan hóa dữ liệu thời gian thực và phù hợp cho việc giám sát dữ liệu mạng.
Sau khi đã tạo một Dashboard thành công, ta sẽ có phần cấu hình panel như sau:
<img width="940" height="448" alt="image" src="https://github.com/user-attachments/assets/8d23866d-e8f5-4c95-8cdc-fbe0dc47d0a6" />

-	Trong phần này, Grafana cho ta hai chức năng để truy vấn dữ liệu có thể sẽ dụng phương pháp chọn (Builder) hoặc viết code MySQL (Code).
-	Hình trên thể hiện cho ta cách sử dụng chức năng Builder để truy vấn dữ liệu, những thông số ta cần phải quan tâm:
●	Dataset: Database mà ta tạo ra để lưu trữ dữ liệu.
●	Table: Bảng dữ liệu cụ thể lưu trữ data của mỗi sensor.
●	Column: Các cột dữ liệu ta cần để vẽ biểu đồ. Ở đây ta sẽ vẽ hai giá trị traffic in và traffic out. Còn datetime sẽ dùng để hiển thị thời gian tại mỗi giá trị của traffic in và traffic out.
-	Sau khi đã chọn xong các mục ta chọn Run Query để thực hiện truy vấn.

<img width="940" height="434" alt="image" src="https://github.com/user-attachments/assets/5982e06f-e01c-4365-9c6c-3247d0084a45" />

-	Dưới đây là biểu đồ Grafana vẽ được sau khi thực hiện truy vấn từ các lựa chọn trên:

<img width="940" height="265" alt="image" src="https://github.com/user-attachments/assets/c3d8f478-4e79-4a0c-9cf0-d25444fe59f2" />

<img width="940" height="325" alt="image" src="https://github.com/user-attachments/assets/01d245b9-753c-4df8-91e0-5eb8f886df87" />

# NHÚNG GRAFANA VÀO WEB VÀ CÁC VẤN ĐỀ LIÊN QUAN

Grafana cung cấp cho ta tính năng có thể nhúng các biểu đồ vào web cá nhân của mình, chỉ cần sử dụng mã HTML để tạo iframe, trong đó đường dẫn tới dashboard Grafana được chỉ định.
Giao diện web

<img width="940" height="486" alt="image" src="https://github.com/user-attachments/assets/014e9ef2-fb55-423f-9e53-f5fd29d918b2" />

Cài đặt Grafana để cho phép nhúng
Sau khi thực hiện các bước trên, ta vẫn chưa thể nhúng được Dashboard vào web, cần phải thực hiện một số bước cấu hình trên file defaults.ini của Grafana.
Truy cập Program Files -> GrafanaLabs -> grafana -> conf -> defaults.ini (Cần truy cập chỉnh sửa dưới quyền Admin).
Tại thư mục defaults.ini ta thực hiện các chỉnh sửa sau:
-	Cho phép nhúng bằng cách thay đổi giá trị allow_embedding thành true
allow_embedding = true
-	Cấu hình CORS để chấp nhận yêu cầu từ trang web
[cors] 
enabled = true 
allow_origins = http://localhost:8000 # URL của trang web 
allow_credentials = true
●	enabled = true: Cho phép yêu cầu CORS từ các nguồn khác.
●	allow_origins: Chỉ định các URL được phép gửi yêu cầu nhúng vào Grafana, ở đây là http://localhost:8000.
●	allow_credentials = true: Cho phép gửi cookie và thông tin xác thực cùng với yêu cầu CORS.
-	Bật truy cập ẩn danh để cho phép người dùng xem dashboard mà không cần đăng nhập
[auth.anonymous] 
enabled = true

Video demo 1: https://drive.google.com/file/d/14hBder7vrHuLllsAwGMQVhlVBDafzqZK/view?usp=drive_link

=> Thiết lập PRTG để có dữ liệu, cài đặt Grafana và tiến hành lấy dữ liệu. Vấn đề xảy ra khi chưa bật Anonymous: Enable = True

Video demo 2: https://drive.google.com/file/d/103pX5qK2dR7ULLfvTtKku-8iISAKK-Tq/view?usp=drive_link
