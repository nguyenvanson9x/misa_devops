### **KẾ HOẠCH TÁCH VÀ CHUẨN HÓA CƠ SỞ DỮ LIỆU BUSINESS**

#### **I. TỔNG QUAN HIỆN TRẠNG VÀ MỤC TIÊU**

| Trạng thái | Tên Database | Vị trí Máy chủ | Ghi chú / Ràng buộc |
| :--- | :--- | :--- | :--- |
| **Hiện tại** | `Login` | `MISAID MSSQL 04` | Đang chứa một phần dữ liệu Business. |
| **Hiện tại** | `Token` | `MSSQL 02` | Dung lượng \~200GB (Phần lớn do bảng `PersistedGrant`). |
| **Mục tiêu** | **`MISA.PersonalCustomerProfile.Business`** (Tạo mới) | **`MISAID MSSQL 05`** | Cụm máy chủ đích có **giới hạn Disk là 100GB**. |

-----

#### **II. CHI TIẾT CÁC BƯỚC THỰC THI**

| Bước | Hạng mục công việc | Môi trường / Máy chủ | Tài nguyên / Ghi chú |
| :--- | :--- | :--- | :--- |
| **1.1** | Backup database `Token` gốc. | `MSSQL 02` | Sinh ra file backup \~200GB. |
| **1.2** | Restore DB `Token` vào máy chủ trung gian. | `MISA CNTT MSSQL 01` | Cần dung lượng trống đủ lớn để chứa DB phục hồi. |
| **1.3** | Xóa dữ liệu bảng `PersistedGrant` để giảm tải. | `MISA CNTT MSSQL 01` | Dùng `TRUNCATE`/`DELETE`. *Lưu ý: Nên chạy thêm `SHRINKDATABASE` để giảm dung lượng file vật lý.* |
| **1.4** | Backup lại DB đã được làm sạch và thu gọn. | `MISA CNTT MSSQL 01` | Bản backup này phải có dung lượng an toàn (\< 100GB). |
| **1.5** | Restore vào máy chủ đích và đổi tên thành: <br>**`MISA.PersonalCustomerProfile.Business`** | `MISAID MSSQL 05` | Hoàn tất khởi tạo DB Business mới. |
| **2** | **Migrate dữ liệu từ DB Login sang DB Business:**<br>Chuyển các bảng: `Totp`, `MigrateAccountLog`, `LinkActive`, `PasswordRegular`, `LoginMethod`, `UserTwofactorMethodLogs`, `ToolAdminActionLogs`, `UserInfo`. | Nguồn: `MISAID MSSQL 04`<br>Đích: `MISAID MSSQL 05` | Đảm bảo tính toàn vẹn dữ liệu giữa hai hệ thống. |
| **3** | Chạy Script chuẩn hóa kiểu dữ liệu cho DB Business. | `MISAID MSSQL 05` | Script: `Personal_Customer_Profile_Business.sql` |
| **4** | Chạy Script chuẩn hóa kiểu dữ liệu cho DB Token gốc. | `MSSQL 02` | Script: `Personal_Customer_Profile_Token.sql` |
| **5** | Cập nhật cấu hình Ứng dụng (Connection String). | Hệ thống Application / Services | Đổi chuỗi kết nối đọc/ghi dữ liệu Business sang máy chủ `MSSQL 05` và test lại luồng. |
