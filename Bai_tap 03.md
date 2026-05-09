Họ Tên : Phạm Thị Vân Anh

MSSV: K235480106003

Lớp: K59KMT.K01

Nội Dung: THIẾT KẾ VÀ CÀI ĐẶT CSDL QUẢN LÝ CẦM ĐỒ

# Nhiệm vụ 1: Thiết kế CSDL

Phần Code

```
-- 1. Tạo Database mới
CREATE DATABASE QuanLyCamDo;
GO
USE QuanLyCamDo;
GO

-- 2. Bảng Khách hàng (KhachHang)
CREATE TABLE KhachHang (
    MaKH INT PRIMARY KEY IDENTITY(1,1), -- Khóa chính tự tăng
    HoTen NVARCHAR(100) NOT NULL,
    SoDienThoai VARCHAR(15),
    CCCD VARCHAR(12) UNIQUE,
    DiaChi NVARCHAR(255)
);

-- 3. Bảng Nhân viên (NhanVien) - Để ghi nhận người thu tiền trong Log
CREATE TABLE NhanVien (
    MaNV INT PRIMARY KEY IDENTITY(1,1),
    HoTen NVARCHAR(100) NOT NULL,
    ChucVu NVARCHAR(50)
);

-- 4. Bảng Hợp đồng (HopDong)
CREATE TABLE HopDong (
    MaHD INT PRIMARY KEY IDENTITY(1,1),
    MaKH INT NOT NULL, -- Khóa ngoại nối tới KhachHang
    NgayVay DATETIME DEFAULT GETDATE(),
    SoTienGoc DECIMAL(18,2) NOT NULL,
    Deadline1 DATE NOT NULL, -- Mốc tính lãi kép
    Deadline2 DATE NOT NULL, -- Mốc thanh lý tài sản
    TrangThai NVARCHAR(50) DEFAULT N'Đang vay',
    CONSTRAINT FK_HopDong_KhachHang FOREIGN KEY (MaKH) REFERENCES KhachHang(MaKH)
);

-- 5. Bảng Tài sản (TaiSan)
CREATE TABLE TaiSan (
    MaTS INT PRIMARY KEY IDENTITY(1,1),
    MaHD INT NOT NULL, -- Khóa ngoại nối tới HopDong
    TenTaiSan NVARCHAR(100) NOT NULL,
    GiaTriDinhGia DECIMAL(18,2),
    TrangThaiTS NVARCHAR(50) DEFAULT N'Đang cầm cố',
    CONSTRAINT FK_TaiSan_HopDong FOREIGN KEY (MaHD) REFERENCES HopDong(MaHD)
);

-- 6. Bảng Nhật ký (LogBienDong)
CREATE TABLE LogBienDong (
    MaLog INT PRIMARY KEY IDENTITY(1,1),
    MaHD INT NOT NULL, -- Khóa ngoại nối tới HopDong
    MaNV INT NOT NULL, -- Khóa ngoại nối tới NhanVien
    NgayGiaoDich DATETIME DEFAULT GETDATE(),
    SoTienTra DECIMAL(18,2),
    NoiDung NVARCHAR(255),
    CONSTRAINT FK_Log_HopDong FOREIGN KEY (MaHD) REFERENCES HopDong(MaHD),
    CONSTRAINT FK_Log_NhanVien FOREIGN KEY (MaNV) REFERENCES NhanVien(MaNV)
);
```

<img width="1915" height="972" alt="image" src="https://github.com/user-attachments/assets/d5da626f-d246-43b0-aa8b-f0e46be584ed" />

<img width="1906" height="975" alt="image" src="https://github.com/user-attachments/assets/648b8690-98e8-4b5c-838e-5dffd2393d6c" />

Để thực hiện Nhiệm vụ 1, em đã khởi tạo cơ sở dữ liệu QuanLyCamDo và thiết lập 5 bảng dữ liệu chuẩn hóa 3NF gồm: Khách hàng, Nhân viên, Hợp đồng, Tài sản và Log biến động. Các bảng được kết nối chặt chẽ bằng khóa ngoại để quản lý chi tiết từ thông tin người vay đến lịch sử dòng tiền và trạng thái tài sản thế chấp theo đúng yêu cầu nghiệp vụ.

<img width="1905" height="973" alt="image" src="https://github.com/user-attachments/assets/d0cb90b2-9dd3-4928-9e3a-bc761cf26dd6" />
Sau khi khởi tạo cơ sở dữ liệu QuanLyCamDo, em đã thiết lập 5 bảng chuẩn hóa 3NF và kết nối chúng qua sơ đồ Diagram. Cấu trúc này giúp quản lý chặt chẽ mối quan hệ giữa khách hàng, nhiều tài sản thế chấp và lưu vết toàn bộ lịch sử giao dịch theo thời gian.

# Nhiệm vụ 2: Cài đặt SQL

* Event 1: Đăng ký hợp đồng mới (Vay tiền)

Phần code
```
CREATE PROCEDURE sp_DangKyHopDong
    @HoTen NVARCHAR(100), @SDT VARCHAR(15), @CCCD VARCHAR(12),
    @SoTienVay DECIMAL(18,2), @DL1 DATE, @DL2 DATE,
    @TenTS NVARCHAR(100), @DinhGia DECIMAL(18,2)
AS
BEGIN
    BEGIN TRANSACTION;
    BEGIN TRY
        -- 1. Xử lý thông tin khách hàng
        DECLARE @MaKH INT;
        SELECT @MaKH = MaKH FROM KhachHang WHERE CCCD = @CCCD;
        IF @MaKH IS NULL
        BEGIN
            INSERT INTO KhachHang (HoTen, SoDienThoai, CCCD) VALUES (@HoTen, @SDT, @CCCD);
            SET @MaKH = SCOPE_IDENTITY();
        END

        -- 2. Tạo hợp đồng mới
        INSERT INTO HopDong (MaKH, SoTienGoc, Deadline1, Deadline2, TrangThai)
        VALUES (@MaKH, @SoTienVay, @DL1, @DL2, N'Đang vay');
        DECLARE @MaHD INT = SCOPE_IDENTITY();

        -- 3. Lưu thông tin tài sản cầm cố
        INSERT INTO TaiSan (MaHD, TenTaiSan, GiaTriDinhGia, TrangThaiTS)
        VALUES (@MaHD, @TenTS, @DinhGia, N'Đang cầm cố');

        COMMIT;
    END TRY
    BEGIN CATCH
        ROLLBACK;
        THROW;
    END CATCH
END;
```
<img width="1914" height="974" alt="image" src="https://github.com/user-attachments/assets/e25f9220-18b6-4c89-b503-b6272a9ebef9" />
Đảm bảo tính toàn vẹn dữ liệu ngay từ đầu. Việc thiết lập đồng thời mã khách hàng, số tiền gốc và hai mốc Deadline (D1, D2) giúp hệ thống có cơ sở dữ liệu chuẩn để tự động hóa các bước tính lãi và thanh lý sau này.

* Event 2: Tính toán công nợ thời gian thực

Phần code
```
CREATE FUNCTION fn_CalcMoneyContract (@MaHD INT, @TargetDate DATETIME)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @Goc DECIMAL(18,2), @NgayVay DATETIME, @DL1 DATE;
    DECLARE @TongNo DECIMAL(18,2);
    DECLARE @r FLOAT = 0.005; 

    SELECT @Goc = SoTienGoc, @NgayVay = NgayVay, @DL1 = Deadline1 
    FROM HopDong WHERE MaHD = @MaHD;

    IF @TargetDate <= @DL1
    BEGIN
        DECLARE @t1 INT = DATEDIFF(DAY, @NgayVay, @TargetDate);
        SET @TongNo = @Goc * (1 + @r * (CASE WHEN @t1 < 0 THEN 0 ELSE @t1 END));
    END
    ELSE
    BEGIN
        DECLARE @t_don INT = DATEDIFF(DAY, @NgayVay, @DL1);
        DECLARE @P_at_DL1 DECIMAL(18,2) = @Goc * (1 + @r * @t_don);
        DECLARE @n INT = DATEDIFF(DAY, @DL1, @TargetDate);
        SET @TongNo = @P_at_DL1 * CAST(POWER(1 + @r, @n) AS DECIMAL(18,2));
    END
    RETURN @TongNo;
END;
```
<img width="1914" height="979" alt="image" src="https://github.com/user-attachments/assets/c207a8c0-545b-44ea-a36a-461a18bfd99b" />
Xử lý chính xác mô hình lãi chồng lãi. Hàm (Function) có khả năng phân tách rạch ròi giữa lãi đơn (trước D1) và lãi kép (sau D1), cho phép truy xuất số nợ tại bất kỳ thời điểm nào (TargetDate), hỗ trợ tốt cho việc tư vấn khách hàng.

* Event 3: Xử lý trả nợ và hoàn trả tài sản

Phần code
```
CREATE PROCEDURE sp_XuLyTraNo
    @MaHD INT, @MaNV INT, @SoTienTra DECIMAL(18,2)
AS
BEGIN
    DECLARE @TongNo DECIMAL(18,2) = dbo.fn_CalcMoneyContract(@MaHD, GETDATE());
    IF EXISTS (SELECT 1 FROM HopDong WHERE MaHD = @MaHD AND TrangThai = N'Đã thanh lý')
    BEGIN
        PRINT N'Tài sản đã bán thanh lý, không thu tiền.';
        RETURN;
    END

    INSERT INTO LogBienDong (MaHD, MaNV, SoTienTra, NgayGiaoDich, NoiDung)
    VALUES (@MaHD, @MaNV, @SoTienTra, GETDATE(), N'Khách trả nợ');

    IF @SoTienTra >= @TongNo
        UPDATE HopDong SET TrangThai = N'Đã thanh toán đủ' WHERE MaHD = @MaHD;
    ELSE
        UPDATE HopDong SET TrangThai = N'Đang trả góp' WHERE MaHD = @MaHD;

    SELECT MaTS, TenTaiSan FROM TaiSan 
    WHERE MaHD = @MaHD AND GiaTriDinhGia >= (@TongNo - @SoTienTra);
END;
```
<img width="1915" height="984" alt="image" src="https://github.com/user-attachments/assets/fb6f279f-f3a1-41b5-bdc1-d1167fe9e089" />
Tối ưu hóa quản trị rủi ro. Quy trình tự động chặn thu tiền nếu đồ đã bán (IsSold) và sử dụng thuật toán so sánh giá trị tài sản với dư nợ để quyết định trả đồ, đảm bảo cửa hàng không bao giờ bị "hớ" khi khách trả nợ từng phần.

* Event 4: Truy vấn danh sách nợ xấu (Nợ khó đòi)

Phần code
```
SELECT k.HoTen, k.SoDienThoai, h.SoTienGoc, 
       DATEDIFF(DAY, h.Deadline1, GETDATE()) AS SoNgayQuaHan,
       dbo.fn_CalcMoneyContract(h.MaHD, GETDATE()) AS TongNoHien Tai,
       dbo.fn_CalcMoneyContract(h.MaHD, DATEADD(MONTH, 1, GETDATE())) AS NoDuKienSau1Thang
FROM HopDong h JOIN KhachHang k ON h.MaKH = k.MaKH
WHERE GETDATE() > h.Deadline1 AND h.TrangThai <> N'Đã thanh toán đủ';
```
<img width="1914" height="971" alt="image" src="https://github.com/user-attachments/assets/e272c942-fb14-4085-805e-3bb0a4dc92d1" />
Công cụ dự báo tài chính hiệu quả. Việc hiển thị đồng thời số nợ hiện tại và dự báo nợ sau 1 tháng giúp bộ phận thu hồi nợ có số liệu thuyết phục để hối thúc khách hàng thanh toán sớm nhằm tránh phát sinh lãi kép khổng lồ.

* Event 5: Quản lý thanh lý tài sản

Phần code
```
CREATE TRIGGER trg_AutoStatus ON HopDong AFTER UPDATE AS
BEGIN
    UPDATE HopDong SET TrangThai = N'Quá hạn (nợ xấu)' 
    FROM HopDong h JOIN inserted i ON h.MaHD = i.MaHD 
    WHERE h.TrangThai = N'Đang vay' AND GETDATE() > h.Deadline1;

    UPDATE TaiSan SET TrangThaiTS = N'Sẵn sàng thanh lý' 
    FROM TaiSan t JOIN HopDong h ON t.MaHD = h.MaHD 
    WHERE h.TrangThai = N'Quá hạn (nợ xấu)' AND GETDATE() > h.Deadline2;
END;
```
<img width="1910" height="986" alt="image" src="https://github.com/user-attachments/assets/181602aa-6456-42e5-9c58-bed74ec0b755" />
Tự động hóa vận hành kho bãi. Các Trigger giúp trạng thái hợp đồng và tài sản luôn khớp với thực tế thời gian (D1, D2) mà không cần thao tác thủ công, giảm thiểu sai sót con người và đảm bảo tính minh bạch pháp lý khi chuyển đồ sang diện thanh lý.

# 4. Các sự kiện bổ sung

Sự kiện Gia hạn hợp đồng: Khách đến trả toàn bộ tiền lãi tính đến thời điểm hiện tại để dời 
Deadline 1 và Deadline 2 sang một kỳ hạn mới để tránh bị tính lãi kép.

Phần code
```
-- A. Store Procedure Gia hạn hợp đồng cho khách (Ví dụ: Nguyễn Văn A)
CREATE PROCEDURE sp_GiaHanHopDong
    @MaHD INT,
    @MaNV INT,
    @SoThangGiaHan INT
AS
BEGIN
    -- Tính toán số lãi khách phải trả hết trước khi gia hạn
    DECLARE @TongNo DECIMAL(18,2) = dbo.fn_CalcMoneyContract(@MaHD, GETDATE());
    DECLARE @Goc DECIMAL(18,2);
    SELECT @Goc = SoTienGoc FROM HopDong WHERE MaHD = @MaHD;
    DECLARE @LaiHienTai DECIMAL(18,2) = @TongNo - @Goc;

    -- 1. Ghi nhận việc trả lãi vào Audit Log
    INSERT INTO LogBienDong (MaHD, MaNV, SoTienTra, NoiDung)
    VALUES (@MaHD, @MaNV, @LaiHienTai, N'Khách Nguyễn Văn A trả lãi để gia hạn thêm ' + CAST(@SoThangGiaHan AS NVARCHAR(5)) + N' tháng');

    -- 2. Cập nhật dời Deadline 1 và Deadline 2 sang kỳ hạn mới
    UPDATE HopDong
    SET Deadline1 = DATEADD(MONTH, @SoThangGiaHan, Deadline1),
        Deadline2 = DATEADD(MONTH, @SoThangGiaHan, Deadline2),
        TrangThai = N'Đang vay'
    WHERE MaHD = @MaHD;
END;
GO

-- B. Truy vấn lịch sử hợp đồng (Audit Log) để chứng thực dòng tiền
SELECT 
    L.NgayGiaoDich, 
    N.HoTen AS NhanVienThuTien, 
    L.SoTienTra, 
    L.NoiDung
FROM LogBienDong L
JOIN NhanVien N ON L.MaNV = N.MaNV
WHERE L.MaHD = 1 -- Giả sử kiểm tra lịch sử của Nguyễn Văn A (ID=1)
ORDER BY L.NgayGiaoDich DESC;
```
<img width="1914" height="969" alt="image" src="https://github.com/user-attachments/assets/8cc42fe6-c859-4d90-bdbb-89071cfa6a16" />
Đây là giải pháp "giải cứu" tài chính cho khách hàng bằng cách tất toán phần lãi đã phát sinh để làm mới (reset) chu kỳ vay

```
-- TRUY VẤN LỊCH SỬ GIAO DỊCH (AUDIT LOG)
SELECT 
    L.NgayGiaoDich AS [Ngày Trả], 
    N.HoTen AS [Nhân Viên Thu], 
    L.SoTienTra AS [Số Tiền Trả], 
    L.NoiDung AS [Ghi Chú Chi Tiết]
FROM LogBienDong L
JOIN NhanVien N ON L.MaNV = N.MaNV
JOIN HopDong H ON L.MaHD = H.MaHD
JOIN KhachHang K ON H.MaKH = K.MaKH
WHERE K.HoTen = N'Nguyễn Văn A' -- Kiểm tra lịch sử riêng của Nguyễn Văn A
ORDER BY L.NgayGiaoDich DESC;
```
<img width="1912" height="973" alt="image" src="https://github.com/user-attachments/assets/faad881f-4549-46fb-a695-188aeb214b29" />
Em đã triển khai cơ chế Audit Log thông qua bảng LogBienDong để ghi lại chi tiết từng lần thanh toán của khách hàng
