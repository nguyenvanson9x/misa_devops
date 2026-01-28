### 🚀 **Quy trình Nâng cấp Apache Superset Lên Phiên bản 6.0.0 (Linux/venv/systemd)**

Tài liệu này hướng dẫn các bước tiêu chuẩn để nâng cấp hệ thống Apache Superset được cài đặt qua pip trong môi trường ảo (venv) và quản lý bằng systemd.

- **Phiên bản mục tiêu:** `6.0.0`
- **Môi trường:** Linux, Python venv, PostgreSQL
- **Thành phần:** Superset Web, Superset Worker, Superset Beat

---

### 📋 **Checklist Nâng cấp**

#### **Giai đoạn 1: Chuẩn bị và Backup (Thực hiện trên tất cả các node Web &amp; Worker)**

**1. Dừng tất cả dịch vụ Superset** 🛑
*Đảm bảo không có tiến trình nào đang chạy để tránh xung đột dữ liệu.*
```bash
# Dừng máy chủ Web
sudo systemctl stop superset-web

# Dừng Celery Beat (nếu có)
sudo systemctl stop superset-beat

# Dừng Celery Worker
sudo systemctl stop superset-worker

# Kiểm tra lại trạng thái để chắc chắn dịch vụ đã dừng hẳn
sudo systemctl status superset-web superset-beat superset-worker
```

**2. Backup Toàn diện** 💽
*   **Backup mã nguồn:**
    ```bash
    # Tạo bản sao của thư mục cài đặt hiện tại
    sudo cp -r /opt/superset /opt/superset_backup_$(date +%Y%m%d)
    ```

*   **Backup Metadata Database (PostgreSQL):** *(Thực hiện trên máy chủ có quyền truy cập DB)*
    ```bash
    # Sử dụng pg_dump để xuất dữ liệu từ database superset
    pg_dump -h mcp-pg-a02-01 -p 5432 -U mcp_user -d superset &gt; superset_db_backup_$(date +%Y%m%d).sql
    ```
    **Lưu ý:** Hãy xác thực tệp backup không bị lỗi và có thể restore được.

---

#### **Giai đoạn 2: Nâng cấp (Thực hiện trên tất cả các node Web &amp; Worker)**

**3. Kích hoạt môi trường ảo (venv)**
*Tất cả các lệnh `pip` và `superset` tiếp theo phải được thực thi trong môi trường này.*
```bash
source /opt/superset/venv/bin/activate
```

**4. Nâng cấp phiên bản Apache Superset**
*Chỉ định rõ phiên bản `6.0.0` để đảm bảo tất cả các node đều được đồng bộ.*
```bash
pip install --upgrade "apache-superset==6.0.0"
```
*Sau khi cài đặt, kiểm tra lại phiên bản để xác nhận:*
```bash
superset version
```

---

#### **Giai đoạn 3: Di chuyển Dữ liệu (Chỉ thực hiện trên 1 node Web)**

**5. Nâng cấp Schema và Khởi tạo lại Metadata**
*Bước này sẽ áp dụng các thay đổi về cấu trúc database cho phiên bản mới và làm mới các quyền mặc định.*
```bash
# Kích hoạt venv nếu chưa có
source /opt/superset/venv/bin/activate

# Chạy di chuyển dữ liệu
superset db upgrade

# Đồng bộ lại quyền và các thiết lập mặc định
superset init
```

---

#### **Giai đoạn 4: Khởi động và Xác minh**

**6. Khởi động lại dịch vụ** ✅
*Khởi động theo thứ tự để đảm bảo các thành phần phụ thuộc hoạt động đúng.*
```bash
# Start máy chủ Web
sudo systemctl start superset-web

# Start Celery Beat
sudo systemctl start superset-beat

# Start Celery Worker
sudo systemctl start superset-worker
```

**7. Kiểm tra và Xác minh hoạt động**
*   **Kiểm tra trạng thái dịch vụ:**
    ```bash
    sudo systemctl status superset-web superset-beat superset-worker
    ```

*   **Theo dõi logs để phát hiện lỗi (trên từng node tương ứng):**
    ```bash
    # Xem log của máy chủ Web
    journalctl -u superset-web -f
    
    # Xem log của Worker
    journalctl -u superset-worker -f
    ```

*   **Kiểm tra chức năng trên giao diện người dùng:**
    1.  Truy cập Superset và xác nhận phiên bản đã được cập nhật.
    2.  Mở một vài dashboard phức tạp để kiểm tra.
    3.  Thực thi một tác vụ bất đồng bộ (ví dụ: export CSV) để xác nhận Celery Worker hoạt động chính xác.
