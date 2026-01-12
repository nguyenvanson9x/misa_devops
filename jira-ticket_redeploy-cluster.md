### **🗺️ Kế hoạch Triển khai lại Cụm Jira Data Center**

Bản kế hoạch này mô tả các bước di chuyển kho lưu trữ và khôi phục hoạt động cho cụm Jira 4 node.

#### **I. Phân tích Hiện trạng**

*   **Lưu trữ:**
    *   **Dữ liệu Local (Index, logs):** Lưu trên ổ đĩa cục bộ của từng máy chủ.
    *   **Dữ liệu Dùng chung (Attachments, avatars...):** Lưu trên hệ thống lưu trữ Ceph.
*   **Trạng thái Cụm:**
    *   Chỉ có một node (`instance 02`) đang hoạt động.
    *   Các node còn lại (`01`, `03`, `04`) đang ở trạng thái dừng.

#### **II. Mục tiêu**

1.  **Di chuyển lưu trữ:** Chuyển toàn bộ dữ liệu dùng chung từ Ceph sang máy chủ NFS.
2.  **Khôi phục cụm:** Đưa cụm Jira về trạng thái hoạt động ổn định với đầy đủ 4 node.

---

### **III. Các Giai đoạn Thực thi**

Kế hoạch được chia thành 3 giai đoạn chính: **Chuẩn bị**, **Triển khai (Downtime)**, và **Kiểm nghiệm**.

#### **Giai đoạn 1: Chuẩn bị (Không yêu cầu Downtime)**

Giai đoạn này thực hiện các công việc chuẩn bị nền tảng và sao chép dữ liệu lần đầu.

1.  **Chuẩn bị Máy chủ NFS:**
    *   **Dọn dẹp:** Đăng nhập vào máy chủ NFS và xóa các thư mục, dữ liệu cũ không còn sử dụng để đảm bảo đủ dung lượng.
    *   **Tạo thư mục:** Tạo một thư mục mới trên NFS dành riêng cho dữ liệu dùng chung của Jira.
2.  **Sao chép Dữ liệu Dùng chung (Lần đầu):**
    *   **Mục tiêu:** Giảm thiểu thời gian sao chép trong lúc downtime.
    *   **Hành động:** Thực hiện sao chép toàn bộ dữ liệu dùng chung từ Ceph sang thư mục mới trên NFS.
    *   **Lưu ý:** Vì dữ liệu lớn, quá trình này nên được thực hiện trước. Những thay đổi phát sinh sau lần sao chép này sẽ được đồng bộ ở giai đoạn sau.

#### **Giai đoạn 2: Triển khai Chính (Yêu cầu Downtime)**

Đây là giai đoạn cốt lõi, yêu cầu tạm dừng hệ thống để thực hiện các thay đổi quan trọng.

1.  **Bắt đầu Downtime:**
    *   **Ngắt Dịch vụ:** Dừng dịch vụ **HAProxy** và **Jira** trên tất cả các node.
2.  **Đồng bộ Dữ liệu Dùng chung (Lần cuối):**
    *   Thực hiện sao chép lại một lần nữa từ Ceph sang NFS để đảm bảo tất cả các thay đổi mới nhất được cập nhật.
3.  **Cấu hình lại các Node Jira:**
    *   Thay đổi cấu hình của Jira để trỏ đến thư mục chia sẻ trên NFS thay vì Ceph.
4.  **Đồng bộ Dữ liệu Local:**
    *   **Xóa thông tin Node cũ:** Truy cập vào cơ sở dữ liệu `jira_ticket` và xóa các bản ghi trong bảng `clusternode` để làm sạch thông tin cụm.
    *   **Lấy Dữ liệu Gốc:** Dữ liệu local (bao gồm cả index) trên `jira_ticket_worker02` được chọn làm dữ liệu gốc chuẩn.
    *   **Nhân bản Dữ liệu:** Sao chép toàn bộ thư mục dữ liệu local từ `worker02` sang cho các `worker 01`, `03`, và `04`.
5.  **Khởi động lại Cụm (Tuần tự):**
    *   **Mục tiêu:** Khởi động có kiểm soát để dễ dàng theo dõi và xử lý lỗi.
    *   **Thứ tự khởi động:**
        1.  Khởi động Jira trên **Worker 02**. Đợi và kiểm tra đến khi node này hoạt động ổn định.
        2.  Khởi động Jira trên **Worker 01**.
        3.  Khởi động Jira trên **Worker 03**.
        4.  Khởi động Jira trên **Worker 04**.
        5.  Cuối cùng, khởi động lại dịch vụ **HAProxy**.

#### **Giai đoạn 3: Kiểm nghiệm và Hoàn tất**

Giai đoạn cuối cùng để xác nhận hệ thống đã hoạt động đúng như mong đợi.

1.  **Kiểm tra Trạng thái HAProxy:**
    *   Truy cập vào trang dashboard thống kê (`/stats`) của HAProxy.
    *   **Xác nhận:** Cả 4 node Jira backend đều phải ở trạng thái **UP** (màu xanh).
2.  **Kiểm tra Trạng thái Cụm Jira:**
    *   Đăng nhập vào giao diện quản trị của Jira.
    *   Đi tới mục **Clustering** để kiểm tra.
    *   **Xác nhận:** Cả 4 node đã tham gia vào cụm thành công và trạng thái index ổn định.
3.  **Kiểm tra Chức năng:**
    *   Thực hiện các thao tác cơ bản như tạo ticket, đính kèm file để đảm bảo hệ thống hoạt động bình thường.
