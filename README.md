# BÁO CÁO THỰC HÀNH HỆ QUẢN TRỊ CƠ SỞ DỮ LIỆU
**Họ và tên:** Phạm Thị Vân Anh
**Mã sinh viên:** K235480106003
**Cơ sở dữ liệu:** `QLSV_VanAnh`

---

## HƯỚNG DẪN LÀM BÀI VÀ KẾT QUẢ THỰC HIỆN

#### 1. Tạo cơ sở dữ liệu
Sử dụng giao diện đồ họa (SSMS) để tạo một Database mới có tên là `QLSV_VanAnh`.
`[CHÈN ẢNH BƯỚC 1]`

#### 2. Tạo bảng dữ liệu
Tạo bảng `svtnut` trong Database vừa tạo.
`[CHÈN ẢNH BƯỚC 2]`

#### 3. Thêm cột dữ liệu
Thêm các cột tương ứng để lưu trữ thông tin sinh viên.
`[CHÈN ẢNH BƯỚC 3]`

#### 4. Sửa kiểu dữ liệu
Điều chỉnh kiểu dữ liệu của các trường cho phù hợp (Ví dụ: `nvarchar` cho các trường văn bản).
`[CHÈN ẢNH BƯỚC 4]`

#### 5. Thiết lập Khóa chính
Cài đặt trường `masv` làm khóa chính (Primary Key) cho bảng `svtnut`.
`[CHÈN ẢNH BƯỚC 5]`

#### 6. Xác định vị trí lưu trữ vật lý
Kiểm tra và xác định đường dẫn lưu trữ hai tệp vật lý `.mdf` và `.ldf` của Database.
`[CHÈN ẢNH BƯỚC 6]`

#### 7. Script tạo bảng
Sử dụng chức năng sinh mã của SSMS để tạo ra đoạn Code SQL thiết lập bảng `svtnut` với khóa chính.
`[CHÈN ẢNH BƯỚC 7]`

#### 8. Import dữ liệu từ file CSV
Sử dụng tính năng Import Flat File để nạp dữ liệu mẫu vào bảng. Đã thực hiện mapping lại các cột (`NULL` -> `noisinh`, `NULL2` -> `diachi`, `sdt` -> `malop`) để khớp với cấu trúc.
`[CHÈN ẢNH BƯỚC 8]`

#### 9. Kiểm tra tổng số dòng dữ liệu
Sử dụng lệnh T-SQL để đếm tổng số bản ghi sau khi import, đảm bảo nạp đủ 12.020 dòng.
* **Lệnh thực hiện:**
  ```sql
  SELECT COUNT(*) AS TongSoSinhVien FROM svtnut;
  10. Chèn thông tin cá nhân (Insert)
Thực hiện thêm mới (insert) 1 bản ghi vào bảng với thông tin cá nhân của người làm bài báo cáo.

Lệnh thực hiện:

SQL
INSERT INTO svtnut (masv, hotensv, ngaysinh) VALUES ('K235480106003', N'Phạm Thị Vân Anh', '15/08/2005');
[CHÈN ẢNH BƯỚC 10]

11. Cập nhật dữ liệu khuyết thiếu (Update)
Cập nhật trường noisinh thành "Sao Hoả" cho những bản ghi bị khuyết thiếu cả thông tin nơi sinh và địa chỉ.

Lệnh thực hiện:

SQL
UPDATE svtnut SET noisinh = N'Sao Hỏa' WHERE noisinh IS NULL AND diachi IS NULL;
[CHÈN ẢNH BƯỚC 11]

12. Phân tách dữ liệu sang bảng phụ (Select Into)
Tạo bảng mới tên là SaoHoa chứa danh sách các sinh viên có nơi sinh ở "Sao Hoả".

Lệnh thực hiện:

SQL
SELECT * INTO SaoHoa FROM svtnut WHERE noisinh = N'Sao Hỏa';
[CHÈN ẢNH BƯỚC 12]

13. Lọc và xóa sinh viên cùng họ (Delete)
Thực hiện xóa (delete) khỏi bảng SaoHoa những sinh viên có cùng họ "Phạm" với người làm bài.

Lệnh thực hiện:

SQL
DELETE FROM SaoHoa WHERE hotensv LIKE N'Phạm %';
[CHÈN ẢNH BƯỚC 13]

14. Xuất Script sao lưu toàn diện (Schema and Data)
Sử dụng tính năng Generate Scripts xuất toàn bộ cấu trúc CSDL và dữ liệu ra tệp dulieu.sql.

Cấu hình: Advanced -> Types of data to script: Schema and data.
[CHÈN ẢNH BƯỚC 14]

15. Mô phỏng mất dữ liệu và kiểm tra file vật lý
Xoá CSDL QLSV_VanAnh trên giao diện SSMS. Sau đó, truy cập vào thư mục vật lý để kiểm tra trạng thái tồn tại của 2 file .mdf và .ldf.
[CHÈN ẢNH BƯỚC 15: Ảnh chụp thư mục hiển thị file vật lý]

16. Hồi sinh hệ thống từ tệp Script dự phòng
Mở file dulieu.sql (tạo ở Bước 14) và Execute toàn bộ lệnh để khôi phục lại CSDL cùng các bảng dữ liệu bên trong.
[CHÈN ẢNH BƯỚC 16: Ảnh chụp Object Explorer sau khi khôi phục]
