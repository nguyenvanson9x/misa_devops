**🎯 Mục tiêu**
- Tăng tính sẵn sàng cao cho hệ thống **Jira-ticket** bằng **cluster 4 node**.
- Phân tải qua **HAProxy** đa instance, tự động loại trừ node lỗi.
- Đảm bảo đủ tài nguyên cho tải hiện tại và tăng trưởng.

---

**🧩 Hiện trạng**

- **Jira**: Đã cấu hình dạng cluster nhưng hiện chỉ chạy **1 node active**.
- **Hạ tầng**: 4 server Docker trong cùng cụm:
  - hl-ticket-dk01b: hiện đang chạy Jira
    - 16 core, 64 GB RAM
  - 3 server chưa sử dụng:
    - hl-ticket-dk01a: 8 core, 32 GB RAM
    - pv-ticket-dk01c: 8 core, 32 GB RAM
    - pv-ticket-dk01d: 8 core, 24 GB RAM
- **Tải trung bình** hiện tại:
  - RAM: ~36 GB
  - CPU: ~6 core

---

**🛠 Đề xuất kỹ thuật**

1) **Mở rộng Jira cluster**
- Tăng từ **1 lên 4 instance Jira**.
- Cài 3 instance mới trên các node:
  - hl-ticket-dk01a
  - pv-ticket-dk01c
  - pv-ticket-dk01d
- Lưu ý bắt buộc:
  - Dùng shared storage và shared index (Data Center yêu cầu).
  - Cấu hình node affinity và unique node ID cho từng instance.
  - Đồng bộ version và plugin giữa các node.

2) **Load Balancer: HAProxy**
- Triển khai **8 instance HAProxy** trên 4 node Docker (mỗi node 2 instance) để tăng HA và khả năng rolling reload.
- Health check:
  - HTTP path: **/**
  - Interval: **5s**
  - Mark fail: **2 lần lỗi liên tiếp**
  - Mark pass: **2 lần đạt liên tiếp**
- Sticky sessions: sử dụng **JSESSIONID** theo khuyến nghị Jira Data Center.

3) **Nâng cấp tài nguyên trước khi triển khai**
- Nâng cấp 3 server mới lên cấu hình đồng nhất: **16 core, 64 GB RAM** cho:
  - hl-ticket-dk01a, pv-ticket-dk01c, pv-ticket-dk01d
- Mục tiêu: đảm bảo đồng đều tài nguyên giữa các node trong cluster để tránh bottleneck.

---

**📋 Kế hoạch triển khai theo bước**

- Bước 0: Chuẩn bị
  - Nâng cấp tài nguyên 3 server lên 16C-64G.
  - Chuẩn bị shared home và shared index.
  - Kiểm tra phiên bản Jira và plugin đồng nhất.

- Bước 1: Triển khai HAProxy
  - Cài 2 instance HAProxy trên mỗi node Docker.
  - Cấu hình backend trỏ tới 4 node Jira.
  - Áp dụng health check 5s, 2 fail down, 2 pass up.
  - Bật sticky session theo cookie Jira.

- Bước 2: Thêm 3 Jira node
  - Cài đặt Jira Data Center trên 01a, 01c, 01d.
  - Cấu hình node ID, kết nối shared home, DB.
  - Join cluster, đồng bộ index.
  - Thêm từng node vào pool HAProxy, canary test.

- Bước 3: Kiểm thử
  - Kiểm tra functional, login, attachment, search.
  - Failover test: dừng 1 node Jira, dừng 1 instance HAProxy.
  - Kiểm tra performance: throughput, response time, GC.

- Bước 4: Chuyển đổi và giám sát
  - Chuyển traffic chính sang VIP mới.
  - Bật quan sát qua Grafana: CPU, RAM, GC, thread, DB pool, LB metrics.
  - Thiết lập alert: node down, 5xx tăng, latency tăng.

- Bước 5: Rollback plan
  - Có sẵn cấu hình để quay về 1 node cũ trên hl-ticket-dk01b nếu có sự cố.
  - Snapshot cấu hình HAProxy và Jira trước khi rollout.

---

**✅ Tiêu chí hoàn thành**
- 4 node Jira hoạt động trong cluster, đồng bộ index.
- HAProxy phân tải ổn định, sticky session hoạt động.
- Tải thực tế phân bổ đều, CPU mỗi node &lt; 40%, RAM còn trống &gt; 30%.
- Failover tự động trong &lt; 15 giây khi 1 node Jira lỗi.
- Người dùng không gián đoạn đáng kể trong giờ triển khai.

---

**📎 Ghi chú vận hành**
- JVM sizing: bắt đầu với **Xms=Xmx ~ 16–24 GB** mỗi node, GC G1, theo dõi GC pause để tinh chỉnh.
- Thread pool và DB connection pool tăng dần theo tải; tránh overcommit DB.
- Lịch bảo trì: rolling restart từng node để không downtime.
- Sao lưu: DB + shared home trước khi rollout.

---

**🔍 Rủi ro và cách giảm thiểu**
- Không đồng bộ plugin/phiên bản: chuẩn hóa artifact, bật kiểm tra CI.
- Sticky session sai: xác thực cookie trên thực tế, test tải JMeter.
- Thắt cổ chai DB: giám sát slow query, tăng index, scale DB nếu cần.

---

**🧪 Kiểm thử nhanh đề xuất**
- Smoke test: login, tạo issue, attach file, search, board.
- Chaos test nhẹ: kill -9 1 node Jira, xác minh HAProxy remove trong ~10s, restore trong ~10s.
- Tải: 10–20% cao hơn bình thường trong 30 phút để quan sát.

---

Bạn muốn mình tạo checklist triển khai chi tiết dạng runbook hoặc file cấu hình mẫu HAProxy + systemd service cho từng node không?
