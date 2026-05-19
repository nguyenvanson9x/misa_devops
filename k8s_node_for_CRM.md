**✅ Mục tiêu (Output)**  
- 4 node **chỉ chạy CRM**  
- Pod khác **bị đẩy sang node khác**  
- Không downtime (hoặc minimal)

---

# **Chiến lược (Lever)**
Bạn cần **3 bước kết hợp**:

1. **Taint node (chặn pod thường)**
2. **Toleration cho CRM**
3. **Drain node (đuổi pod hiện tại)**

---

# **1. Taint 4 node (chặn pod khác)**

```bash
kubectl taint node node1 appCode=CRM:NoSchedule
kubectl taint node node2 appCode=CRM:NoSchedule
kubectl taint node node3 appCode=CRM:NoSchedule
kubectl taint node node4 appCode=CRM:NoSchedule
```

👉 Kết quả:
- Pod KHÔNG có toleration → **không schedule vào**
- Node trở thành **CRM-only**

---

# **2. Gắn label (để control scheduling CRM)**

```bash
kubectl label node node1 appCode=CRM
kubectl label node node2 appCode=CRM
kubectl label node node3 appCode=CRM
kubectl label node node4 appCode=CRM
```

---

# **3. Update Deployment CRM**

```yaml
spec:
  template:
    spec:
      nodeSelector:
        appCode: CRM
      tolerations:
        - key: "appCode"
          operator: "Equal"
          value: "CRM"
          effect: "NoSchedule"
```

👉 CRM:
- **Chỉ chạy trên 4 node**
- **Vượt qua taint**

---

# **4. Đuổi pod hiện tại khỏi 4 node (CRITICAL)**

⚠️ Nếu không làm bước này → pod cũ vẫn nằm đó

```bash
kubectl drain node1 --ignore-daemonsets --delete-emptydir-data
kubectl drain node2 --ignore-daemonsets --delete-emptydir-data
kubectl drain node3 --ignore-daemonsets --delete-emptydir-data
kubectl drain node4 --ignore-daemonsets --delete-emptydir-data
```

👉 Kết quả:
- Pod non-CRM bị evict
- Scheduler sẽ **đẩy sang node khác**

---

# **5. Sau khi drain xong → uncordon**

```bash
kubectl uncordon node1
kubectl uncordon node2
kubectl uncordon node3
kubectl uncordon node4
```

---

# **6. Đảm bảo không downtime (IMPORTANT)**

Trước khi drain:

### ✅ Check PDB (PodDisruptionBudget)

```bash
kubectl get pdb -A
```

Nếu chưa có → tạo:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: crm-pdb
spec:
  minAvailable: 80%
  selector:
    matchLabels:
      app: crm
```

---

# **Tóm tắt (Input → Lever → Output)**

- **Input**: 4 node đang có workload khác
- **Lever**:
  - Taint: block pod thường
  - Toleration: cho CRM vào
  - Drain: đuổi pod hiện tại
- **Output**:
  - 4 node = **CRM-only**
  - Pod khác auto migrate
  - Hệ thống vẫn hoạt động

---

# **Lưu ý thực tế (DevOps critical)**

- ❌ Không drain cùng lúc nếu cluster tight resource  
  → nên làm **rolling (1 node/lần)**

- ✅ Ưu tiên:
  ```bash
  kubectl drain node1 ...
  # verify
  kubectl drain node2 ...
  ```
