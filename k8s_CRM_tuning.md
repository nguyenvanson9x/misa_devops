**BÁO CÁO NHANH VỀ TÌNH TRẠNG QUÁ TẢI CỤM K8S VÀ ĐỊNH HƯỚNG TỐI ƯU**

**1. Hiện trạng**
- Ngày hôm qua, cụm **k8s-misa** xảy ra tình trạng **quá tải RAM**.
- Qua kiểm tra nhanh, nguyên nhân chính đến từ các **dịch vụ CRM sử dụng tài nguyên cao**.

**2. Xử lý tạm thời**
- Đã thực hiện **di chuyển các dịch vụ CRM từ cụm k8s-misa sang k8s-cntt**.
- Kết quả: tình trạng quá tải trên **k8s-misa đã được xử lý ổn định**.

**3. Đánh giá hiện tại**
- Tổng số pod của CRM đang ở mức **~254 pod**, chiếm khoảng **4 node Kubernetes**.
- Một số dịch vụ đang **over-provisioning**, có thể tối ưu:
  - Ví dụ: `trace-log-other`, `cdc-kafka-partner` đang chạy ~10 pod nhưng có thể giảm xuống **~3 pod** mà vẫn đảm bảo tải.

**4. Vấn đề tồn tại**
- Việc cấu hình số lượng pod hiện tại **chưa tối ưu theo tải thực tế**.
- Có dấu hiệu **leak tài nguyên (RAM/CPU)** ở một số dịch vụ khi chạy lâu.
- Chưa áp dụng cơ chế **autoscaling theo tải động**, dẫn đến lãng phí tài nguyên.

**5. Định hướng xử lý**
- **Phía nghiệp vụ (anh Đại)**:
  - Rà soát lại toàn bộ các dịch vụ CRM.
  - Xác định mức tải thực tế và cấu hình lại số lượng pod phù hợp.

- **Phía DevOps**:
  - Triển khai **KEDA (Kubernetes Event-Driven Autoscaling)**:
    - Tự động scale pod theo **resource usage / message queue (Kafka, RabbitMQ,...)**.
    - Giảm pod khi tải thấp, tăng pod khi tải cao.

  - **Bổ sung cơ chế xử lý leak tài nguyên (ngắn hạn)**:
    - Thiết lập **CronJob tự động restart pod định kỳ** (off-peak giờ thấp tải).
    - Áp dụng cho các service có dấu hiệu leak RAM/CPU.
    - Mục tiêu: giải phóng tài nguyên, tránh tích tụ gây quá tải node.

  - Xem xét bổ sung:
    - Chuẩn hóa **resource request/limit**
    - Áp dụng **node affinity / pod anti-affinity** để phân bổ tải hợp lý giữa các node

**6. Kỳ vọng**
- Giảm thiểu tình trạng **quá tải tài nguyên đột ngột**.
- Hạn chế ảnh hưởng từ **memory leak trong ngắn hạn**.
- Tối ưu chi phí hạ tầng.
- Hệ thống vận hành **ổn định và linh hoạt theo tải thực tế**.
