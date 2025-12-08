# Quy trình phối hợp team thi công và DevOps
09/12/2025

**Ghi chú:**
- DevOps luôn đồng hành “cuốn chiếu” theo tiến độ Team Thi công.
- Mọi vấn đề phát sinh cần chủ động trao đổi, thông báo hai chiều sớm nhất.

---

| **Bước/Luồng Phối hợp** | **Vai trò Team Thi công** | **Vai trò Team DevOps** | **Mục tiêu/Chú ý quan trọng** |
|---|---|---|---|
| **1. Thông báo build phát hành** | - Thông báo trước 9h sáng mỗi ngày.<br>- Gửi rõ: Tên dịch vụ, phiên bản, phạm vi thay đổi, môi trường cần triển khai. | - Tiếp nhận thông tin, lên kế hoạch chuẩn bị hạ tầng và pipeline. | Đảm bảo DevOps chủ động sắp xếp, không thiếu thông tin. |
| **2. Khởi động dịch vụ/solution mới** | - Cung cấp thông tin dịch vụ: tên, sơ đồ kết nối, kiến trúc tổng quan. | - Chuẩn bị Helm chart, cấu hình các nhánh pipeline CI/CD.<br>- Cấu hình ingress nếu có nhu cầu kết nối từ ngoài. | Chuẩn hóa thông tin ngay từ đầu để tránh phát sinh về sau. |
| **3. Thi công song song CI/CD** | - Code, build dịch vụ tại môi trường DEV.<br>- Gửi cấu hình cho DevOps khi cần triển khai lên Local. | - Cập nhật Helm chart/pipeline dựa trên thông tin cung cấp.<br>- Thử nghiệm triển khai trên Local. | Dev và DevOps phối hợp cuốn chiếu, không đợi hoàn thiện toàn bộ mới bắt đầu triển khai CI/CD. |
| **4. Nghiệm thu dịch vụ** | - Test kỹ dịch vụ sau khi DevOps triển khai Local.<br>- Thông báo kết quả nghiệm thu (routing, kết nối ngoại vi,...) | - Hoàn thiện, điều chỉnh cấu hình nếu cần.<br>- Bàn giao luồng CI/CD ổn định. | Đảm bảo dịch vụ hoạt động đúng/yêu cầu trên môi trường đã triển khai. |
| **5. Nguyên tắc làm việc chung** | - Thông báo sớm, cung cấp đủ nhu cầu kỹ thuật cho DevOps.<br>- Sử dụng template yêu cầu thống nhất khi cần triển khai mới. | - Thực hiện mọi thay đổi hạ tầng dựa trên thông tin được yêu cầu.<br>- Chủ động hỗ trợ khi team thi công phát sinh vấn đề triển khai. | Tăng hiệu quả phối hợp, tối ưu tiến độ dự án, tránh bỏ sót thông tin/hạ tầng. |


