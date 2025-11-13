# **Cải Tiến Quy Trình Quản Lý Resource Đa Ngôn Ngữ** 🔄

## **Hiện Trạng &amp; Vấn Đề** ❌

### **Quy Trình Hiện Tại**
- **Gửi resource sang platform dịch**:
  - Chỉ thực hiện khi **restart** dịch vụ business-api
- **Lấy nội dung resource theo ngôn ngữ**:
  - Gọi API của platform
  - **Cache** nội dung trong memory (4h)

### **Các Rủi Ro Hiện Tại**
1. **Phụ thuộc vào restart business-api** để cập nhật resource mới
   - Rủi ro cao khi restart API chính của mcp-web

2. **Vấn đề đồng bộ khi triển khai**
   - Restart business-api nhưng resource vẫn cũ do CDN chưa deploy xong
   - Cần phải đợi CDN deploy hoàn tất rồi mới restart business-api

3. **Cache gây trễ cập nhật**
   - Resource đã được dịch nhưng vẫn trả về nội dung cũ do cache memory 4h
   - Buộc phải restart business-api để nạp lại resource mới

## **Giải Pháp Đề Xuất** ✅

### **1. Chuyển Cache Sang Redis**
- Thay thế cache memory bằng **Redis**
- **Lợi ích**: Có thể chủ động clear cache để nạp resource mới mà không cần restart business-api

### **2. Tự Động Hóa Quy Trình Cập Nhật Resource**
- Xây dựng **worker** định kỳ kiểm tra thay đổi từ CDN
- Tự động gửi resource sang platform để dịch khi phát hiện thay đổi

### **3. Cơ Chế Phát Hiện Thay Đổi**
- CDN cung cấp API `/resource-version/` để worker kiểm tra
- Giá trị version được DevOps gắn tự động sau mỗi lần build CDN

## **Kế Hoạch Triển Khai** 🛠️

### **Các Công Việc Cần Thực Hiện**
1. **Cấu hình Redis** để cache resource
   - Thiết lập cơ chế invalidate cache

2. **Phát triển Worker**
   - Lập lịch kiểm tra định kỳ
   - Xử lý logic phát hiện thay đổi và gửi resource

3. **Triển khai API `/resource-version/`**
   - Cấu hình tại mức nginx config
   - Tích hợp với quy trình CI/CD để cập nhật version tự động

### **Lợi Ích Sau Triển Khai**
- **Giảm thiểu downtime**: Không cần restart business-api để cập nhật resource
- **Tự động hóa**: Quy trình cập nhật resource diễn ra tự động
- **Độ tin cậy cao**: Giảm lỗi do sự phụ thuộc vào thứ tự triển khai
