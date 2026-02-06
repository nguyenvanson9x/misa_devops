### **Cải tiến hệ thống giám sát API**

#### **🎯 Mục tiêu chính**
*   Ghi nhận toàn bộ lượt gọi (request) vào hệ thống.
*   Xác định chính xác **ai (client nào)** đang gọi API.
*   Phân tích lưu lượng truy cập theo từng client cụ thể.

---

#### **📉 Thực trạng Hệ thống**
*   **Không đồng nhất về công nghệ:**
    *   **REST API:** Đã tích hợp giám sát hiệu năng (APM), theo dõi được tổng lượt gọi.
    *   **WCF Services:** Công nghệ cũ, không tích hợp được APM, tạo ra **"điểm mù" (blind spot)**, không có dữ liệu giám sát.
*   **Thiếu khả năng định danh người gọi:**
    *   APM hiện tại chưa được cấu hình để phân tách request theo client.
    *   Không thể trả lời câu hỏi: *"Client nào (Mobile, Web, Partner) đang tạo ra nhiều traffic nhất?"*
*   **Hệ quả:**
    *   Không có dữ liệu để tối ưu tài nguyên.
    *   Khó phát hiện các vấn đề phát sinh từ một client đơn lẻ (ví dụ: một client gọi tăng đột biến gây quá tải).

---

#### **🛠️ Giải pháp Đề xuất**

##### **1. Chuẩn hóa Định danh Client qua Header `X-API-Key`**
*   **Mục đích:** Cung cấp một phương thức chuẩn để client tự định danh.
*   **Quy ước:**
    *   Mỗi client (ứng dụng, đối tác) sẽ được cấp một `API Key` duy nhất.
    *   Client **bắt buộc** phải gửi kèm `API Key` này trong header `X-API-Key` của mỗi request.
*   **Lợi ích:**
    *   Tạo ra một thuộc tính (attribute) chung để lọc và nhóm các request trên toàn hệ thống.

##### **2. Cấu hình APM cho REST API**
*   **Mục đích:** Tận dụng công cụ APM có sẵn để phân tích traffic theo client.
*   **Hành động:**
    *   **Bật tính năng "Capture Headers"** trên hệ thống APM.
    *   Cấu hình để APM tự động ghi nhận và lưu trữ giá trị của header `X-API-Key`.
*   **Kết quả:**
    *   Có thể dễ dàng lọc, tìm kiếm và tạo báo cáo dựa trên `X-API-Key` ngay trên giao diện APM.
    *   **Không cần thay đổi code ứng dụng**, chỉ cần thay đổi cấu hình của APM.

##### **3. Hiện đại hóa WCF Services**
*   **Mục đích:** Loại bỏ "điểm mù" giám sát và đồng bộ hóa công nghệ.
*   **Kế hoạch:** Nâng cấp các dịch vụ WCF cũ lên công nghệ **REST API**.
*   **Lợi ích:**
    *   Sau khi nâng cấp, các dịch vụ này sẽ được tích hợp APM.
    *   Áp dụng được ngay cơ chế giám sát qua `X-API-Key` một cách đồng bộ.

---

#### **🚀 Kế hoạch Hành động**
*   **Giai đoạn 1: Chuẩn bị &amp; Áp dụng cho REST API**
    *   Xây dựng hệ thống quản lý và cấp phát `API Key`.
    *   Cấu hình APM để ghi nhận header `X-API-Key`.
    *   Làm việc với các bên liên quan để họ bắt đầu gửi kèm `API Key` trong các request.
    *   Danh sách dịch vụ: Tất cả dịch vụ dùng Rest API
*   **Giai đoạn 2: Nâng cấp WCF**
    *   Đội dự án đánh giá và lên lộ trình chi tiết để chuyển đổi WCF sang REST API.
    *   Ưu tiên các dịch vụ quan trọng hoặc có traffic cao để chuyển đổi trước.
    *   Danh sách dịch vụ: MISA SUMAN (suman-service)
