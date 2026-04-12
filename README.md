Bước 1: Tải và cài đặt SQL Server 2025 (phiên bản Developer), chọn tất cả các tính năng mở rộng.
Bước 2: Cấu hình bảo mật: Thiết lập "Mixed Mode" (Windows Authentication và SQL Server Authentication - sa/123).<img width="975" height="1073" alt="image" src="https://github.com/user-attachments/assets/acfecbe1-f29a-4632-a08b-74a8ac97350d" />

Bước 3: Cấu hình kết nối: Thiết lập TCP/IP, chọn cổng động (Dynamic Port) theo MSSV (3xxxx, 4xxxx, hoặc 5xxxx).<img width="966" height="1181" alt="image" src="https://github.com/user-attachments/assets/5a007cdf-6a74-4758-b516-61e7bbedc0b7" />

Bước 4: Kiểm tra trạng thái: Sử dụng lệnh netstat -ano trên CMD để xác nhận dịch vụ SQL Server đang chạy và mở đúng cổng đã đặt.<img width="975" height="554" alt="image" src="https://github.com/user-attachments/assets/0f49b2d8-1464-4681-b2d3-bfd2f342b816" />

Bước 5: Cài đặt SQL Server Management Studio (SSMS) để quản trị cơ sở dữ liệu.<img width="975" height="506" alt="image" src="https://github.com/user-attachments/assets/aa651862-ec1d-4443-9554-17661b47ee3f" />

Bước 6: Tạo Database: Tạo CSDL mới tại ổ đĩa khác ổ C, xác nhận sự tồn tại của file dữ liệu (.mdf) và file log (.ldf).<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f8ae1531-6aae-4fd8-b1aa-0a7e55105843" />

Bước 7: Thiết kế Table: Tạo bảng dữ liệu mới, đặt khóa chính (masv) và các trường phù hợp với file CSV mẫu.<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/09eba67a-4d5e-42f5-9e42-47630bf08b2a" />

Bước 8: Nhập dữ liệu: Sử dụng tính năng Import của SSMS để đưa dữ liệu từ file CSV vào bảng.<img width="975" height="518" alt="image" src="https://github.com/user-attachments/assets/dae0b82c-8830-4651-b40c-9378178f854c" />

Bước 9: Kiểm tra dữ liệu: Dùng lệnh SELECT COUNT(*) để xác nhận đã import thành công khoảng 12020 dòng.<img width="612" height="427" alt="image" src="https://github.com/user-attachments/assets/74d711e4-d959-4ee1-a1e6-557e3a19126f" />

Bước 10: Thêm dữ liệu (Insert): Chạy lệnh SQL thêm một hàng dữ liệu chứa thông tin cá nhân của bản thân.<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3aeb57b5-015a-4203-9114-5553812ab902" />

Bước 11: Cập nhật dữ liệu (Update): Chạy lệnh SQL chuyển noisinh thành 'Sao Hoả' cho các bản ghi có noisinh và diachi là NULL.<img width="975" height="524" alt="image" src="https://github.com/user-attachments/assets/682aa863-e8ca-4e59-b3ac-a8e610509362" />

Bước 12: Tạo bảng mới (SELECT INTO): Tạo bảng SaoHoa chứa các sinh viên có nơi sinh là 'Sao Hoả'.<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/836c5695-99a9-4e5f-8ed7-316236d7b324" />

Bước 13: Xóa dữ liệu (Delete): Chạy lệnh SQL xóa các sinh viên có cùng họ với bản thân trong bảng SaoHoa.<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8de41123-f7b0-45db-b804-fbcd2bafb124" />

Bước 14: Xuất dữ liệu (Generate Scripts): Sử dụng tính năng "Generate Scripts" trong SSMS để xuất toàn bộ cấu trúc và dữ liệu ra file dulieu.sql.<img width="497" height="785" alt="image" src="https://github.com/user-attachments/assets/57aaf56d-19de-400a-8d89-ae69941db099" />

Bước 15: Kiểm tra xóa & Phục hồi: Xóa Database, kiểm tra file vật lý, sau đó chạy lại file dulieu.sql để phục hồi và kiểm chứng.<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5cb66cd6-6270-464f-83bb-673897d8e0e2" />
Bước 16: Tạo cửa sổ mới để gõ lệnh: mở file dulieu.sql của bước 14, chạy toàn bộ các lệnh này. REFRESH lại cây liệt kê các database => kiểm chứng kết quả được tạo ra tương đương với các bước 6,7,8,9,10,11,12,13.
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/24cfb507-0ce0-4574-9b81-13fb1c8f572c" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/57f09158-eaa8-44f7-9233-a590557cb1e8" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/bd741e72-b688-4ced-bb98-63fea4ace139" />


