**KỊCH BẢN CHUYỂN ĐỔI ĐIỀU HƯỚNG TRAFFIC: TỪ F5 SANG NGINX INGRESS**

**1. Mục tiêu**
Chuyển đổi mô hình điều hướng từ F5 trực tiếp đến IIS sang mô hình trung gian qua Nginx Ingress (K8s) nhằm tăng khả năng linh hoạt và quản lý tập trung, đảm bảo không gián đoạn dịch vụ (**Zero Downtime**).

**2. Hiện trạng hệ thống**
* **Domain quản lý:** `id.misa.vn` và `misaidapi.misaonline.local`.
* **Mô hình hiện tại:** Traffic $\rightarrow$ F5 $\rightarrow$ Máy chủ IIS (mỗi domain trỏ vào 5 server).

**3. Giải pháp đề xuất**
* **Mô hình đích:** Traffic $\rightarrow$ F5 $\rightarrow$ **Nginx Ingress (K8s)** $\rightarrow$ Máy chủ IIS.
* **Cấu hình Ingress:** Triển khai **20 Pods Nginx Ingress** trên cụm **03 Worker Nodes** chuyên dụng để đảm bảo hiệu năng xử lý luồng traffic lớn.
* **Cơ chế:** Nginx Ingress đóng vai trò lớp điều hướng trung gian, tiếp nhận request từ F5 và phân phối đến các máy chủ IIS.

**4. Kế hoạch triển khai (Chiến lược Rollout cuốn chiếu)**
Việc chuyển đổi được thực hiện theo phương pháp cắt giảm dần tải trọng của mô hình cũ và tăng dần tải trọng cho mô hình mới để kiểm soát rủi ro.

* **Giai đoạn 1: Chuẩn bị hạ tầng**
    * Kiểm tra tính sẵn sàng của cụm 20 Pods Nginx Ingress trên 03 Worker Nodes.
    * Xác nhận cơ chế điều hướng từ Ingress đến các máy chủ IIS đã hoạt động chính xác trong môi trường thử nghiệm.

* **Giai đoạn 2: Thực hiện chuyển đổi từng phần (Phased Migration)**
    * **Bước 1:** Trên F5, thực hiện ngắt kết nối (disable) **01 máy chủ IIS** khỏi nhóm điều hướng trực tiếp.
    * **Bước 2:** Ngay lập tức cấu hình thêm máy chủ IIS đó vào danh sách điều hướng của **Nginx Ingress**.
    * **Bước 3:** Lặp lại quy trình cho đến khi tất cả 05 máy chủ IIS đều được điều hướng thông qua Nginx Ingress.

* **Giai đoạn 3: Kiểm soát và Hoàn tất**
    * Giám sát lưu lượng và tỉ lệ lỗi (error rate) trong suốt quá trình chuyển đổi.
    * Sau khi hoàn tất việc chuyển hướng toàn bộ traffic qua Ingress, thực hiện cấu hình lại F5 để chỉ đóng vai trò dẫn luồng đến cụm K8s.

**5. Tài nguyên sử dụng**
* **Máy chủ đích:** 05 Máy chủ IIS.
* **Hạ tầng K8s:** 03 Worker Nodes chuyên dụng cho Ingress.
* **Lớp điều hướng:** 20 Pods Nginx Ingress.
