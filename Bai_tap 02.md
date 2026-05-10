Họ Tên: Phạm Thị Vân Anh

MSSV: K235480106003

Lớp: K59KMT.K01

Nội dung: Bài tập 02

# Phần 1: Thiết kế và Khởi tạo Cấu trúc Dữ liệu
```
-- 1. Tạo Database mới theo đúng tên dự án và MSSV


CREATE DATABASE [QuanLyKhachSan_k235480106003];

GO

-- Chuyển sang sử dụng Database vừa tạo

USE [QuanLyKhachSan_k235480106003];

GO


-- 2. Tạo bảng [LoaiPhong] (Bảng chứa thông tin các loại phòng như VIP, Thường...)

CREATE TABLE [LoaiPhong] (

    [MaLoaiPhong] INT PRIMARY KEY IDENTITY(1,1), -- PK: Khóa chính, tự tăng
    
    [TenLoaiPhong] NVARCHAR(100) NOT NULL,        -- Chuỗi Unicode
    
    [DonGiaTheoNgay] MONEY NOT NULL,              -- Kiểu tiền tệ
    
    -- CK: Đơn giá phải lớn hơn 0
    
    CONSTRAINT [CK_DonGia_Duong] CHECK ([DonGiaTheoNgay] > 0)
);
GO

-- 3. Tạo bảng [Phong] (Thông tin chi tiết từng phòng)

CREATE TABLE [Phong] (

    [MaPhong] INT PRIMARY KEY IDENTITY(1,1),      -- PK: Khóa chính
    
    [SoPhong] VARCHAR(10) NOT NULL,                -- Số phòng (ví dụ: P101)
    
    [TrangThai] NVARCHAR(50) DEFAULT N'Trống',    -- Trạng thái phòng
    
    [MaLoaiPhong] INT,                            -- Khóa ngoại

    -- FK: Nối tới bảng LoaiPhong
    
    CONSTRAINT [FK_Phong_LoaiPhong] FOREIGN KEY ([MaLoaiPhong]) 
    
    REFERENCES [LoaiPhong]([MaLoaiPhong])
    
);

GO

-- 4. Tạo bảng [KhachHang] (Thông tin khách thuê)

CREATE TABLE [KhachHang] (

    [MaKhachHang] INT PRIMARY KEY IDENTITY(1,1),  -- PK
    
    [HoTenKhach] NVARCHAR(200) NOT NULL,          -- Quy tắc BướuLạcĐà
    
    [SoDienThoai] VARCHAR(15),                    -- Chuỗi ký tự
    
    [NgaySinh] DATE,                              -- Kiểu ngày tháng
    
    [DiemTichLuy] FLOAT DEFAULT 0,                -- Số thực

    -- CK: Điểm tích lũy không được âm
    CONSTRAINT [CK_DiemTichLuy_KhongAm] CHECK ([DiemTichLuy] >= 0)
);
GO
```
<img width="1911" height="966" alt="image" src="https://github.com/user-attachments/assets/fb5ed42a-6e45-4a29-8944-4500171dd9f0" />
Ảnh tạo trên database tên là QuanLyKhachSan cùng mssv của em
<img width="1911" height="979" alt="image" src="https://github.com/user-attachments/assets/9a32d475-9bf2-469f-b025-c9562e4d49b6" />
Ảnh này tạo 3 bảng theo yêu cầu của đề bài: Sử dụng đúng quy tắc đặt tên (BuouLacDa), sử dụng dấu [] đẻ đọc bảng tên và trường tên trong tập lệnh khởi tạo, có giải thích bất kì trong PK,FK,CK

# Phần 2: Xây dựng chức năng

* 1. Các loại Function Built-in trong SQL Server
SQL Server cung cấp một hệ thống hàm phong phú để xử lý dữ liệu nhanh chóng mà không cần viết code phức tạp. Có 3 nhóm chính thường dùng:

Scalar Functions (Hàm vô hướng): Trả về một giá trị duy nhất (Ví dụ: hàm xử lý chuỗi, toán học).

Aggregate Functions (Hàm tập hợp): Tính toán trên một cột dữ liệu (Ví dụ: SUM, AVG, COUNT).

System Functions (Hàm hệ thống): Truy xuất thông tin về cấu hình, tài khoản hoặc đối tượng trong Database.

* 2. Mục đích của Hàm do người dùng tự viết (User-Defined Functions - UDF)
Hàm tự viết sinh ra để giải quyết những yêu cầu đặc thù mà các hàm có sẵn không có. Mục đích chính bao gồm:
Tái sử dụng mã nguồn (Reuse): Viết một công thức phức tạp một lần và gọi lại ở nhiều nơi (Stored Procedure, View, Query).

Đơn giản hóa câu lệnh SQL: Thay vì viết một đoạn tính toán dài 10 dòng trong câu lệnh SELECT, bạn chỉ cần gọi tên hàm.

Đóng gói logic nghiệp vụ: Đảm bảo các quy tắc tính toán (ví dụ: cách tính lương, thuế của riêng công ty bạn) được áp dụng thống nhất.

* Phân Loại
Scalar Function (Hàm vô hướng)	Một giá trị duy nhất (số, chuỗi, ngày...)	Dùng để tính toán các công thức toán học, xử lý chuỗi hoặc chuyển đổi định dạng dữ liệu đơn lẻ.

Inline Table-Valued Function (Hàm giá trị bảng trực tiếp)	Một tập hợp dữ liệu (bảng)	Dùng để lọc dữ liệu phức tạp. Nó hoạt động giống như một View có tham số, giúp lấy ra một danh sách bản ghi dựa trên đầu vào.

Multi-statement Table-Valued Function	Một bảng (có cấu trúc tự định nghĩa)	Dùng khi cần xử lý logic cực kỳ phức tạp, phải qua nhiều bước trung gian trước khi trả về một bảng kết quả cuối cùng.

Tại sao cần tự viết hàm khi đã có hàng trăm System Function?
Dù SQL Server rất mạnh, nhưng nó vẫn cần UDF vì:

Logic nghiệp vụ riêng biệt: System function chỉ giải quyết các vấn đề chung (toán học, thời gian). Nó không biết cách tính "Điểm rèn luyện" hay "Phí lưu kho" theo quy định riêng của trường học hay doanh nghiệp của bạn.

Tính linh hoạt: Bạn có thể kết hợp nhiều system function lại với nhau để tạo ra một chức năng mới phục vụ đúng mục đích công việc hiện tại.

Dễ bảo trì: Khi công thức tính toán thay đổi (ví dụ: thay đổi mức thuế), bạn chỉ cần sửa trong hàm duy nhất thay vì đi tìm và sửa hàng trăm câu lệnh SQL rải rác trong hệ thống.
Yêu cầu: Trong hệ thống quản lý phòng Lab của sinh viên Phạm Thị Vân Anh, mỗi thiết bị khi nhập về đều có năm sản xuất. Nhà quản lý cần một hàm tự động tính toán:

Nếu thiết bị đã sử dụng trên 5 năm: Trả về "Cần thay thế".

Nếu thiết bị sử dụng từ 3 đến 5 năm: Trả về "Cần bảo trì".

Nếu dưới 3 năm: Trả về "Hoạt động tốt"
```
-- 1. Tạo hàm (Scalar Function)
CREATE FUNCTION fn_DanhGiaThietBi (@NamSanXuat INT)
RETURNS NVARCHAR(50)
AS
BEGIN
    DECLARE @SoNamSuDung INT;
    DECLARE @KetQua NVARCHAR(50);
    
    -- Tính số năm tính đến năm hiện tại (2026)
    SET @SoNamSuDung = YEAR(GETDATE()) - @NamSanXuat;
    
    -- Logic đánh giá
    IF @SoNamSuDung > 5
        SET @KetQua = N'Cần thay thế';
    ELSE IF @SoNamSuDung >= 3
        SET @KetQua = N'Cần bảo trì';
    ELSE
        SET @KetQua = N'Hoạt động tốt';
        
    RETURN @KetQua;
END;
GO

-- 2. Khai thác hàm bằng câu lệnh SELECT
SELECT 
    N'Máy chiếu Sony' AS TenThietBi,
    2019 AS NamSanXuat,
    dbo.fn_DanhGiaThietBi(2019) AS TinhTrangBaoTri,
    N'Máy tính Dell' AS TenThietBi2,
    2025 AS NamSanXuat2,
    dbo.fn_DanhGi
```
<img width="1912" height="979" alt="image" src="https://github.com/user-attachments/assets/ee3c925c-a423-4aab-9204-97028ab9ce9b" />
Ảnh trên đã khai thác hàm SQL

* Yêu cầu: Viết một hàm nhận vào Tên loại thiết bị (ví dụ: 'Máy tính', 'Thiết bị đo') và trả về danh sách toàn bộ các thiết bị thuộc loại đó kèm theo trạng thái hiện tại.
```
-- BƯỚC 1: TẠO DATABASE (Nếu chưa có)
IF NOT EXISTS (SELECT * FROM sys.databases WHERE name = 'QuanLyLab_VanAnh_003')
BEGIN
    CREATE DATABASE QuanLyLab_VanAnh_003;
END
GO

USE QuanLyLab_VanAnh_003;
GO

-- BƯỚC 2: TẠO CẤU TRÚC BẢNG (Để không bị báo lỗi "Invalid object name")
IF OBJECT_ID('ThietBi', 'U') IS NULL
BEGIN
    CREATE TABLE LoaiThietBi (
        MaLoai INT PRIMARY KEY IDENTITY(1,1),
        TenLoai NVARCHAR(100) NOT NULL
    );

    CREATE TABLE ThietBi (
        MaTB VARCHAR(10) PRIMARY KEY,
        TenTB NVARCHAR(100) NOT NULL,
        MaLoai INT REFERENCES LoaiThietBi(MaLoai),
        TinhTrang NVARCHAR(50) DEFAULT N'Sẵn sàng'
    );

    -- Thêm dữ liệu mẫu để hàm có cái để lọc
    INSERT INTO LoaiThietBi (TenLoai) VALUES (N'Máy tính');
    INSERT INTO ThietBi (MaTB, TenTB, MaLoai, TinhTrang) VALUES ('TB01', N'Dell Precision', 1, N'Sẵn sàng');
END
GO

-- BƯỚC 3: TẠO HÀM (Đã sửa lỗi cú pháp)
IF OBJECT_ID('dbo.fn_LocThietBiTheoLoai', 'IF') IS NOT NULL
    DROP FUNCTION dbo.fn_LocThietBiTheoLoai;
GO

CREATE FUNCTION dbo.fn_LocThietBiTheoLoai (@TenLoaiInput NVARCHAR(100))
RETURNS TABLE
AS
RETURN 
(
    SELECT 
        T.MaTB AS [Mã Thiết Bị], 
        T.TenTB AS [Tên Thiết Bị], 
        L.TenLoai AS [Loại Thiết Bị], 
        T.TinhTrang AS [Trạng Thái]
    FROM ThietBi T
    INNER JOIN LoaiThietBi L ON T.MaLoai = L.MaLoai
    WHERE L.TenLoai = @TenLoaiInput
);
GO

-- BƯỚC 4: KHAI THÁC HÀM (Chụp ảnh kết quả ở đây)
SELECT * FROM dbo.fn_LocThietBiTheoLoai(N'Máy tính');
GO
```
<img width="1910" height="965" alt="image" src="https://github.com/user-attachments/assets/59060182-7615-41dd-b7a0-da2b2eb2cabb" />
Ảnh trên em đã khai thác hàm SQL

* Yêu cầu: Quản trị viên phòng Lab của bạn cần một báo cáo "Tổng hợp tình trạng thiết bị" nhưng phải đi kèm với phân loại ưu tiên.

Thiết bị nào "Hỏng" -> Mức ưu tiên: 1 (Cao nhất - Cần sửa ngay).

Thiết bị nào "Đang mượn" -> Mức ưu tiên: 2 (Trung bình - Cần theo dõi).

Thiết bị nào "Sẵn sàng" -> Mức ưu tiên: 3 (Thấp - Có thể sử dụng).

Các trường hợp khác -> Mức ưu tiên: 4.
```
USE QuanLyLab_VanAnh_003;
GO

-- 1. Xóa hàm nếu đã tồn tại
IF OBJECT_ID('dbo.fn_BaoCaoUuTienThietBi', 'TF') IS NOT NULL
    DROP FUNCTION dbo.fn_BaoCaoUuTienThietBi;
GO

-- 2. Tạo hàm Multi-statement
CREATE FUNCTION dbo.fn_BaoCaoUuTienThietBi()
RETURNS @BangKetQua TABLE (
    MaTB VARCHAR(10),
    TenTB NVARCHAR(100),
    TinhTrang NVARCHAR(50),
    MucUuTien INT,
    GhiChu NVARCHAR(100)
)
AS
BEGIN
    -- Bước 1: Đổ dữ liệu từ bảng gốc vào biến bảng trung gian @BangKetQua
    INSERT INTO @BangKetQua (MaTB, TenTB, TinhTrang, MucUuTien)
    SELECT MaTB, TenTB, TinhTrang, 4 -- Mặc định mức 4
    FROM ThietBi;

    -- Bước 2: Xử lý logic phức tạp (Cập nhật mức ưu tiên và ghi chú)
    UPDATE @BangKetQua SET MucUuTien = 1, GhiChu = N'GỬI ĐI SỬA GẤP' WHERE TinhTrang = N'Hỏng';
    UPDATE @BangKetQua SET MucUuTien = 2, GhiChu = N'ĐANG CHO MƯỢN' WHERE TinhTrang = N'Đang mượn';
    UPDATE @BangKetQua SET MucUuTien = 3, GhiChu = N'CÓ THỂ SỬ DỤNG' WHERE TinhTrang = N'Sẵn sàng';

    RETURN; -- Trả về bảng đã được xử lý
END;
GO

-- 3. Khai thác hàm (Chụp ảnh kết quả ở đây)
SELECT * FROM dbo.fn_BaoCaoUuTienThietBi()
ORDER BY MucUuTien ASC;
GO
```
<img width="1901" height="972" alt="image" src="https://github.com/user-attachments/assets/4f5cd50a-80c8-4afd-945c-1b7dd522adf5" />
Ảnh trên em đã khai thác hàm SQL

# Phần 3: Xây dựng Store Procedure

* Các loại Stored Procedure trong SQL Server
Có 2 nhóm SP chính mà bạn cần biết:

User-defined Stored Procedures: Là những thủ tục do chính bạn (người dùng) tự viết để xử lý nghiệp vụ riêng (như thêm, sửa, xóa dữ liệu).

System Stored Procedures: Là những thủ tục được tích hợp sẵn khi cài đặt SQL Server. Chúng thường bắt đầu bằng tiền tố sp_ và được lưu trữ trong database master.

* Một số System Stored Procedure đặc sắc
Dưới đây là 3 thủ tục hệ thống cực kỳ hữu ích mà mình tin là bất kỳ quản trị viên nào cũng cần dùng:

-sp_help
Mục đích: Đây là lệnh "vạn năng" để xem thông tin chi tiết của bất kỳ đối tượng nào (Bảng, View, Procedure).

Cách dùng: EXEC sp_help 'Ten_Doi_Tuong';

Tác dụng: Nó sẽ cho bạn biết bảng đó có những cột nào, kiểu dữ liệu gì, khóa chính là gì... mà không cần phải mở Object Explorer.

-sp_helpdb
Mục đích: Xem danh sách và thông tin chi tiết về các Database đang có trên Server.

Cách dùng: EXEC sp_helpdb; (xem tất cả) hoặc EXEC sp_helpdb 'Ten_DB';

Tác dụng: Hiển thị dung lượng file, ngày tạo và trạng thái của Database.

c. sp_rename
Mục đích: Đổi tên đối tượng (Bảng hoặc Cột) mà không cần phải xóa đi tạo lại.

Cách dùng: EXEC sp_rename 'Ten_Cu', 'Ten_Moi';

Tác dụng: Rất hữu ích khi bạn lỡ đặt tên sai hoặc muốn thay đổi quy tắc đặt tên.

* Yêu cầu: Viết một SP có tên sp_CapNhatTrangThaiThietBi để cập nhật cột TinhTrang cho một thiết bị cụ thể.

Tham số đầu vào: Mã thiết bị (@MaTB) và Trạng thái mới (@TrangThaiMoi).

Kiểm tra logic: * Nếu mã thiết bị không tồn tại trong bảng ThietBi, thông báo lỗi và không làm gì cả.

Nếu mã thiết bị tồn tại, thực hiện cập nhật và thông báo thành công.
```
USE QuanLyLab_VanAnh_003;
GO

-- 1. Xóa SP nếu đã tồn tại
IF OBJECT_ID('dbo.sp_CapNhatTrangThaiThietBi', 'P') IS NOT NULL
    DROP PROCEDURE dbo.sp_CapNhatTrangThaiThietBi;
GO

-- 2. Tạo Stored Procedure
CREATE PROCEDURE dbo.sp_CapNhatTrangThaiThietBi
    @MaTB VARCHAR(10),
    @TrangThaiMoi NVARCHAR(50)
AS
BEGIN
    -- Kiểm tra sự tồn tại của thiết bị
    IF NOT EXISTS (SELECT 1 FROM ThietBi WHERE MaTB = @MaTB)
    BEGIN
        PRINT N'Lỗi: Không tìm thấy thiết bị có mã ' + @MaTB;
        RETURN; -- Thoát thủ tục
    END

    -- Nếu tồn tại thì thực hiện cập nhật
    UPDATE ThietBi
    SET TinhTrang = @TrangThaiMoi
    WHERE MaTB = @MaTB;

    PRINT N'Cập nhật trạng thái cho thiết bị ' + @MaTB + N' thành công!';
END;
GO

-- 3. Thực thi thủ tục (Chụp ảnh kết quả ở đây)
-- Trường hợp 1: Thử với mã sai (để xem thông báo lỗi)
EXEC dbo.sp_CapNhatTrangThaiThietBi 'TB_SAI', N'Hỏng';

-- Trường hợp 2: Thử với mã đúng (đã có trong bảng ở các bài trước)
EXEC dbo.sp_CapNhatTrangThaiThietBi 'TB01', N'Đang bảo trì';
GO

-- Kiểm tra lại kết quả trong bảng
SELECT * FROM ThietBi WHERE MaTB = 'TB01';
```
<img width="1903" height="966" alt="image" src="https://github.com/user-attachments/assets/f32551d0-adcf-4e79-b174-23722a23f1a4" />
Đây em đã khai thác thành công 1 Store Procedure

* Yêu cầu: Viết một SP có tên sp_DemThietBiTheoTrangThai.

Tham số đầu vào (INPUT): Tên trạng thái cần đếm (ví dụ: N'Sẵn sàng').

Tham số đầu ra (OUTPUT): Một biến kiểu số nguyên để chứa tổng số thiết bị đếm được.

Mục đích: Giúp lập trình viên lấy được con số cụ thể để hiển thị lên các biểu đồ hoặc thông báo trên giao diện phần mềm.
```
USE QuanLyLab_VanAnh_003;
GO

-- 1. Xóa SP nếu đã tồn tại
IF OBJECT_ID('dbo.sp_DemThietBiTheoTrangThai', 'P') IS NOT NULL
    DROP PROCEDURE dbo.sp_DemThietBiTheoTrangThai;
GO

-- 2. Tạo Stored Procedure với tham số OUTPUT
CREATE PROCEDURE dbo.sp_DemThietBiTheoTrangThai
    @TenTrangThai NVARCHAR(50), -- Tham số vào
    @TongSo INT OUTPUT          -- Tham số ra (có từ khóa OUTPUT)
AS
BEGIN
    SELECT @TongSo = COUNT(*) 
    FROM ThietBi 
    WHERE TinhTrang = @TenTrangThai;
END;
GO

-- 3. Thực thi thủ tục và nhận kết quả (Chụp ảnh kết quả ở đây)
DECLARE @KetQua INT; -- Khai báo biến để nhận giá trị từ OUTPUT

EXEC dbo.sp_DemThietBiTheoTrangThai 
    @TenTrangThai = N'Sẵn sàng', 
    @TongSo = @KetQua OUTPUT; -- Phải có từ khóa OUTPUT ở đây

-- Hiển thị con số cuối cùng
SELECT N'Phạm Thị Vân Anh báo cáo: Số thiết bị đang Sẵn sàng là: ' + CAST(@KetQua AS NVARCHAR(10)) AS [Ket_Qua_Bao_Cao];
GO
```
<img width="1905" height="961" alt="image" src="https://github.com/user-attachments/assets/697bfb9f-b359-4f58-b475-787ae8a7652f" />
Đây em đã khai thác thành công 1 Store Procedure

* Yêu cầu: Viết một SP có tên sp_BaoCaoChiTietThietBi.

Mục đích: Thay vì chỉ xem mã số khô khan, quản trị viên muốn một báo cáo hiển thị: Tên thiết bị, Tên loại thiết bị (từ bảng LoaiThietBi), và Trạng thái.

Kỹ thuật: Sử dụng INNER JOIN để kết nối bảng ThietBi và LoaiThietBi.

Tính năng thêm: Cho phép lọc báo cáo theo một từ khóa tên thiết bị (ví dụ: chỉ hiện các máy có chữ 'Dell').
```
USE QuanLyLab_VanAnh_003;
GO

-- 1. Xóa SP nếu đã tồn tại
IF OBJECT_ID('dbo.sp_BaoCaoChiTietThietBi', 'P') IS NOT NULL
    DROP PROCEDURE dbo.sp_BaoCaoChiTietThietBi;
GO

-- 2. Tạo Stored Procedure
CREATE PROCEDURE dbo.sp_BaoCaoChiTietThietBi
    @TuKhoaTen NVARCHAR(100) = N'' -- Mặc định là rỗng để hiện tất cả
AS
BEGIN
    -- Lệnh SELECT join nhiều bảng để trả về tập kết quả (Result Set)
    SELECT 
        T.MaTB AS [Mã Thiết Bị],
        T.TenTB AS [Tên Thiết Bị],
        L.TenLoai AS [Danh Mục Loại],
        T.TinhTrang AS [Trạng Thái Hiện Tại]
    FROM ThietBi T
    INNER JOIN LoaiThietBi L ON T.MaLoai = L.MaLoai
    WHERE T.TenTB LIKE '%' + @TuKhoaTen + '%'
    ORDER BY L.TenLoai ASC;
END;
GO

-- 3. Thực thi thủ tục (Chụp ảnh màn hình kết quả ở bước này)
-- Trường hợp 1: Lấy toàn bộ danh sách
EXEC dbo.sp_BaoCaoChiTietThietBi;

-- Trường hợp 2: Lọc các thiết bị có tên chứa chữ 'Dell'
EXEC dbo.sp_BaoCaoChiTietThietBi @TuKhoaTen = N'Dell';
GO
```
<img width="1912" height="967" alt="image" src="https://github.com/user-attachments/assets/42d83e2a-61a9-4e24-ab6c-9dae35b5500b" />

# Phần 4: Trigger và Xử lý logic nghiệp vụ

* Yêu cầu: Khi một thiết bị bị xóa khỏi bảng ThietBi, hệ thống phải tự động ghi chép lại thông tin thiết bị đó vào một bảng nhật ký tên là NhatKyXoa_VanAnh.

Mục đích: Đảm bảo tính minh bạch. Nếu ai đó lỡ tay xóa nhầm, quản trị viên vẫn biết thiết bị nào đã bị xóa và xóa vào lúc nào.

Cơ chế: Trigger sẽ tự động kích hoạt ngay sau khi lệnh DELETE trên bảng ThietBi hoàn tất.
```
USE QuanLyLab_VanAnh_003;
GO

-- Bước 1: Tạo bảng B (Bảng Nhật ký) để chứa dữ liệu bị xóa
IF OBJECT_ID('NhatKyXoa_VanAnh', 'U') IS NULL
BEGIN
    CREATE TABLE NhatKyXoa_VanAnh (
        ID_Log INT PRIMARY KEY IDENTITY(1,1),
        MaTB_DaXoa VARCHAR(10),
        TenTB_DaXoa NVARCHAR(100),
        NgayXoa DATETIME DEFAULT GETDATE(),
        NguoiXoa NVARCHAR(100)
    );
END
GO

-- Bước 2: Tạo Trigger trên bảng A (ThietBi)
CREATE TRIGGER trg_LuuNhatKyXoa
ON ThietBi
AFTER DELETE
AS
BEGIN
    -- Chèn dữ liệu từ bảng tạm 'deleted' (chứa các dòng vừa bị xóa) vào bảng Nhật ký
    INSERT INTO NhatKyXoa_VanAnh (MaTB_DaXoa, TenTB_DaXoa, NguoiXoa)
    SELECT MaTB, TenTB, SYSTEM_USER
    FROM deleted;
    
    PRINT N'Trigger của Vân Anh: Đã tự động sao lưu thông tin thiết bị vào Nhật ký xóa.';
END;
GO

-- Bước 3: Kiểm tra Trigger (Chụp ảnh kết quả ở đây)
-- Xóa thử một thiết bị (Giả sử mã TB01 đã tạo ở các bài trước)
DELETE FROM ThietBi WHERE MaTB = 'TB01';

-- Xem bảng Nhật ký để chứng minh Trigger đã chạy
SELECT * FROM NhatKyXoa_VanAnh;
```
<img width="1908" height="971" alt="image" src="https://github.com/user-attachments/assets/e15c2150-e49e-4b6a-816c-36a656ef5b5f" />
Đã hoàn thành

Khi cập nhật trạng thái ở bảng Thiết bị (A) thì tự động cập nhật bảng Nhật ký (B), và ngược lại.
```
USE QuanLyLab_VanAnh_003;
GO

-- 1. Tạo Trigger cho Bảng A (ThietBi): Khi Update thì cập nhật sang Bảng B
CREATE TRIGGER trg_A_to_B
ON ThietBi
AFTER UPDATE
AS
BEGIN
    PRINT N'Trigger A đang chạy: Cập nhật sang bảng B...';
    UPDATE NhatKyXoa_VanAnh SET GhiChu = N'Thay đổi từ A' 
    WHERE MaTB_DaXoa IN (SELECT MaTB FROM inserted);
END;
GO

-- 2. Tạo Trigger cho Bảng B (NhatKyXoa_VanAnh): Khi Update thì cập nhật ngược lại A
CREATE TRIGGER trg_B_to_A
ON NhatKyXoa_VanAnh
AFTER UPDATE
AS
BEGIN
    PRINT N'Trigger B đang chạy: Cập nhật ngược lại bảng A...';
    UPDATE ThietBi SET TinhTrang = N'Thay đổi từ B' 
    WHERE MaTB IN (SELECT MaTB_DaXoa FROM inserted);
END;
GO

-- 3. CHẠY LỆNH CẬP NHẬT ĐỂ QUAN SÁT LỖI
UPDATE ThietBi SET TinhTrang = N'Đang test' WHERE MaTB = 'TB01';
GO
```
<img width="1911" height="973" alt="image" src="https://github.com/user-attachments/assets/90c997e6-e337-4753-a437-aa19b053d727" />
Đây là quá trình này tạo thành một vòng lặp vô tận

Nguyên nhân: Khi bảng A cập nhật -> Kích hoạt Trigger A -> Trigger A thực hiện lệnh Update trên bảng B. Khi bảng B vừa bị Update -> Kích hoạt Trigger B -> Trigger B thực hiện lệnh Update ngược lại bảng A

Phản ứng của SQL Server: Để bảo vệ hệ thống không bị treo hoặc tràn bộ nhớ, SQL Server có một "chốt an toàn". Nó cho phép các đối tượng gọi nhau tối đa 32 cấp. Khi vượt quá con số này, hệ thống sẽ tự động ngắt lệnh, hủy bỏ mọi thay đổi (Rollback) và thông báo lỗi "Nesting level exceeded".

Tính rủi ro cao: Việc thiết kế Trigger vòng lặp là một sai lầm nghiêm trọng trong thiết kế cơ sở dữ liệu. Nó gây lãng phí tài nguyên hệ thống và làm treo các giao dịch (Transactions).

Sự bảo vệ của DBMS: Rất may là các hệ quản trị CSDL như SQL Server có cơ chế kiểm soát mức độ lồng nhau (nesting level) để ngăn chặn thảm họa treo máy.

Giải pháp thực tế: * Nên hạn chế dùng Trigger để cập nhật qua lại giữa các bảng có quan hệ trực tiếp.

Nếu bắt buộc phải dùng, cần sử dụng hàm IF TRIGGER_NESTLEVEL() < 2 để chặn Trigger không được chạy quá một cấp.

Thay vì dùng Trigger, nên xử lý logic này ở tầng Stored Procedure để kiểm soát luồng dữ liệu tốt hơn.

# Phần 5: Cursor và Duyệt dữ liệu

* Yêu cầu: Giả sử cuối năm, sinh viên Phạm Thị Vân Anh cần làm một báo cáo tổng kết tình trạng thiết bị dưới dạng một chuỗi văn bản dài để gửi qua email hoặc in nhãn dán.

Chúng ta cần duyệt qua từng thiết bị trong bảng ThietBi.

Với mỗi dòng, ta sẽ ghép nối thông tin thành một câu mô tả hoàn chỉnh: "Thiết bị [Tên] (Mã: [Mã]) hiện đang có trạng thái: [Trạng thái]".

In kết quả đó ra màn hình hoặc lưu vào một biến tổng.
```
USE QuanLyLab_VanAnh_003;
GO

-- 1. Khai báo các biến để chứa dữ liệu từng dòng khi duyệt
DECLARE @MaCur VARCHAR(10);
DECLARE @TenCur NVARCHAR(100);
DECLARE @TrangThaiCur NVARCHAR(50);
DECLARE @ThongBao NVARCHAR(MAX);

PRINT '---------- BAO CAO CHI TIET THIET BI (BY VAN ANH) ----------';

-- 2. Khai báo CURSOR để duyệt qua câu lệnh SELECT
DECLARE cur_ThietBi CURSOR FOR 
    SELECT MaTB, TenTB, TinhTrang FROM ThietBi;

-- 3. Mở CURSOR
OPEN cur_ThietBi;

-- 4. Lấy dòng dữ liệu đầu tiên
FETCH NEXT FROM cur_ThietBi INTO @MaCur, @TenCur, @TrangThaiCur;

-- 5. Vòng lặp duyệt cho đến khi hết dữ liệu (@@FETCH_STATUS = 0)
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Xử lý logic riêng cho từng bản ghi: Ghép chuỗi thông báo
    SET @ThongBao = N'>>> Thiet bi: ' + @TenCur + 
                    N' (Ma: ' + @MaCur + 
                    N') - Tinh trang: ' + @TrangThaiCur;
    
    -- In kết quả xử lý của dòng hiện tại
    PRINT @ThongBao;

    -- Lấy dòng tiếp theo
    FETCH NEXT FROM cur_ThietBi INTO @MaCur, @TenCur, @TrangThaiCur;
END;

-- 6. Đóng và Giải phóng CURSOR (Rất quan trọng để giải phóng bộ nhớ)
CLOSE cur_ThietBi;
DEALLOCATE cur_ThietBi;

PRINT '-------------------- KET THUC BAO CAO --------------------';
GO
```
<img width="1909" height="965" alt="image" src="https://github.com/user-attachments/assets/b33b1e52-616e-4283-9871-7f987ce1a08e" />
Minh họa cách sử dụng CURSOR

Giải pháp thay thế CURSOR: Tư duy "Xử lý tập hợp" (Set-based)Trong SQL, thay vì "nhặt" từng dòng ra để xử lý (như Cursor), ta nên dùng các hàm xử lý chuỗi trực tiếp trên toàn bộ cột của bảng.Cách 1: Sử dụng câu lệnh SELECT thuần túy (Nhanh nhất)Đây là cách tối ưu nhất để lấy ra danh sách báo cáo. SQL Server sẽ quét một lượt qua bảng và trả về toàn bộ kết quả.Cách 2: Sử dụng biến cộng dồn chuỗi (Nếu muốn gộp thành 1 đoạn văn duy nhất)Nếu em muốn tất cả thông báo được gộp lại thành một khối văn bản lớn (giống như việc Cursor in từng dòng), em có thể dùng kỹ thuật sau:
```
USE QuanLyLab_VanAnh_003;
GO

DECLARE @BaoCaoTongHop NVARCHAR(MAX) = '';

-- Dùng SELECT để cộng dồn dữ liệu vào biến
SELECT @BaoCaoTongHop = @BaoCaoTongHop + 
    N'>>> Thiet bi: ' + TenTB + N' (Ma: ' + MaTB + N') - Tinh trang: ' + TinhTrang + CHAR(13)
FROM ThietBi;

PRINT @BaoCaoTongHop;
<img width="1905" height="971" alt="image" src="https://github.com/user-attachments/assets/d25999c0-a403-4d83-97be-cc6725673189" />
Kết quả
So sánh chi tiết: CURSOR vs. SET-BASEDDưới đây là bảng so sánh để em đưa vào bài báo cáo nhằm tăng tính chuyên môn:Tiêu chíSử dụng CURSOR (Row-by-Row)Sử dụng SELECT (Set-based)Cách thức hoạt độngMở một con trỏ, duyệt dòng 1, xử lý, duyệt dòng 2...SQL Optimizer tính toán đường đi ngắn nhất để xử lý toàn bộ bảng một lúc.Tốc độ (Latency)Chậm. Mỗi bước FETCH là một lần yêu cầu tài nguyên hệ thống.Cực nhanh. Tận dụng tối đa bộ nhớ đệm (Cache) và lập chỉ mục (Index).Tốn tài nguyênTốn RAM để duy trì trạng thái con trỏ và khóa dữ liệu (Locks).Tối ưu hóa CPU, giải phóng tài nguyên ngay sau khi lệnh kết thúc.Độ dài mã nguồnDài (Declare, Open, Fetch, While, Close, Deallocate).Ngắn gọn (Thường chỉ là 1 câu lệnh SELECT).3. Mã SQL đo đạc thực tế (Minh chứng ảnh chụp)Em hãy copy đoạn mã này, chạy nó và chụp ảnh cả tab Results và tab Messages.
USE QuanLyLab_VanAnh_003;
GO

-- Bật đo lường thời gian và đọc dữ liệu (IO)
SET STATISTICS TIME ON;
SET STATISTICS IO ON;
GO

PRINT '--------------------------------------------------';
PRINT 'TEST 1: SU DUNG CURSOR';
DECLARE @m VARCHAR(10), @t NVARCHAR(100), @tt NVARCHAR(50);
DECLARE test_cursor CURSOR FOR SELECT MaTB, TenTB, TinhTrang FROM ThietBi;
OPEN test_cursor;
FETCH NEXT FROM test_cursor INTO @m, @t, @tt;
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Xử lý giả lập
    FETCH NEXT FROM test_cursor INTO @m, @t, @tt;
END;
CLOSE test_cursor; DEALLOCATE test_cursor;
GO

PRINT '--------------------------------------------------';
PRINT 'TEST 2: SU DUNG SELECT (KHONG CURSOR)';
SELECT N'>>> Thiet bi: ' + TenTB + N' (Ma: ' + MaTB + N') - Tinh trang: ' + TinhTrang 
FROM ThietBi;
GO

-- Tắt đo lường
SET STATISTICS TIME OFF;
SET STATISTICS IO OFF;
GO
```
<img width="1918" height="961" alt="image" src="https://github.com/user-attachments/assets/12a4eeef-d0e2-4985-9d47-123178a4b734" />
Kết quả

Giải thích kết quả
CPU time: Thời gian mà bộ vi xử lý dành ra để tính toán. Ở cách dùng SELECT, con số này thường là 0ms. Ở cách dùng Cursor, con số này sẽ lớn hơn 0 vì CPU phải quản lý vòng lặp.

Elapsed time: Tổng thời gian từ lúc nhấn nút chạy đến khi xong. Em sẽ thấy SELECT hoàn thành nhanh hơn hẳn.

Logical reads: (Trong phần IO) Số lần SQL phải đọc trang dữ liệu. Cursor thường có số lần đọc cao hơn vì mỗi lệnh FETCH có thể gây ra các thao tác thừa.

Kết luận: Qua thực nghiệm trên cơ sở dữ liệu quản lý Lab, em nhận thấy phương pháp xử lý tập hợp (Set-based) bằng câu lệnh SELECT đơn giản mang lại hiệu suất vượt trội. Việc không sử dụng Cursor không chỉ giúp mã nguồn ngắn gọn, dễ bảo trì mà còn giảm tải cho hệ thống, đặc biệt là khi số lượng thiết bị trong Lab tăng lên hàng nghìn hoặc hàng triệu bản ghi. Cursor chỉ nên là lựa chọn cuối cùng khi các phương pháp Set-based không thể đáp ứng được logic nghiệp vụ phức tạp

* Yêu cầu: Duyệt danh sách đăng ký theo thời gian. Nếu số lượng trong kho còn đủ thì phê duyệt (Approved) và trừ vào kho. Nếu không đủ thì từ chối (Rejected).
```
USE QuanLyLab_VanAnh_003;
GO

-- Bước A: Tạo bảng PhieuMuon và nạp dữ liệu mẫu
IF OBJECT_ID('PhieuMuon', 'U') IS NOT NULL DROP TABLE PhieuMuon;
CREATE TABLE PhieuMuon (
    MaPhieu INT PRIMARY KEY,
    TenSV NVARCHAR(50),
    SoLuongMuon INT,
    NgayDangKy DATETIME,
    TrangThai NVARCHAR(50) DEFAULT N'Chờ duyệt'
);

INSERT INTO PhieuMuon (MaPhieu, TenSV, SoLuongMuon, NgayDangKy) VALUES 
(1, N'Nguyễn Văn A', 5, '2024-05-01 08:00'),
(2, N'Trần Thị B', 4, '2024-05-01 08:30'),
(3, N'Lê Văn C', 3, '2024-05-01 09:00'), -- Tổng đến đây là 12, sẽ bị thiếu hàng
(4, N'Phạm Văn D', 1, '2024-05-01 09:30');
GO

-- Bước B: Chạy Cursor xử lý tuần tự từng phiếu
DECLARE @TonKho INT = 10; -- Giả sử kho có 10 máy
DECLARE @MaP INT, @Sl INT, @Ten NVARCHAR(50);

DECLARE cur_Kho CURSOR FOR 
    SELECT MaPhieu, TenSV, SoLuongMuon FROM PhieuMuon ORDER BY NgayDangKy ASC;

OPEN cur_Kho;
FETCH NEXT FROM cur_Kho INTO @MaP, @Ten, @Sl;

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @TonKho >= @Sl
    BEGIN
        UPDATE PhieuMuon SET TrangThai = N'Đã duyệt' WHERE MaPhieu = @MaP;
        SET @TonKho = @TonKho - @Sl; -- Cập nhật số dư tức thời để xét cho dòng sau
        PRINT N'>>> Duyệt cho ' + @Ten + N' (' + CAST(@Sl AS NVARCHAR) + N' máy). Kho còn: ' + CAST(@TonKho AS NVARCHAR);
    END
    ELSE
    BEGIN
        UPDATE PhieuMuon SET TrangThai = N'Từ chối (Hết hàng)' WHERE MaPhieu = @MaP;
        PRINT N'>>> Từ chối ' + @Ten + N' (Cần ' + CAST(@Sl AS NVARCHAR) + N' máy). Kho chỉ còn: ' + CAST(@TonKho AS NVARCHAR);
    END
    FETCH NEXT FROM cur_Kho INTO @MaP, @Ten, @Sl;
END

CLOSE cur_Kho; DEALLOCATE cur_Kho;
GO

-- Bước C: Xem kết quả cuối cùng
SELECT * FROM PhieuMuon;
```
<img width="1907" height="989" alt="image" src="https://github.com/user-attachments/assets/6fe53270-0e2c-41e4-ad74-4d41b0e3c676" />
Kết quả

Tại sao SQL thuần (Set-based) lại cực kỳ khó giải quyết bài này?
Tính phụ thuộc của dữ liệu (Interdependency): Quyết định của dòng thứ 3 (Lê Văn C) phụ thuộc trực tiếp vào việc dòng 1 và 2 có được duyệt hay không. SQL thuần nhìn dữ liệu theo "cả mớ", nó không thể vừa tính toán, vừa cập nhật rồi lấy kết quả đó để tính tiếp cho dòng sau trong cùng một câu lệnh đơn giản.

Hành động thay đổi (Update) đi kèm điều kiện: SQL thuần có thể tính được tổng lũy kế (Running Total), nhưng để thực hiện lệnh UPDATE trạng thái khác nhau (Đã duyệt vs Từ chối) cho từng dòng dựa trên số dư thay đổi liên tục thì SQL thuần cần những kỹ thuật cực kỳ phức tạp (như đệ quy CTE) và rất khó kiểm soát lỗi.

Tư duy lập trình: Cursor cho phép chúng ta chèn các câu lệnh IF...ELSE, PRINT, hoặc thậm chí gọi các Stored Procedure khác bên trong mỗi dòng. SQL thuần không làm được điều này.

Kết luận:Qua bài thực hành này, em nhận thấy Cursor không phải lúc nào cũng xấu. Trong bài toán quản lý kho và phê duyệt đăng ký, Cursor giúp đảm bảo tính công bằng (ai đến trước được trước) và kiểm soát số dư tồn kho tức thời một cách chính xác nhất. Đây là trường hợp mà sự chính xác của nghiệp vụ quan trọng hơn tốc độ xử lý thuần túy của hệ thống.
