# Kế hoạch Tách Database SQL Server

## A. Hiện Trạng

| Thành phần | Chi tiết |
|---|---|
| **Database Login** | Máy chủ: `MISAID MSSQL 04` |
| **Database Token** | Máy chủ: `MSSQL 02`, dung lượng: **&gt; 200GB** |
| **Vấn đề** | Dữ liệu Business đang rải rác ở cả 2 database trên |

---

## B. Mong Muốn

| Mục tiêu | Chi tiết |
|---|---|
| **Tách dữ liệu Business** | Ra database riêng biệt |
| **Database mới** | `MISA.PersonalCustomerProfile.Business` |
| **Máy chủ triển khai** | `MISAID MSSQL 05` |
| **⚠️ Lưu ý** | Cụm `MISAID MSSQL 05` chỉ có **100GB** disk — cần kiểm tra dung lượng dữ liệu Business trước khi migrate |

---

## C. Kế Hoạch Thực Hiện

### Step 1 — Khởi tạo cấu trúc database `MISA.PersonalCustomerProfile.Business`

| STT | Hành động | Chi tiết |
|---|---|---|
| 1.1 | Sinh SQL cấu trúc | Export script DDL từ database `Token` |
| 1.2 | Tạo database rỗng | Tạo `MISA.PersonalCustomerProfile.Business` trên `MISAID MSSQL 05` |
| 1.3 | Chạy script cấu trúc | Apply DDL script vào database mới |
| 1.4 | Giữ lại các bảng cần thiết | Xem danh sách bên dưới |

**Danh sách bảng giữ lại tại `MISA.PersonalCustomerProfile.Business`:**

| STT | Tên bảng |
|---|---|
| 1 | `ClientConnect` |
| 2 | `ContactInfo` |
| 3 | `EducationInfo` |
| 4 | `HighSchoolInfo` |
| 5 | `LocationCountry` |
| 7 | `SharedLicenseLog` |
| 8 | `SuggestionInfo` |
| 9 | `UpdateInfoLog` |
| 10 | `UserUrl` |
| 11 | `WorkInfo` |

---

### Step 2 — Cập nhật dữ liệu cho database Business

| STT | Nguồn | Hành động | Danh sách bảng |
|---|---|---|---|
| 2.1 | `Token` → `Business` | Chuyển dữ liệu từ các bảng tương ứng | Các bảng đã giữ lại ở Step 1 |
| 2.2 | `Login` → `Business` | Sinh DDL + Index script, sau đó chuyển dữ liệu | `Totp`, `MigrateAccountLog`, `LinkActive`, `PasswordRegular`, `LoginMethod`, `UserTwofactorMethodLogs`, `ToolAdminActionLogs`, `UserInfo` |

---

### Step 3 — Chuẩn hóa kiểu dữ liệu `MISA.PersonalCustomerProfile.Business`

| Hành động | Chi tiết |
|---|---|
| Chạy script chuẩn hóa | Theo file đính kèm: `Personal_Customer_Profile_Business` |

---

### Step 4 — Chuẩn hóa kiểu dữ liệu `MISA.PersonalCustomerProfile.Token`

| Hành động | Chi tiết |
|---|---|
| Chạy script chuẩn hóa | Theo file đính kèm: `Personal_Customer_Profile_Token` |

---

### Step 5 — Cập nhật Connection String

| STT | Hành động | Chi tiết |
|---|---|---|
| 5.1 | Xác định các service đang dùng DB `Login` / `Token` | Rà soát config của tất cả ứng dụng liên quan |
| 5.2 | Cập nhật connection string | Trỏ về `MISA.PersonalCustomerProfile.Business` trên `MISAID MSSQL 05` |
| 5.3 | Kiểm tra &amp; smoke test | Verify kết nối + chạy test cơ bản sau khi cập nhật |
