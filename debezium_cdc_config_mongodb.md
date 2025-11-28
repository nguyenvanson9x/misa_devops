# 📋 Hướng dẫn cấu hình CDC cho MongoDB
## **Sơ đồ tóm tắt luồng xử lý**

1️⃣ **Xin tài khoản DB**  
⬇️  
2️⃣ **Nhận user, password từ QTHT**  
⬇️  
3️⃣ **Cấu hình &amp; tạo connector trên Debezium**  
⬇️  
4️⃣ **Connector đọc dữ liệu CDC từ MongoDB**  
⬇️  
5️⃣ **Đẩy dữ liệu thay đổi ra topic Kafka**  
⬇️  
6️⃣ **Kiểm tra log và dữ liệu trên Kafka**

## **1. Chuẩn bị tài khoản kết nối MongoDB**

### **1.1. Thông tin yêu cầu cấp tài khoản**
- **Tài khoản**: mcp_debezium
- **Mật khẩu**: Do QTHT tự sinh và gửi lại
- **Danh sách máy chủ cần truy cập**:
  | Địa chỉ IP | Môi trường |
  |------------|------------|
  | 1.2.3.4 | Tri thức Việt |
  | 1.2.3.5 | MISA |

### **1.2. Câu lệnh tạo tài khoản MongoDB**

```javascript
// Tạo role replication
use admin
db.createRole({
  role: "replication",
  privileges: [
    { resource: { db: "local", collection: "" }, actions: [ "find" ] }
  ],
  roles: []
});

// Tạo user với quyền đọc và replication
db.createUser({
  user: "mcp_debezium",
  pwd: "account_password",
  roles: [
    { role: "replication", db: "admin" },
    "readAnyDatabase"
  ]
})
```

&gt; **Lưu ý**: Thay `account_password` bằng mật khẩu do QTHT cung cấp.

## **2. Thiết lập Connector trên Debezium**

### **2.1. Xác định cụm Debezium phù hợp**
- Các cụm Debezium đã được quy hoạch theo từng nhóm sản phẩm
- Chọn cụm phù hợp với dự án/sản phẩm của bạn

### **2.2. Xây dựng cấu hình Connector**

#### **Các thông số cần cấu hình**:
- **name**: Tên định danh cho connector
- **mongodb.connection.string**: Chuỗi kết nối MongoDB (ví dụ: `mongodb://mongodb_server_name:27017/?replicaSet=rs0`)
- **mongodb.user**: Tài khoản kết nối (mcp_debezium)
- **mongodb.password**: Mật khẩu tài khoản
- **collection.include.list**: Danh sách các collection cần theo dõi, định dạng `{database_name}.{collection_name}`, ngăn cách bằng dấu phẩy
- **topic.prefix**: Tiền tố cho topic Kafka
- **snapshot.mode**: Thiết lập là "never"

#### **Mẫu cấu hình Connector hoàn chỉnh**:

```json
{
  "name": "mcp_inbot02_datalake_connector",
  "connector.class": "io.debezium.connector.mongodb.MongoDbConnector",
  "connector.displayName": "MongoDB",
  "connector.id": "mongodb",
  "mongodb.connection.string": "mongodb://mongodb_server_name:27017/?replicaSet=rs0",
  "mongodb.user": "mcp_debezium",
  "mongodb.password": "your_password",
  "collection.include.list": "inbot_invoice_data_21.InvoiceInfo,inbot_invoice_data_22.InvoiceInfo",
  "topic.prefix": "mcp_inbot02_datalake",
  "snapshot.mode": "never",
  
  "errors.log.include.messages": "true",
  "max.queue.size": "1024",
  "incremental.snapshot.chunk.size": "256",
  "max.queue.size.in.bytes": "5000",
  "heartbeat.interval.ms": "1000",
  "include.schema.changes": "false",
  "key.converter.schemas.enable": "false",
  "value.converter.schemas.enable": "false",
  "errors.tolerance": "all",
  "value.converter": "org.apache.kafka.connect.json.JsonConverter",
  "max.batch.size": "256",
  "errors.log.enable": "true",
  "key.converter": "org.apache.kafka.connect.json.JsonConverter"
}
```

### **2.3. Tạo Connector trên Debezium Tool**

1. **Chọn cụm Debezium** → Bấm **Connect**
2. **Bấm "Create Connector"**
3. **Nhập thông tin**:
   - **Name**: Tên connector
   - **Connector class**: Chọn MongoDB connector
   - **Configuration (JSON)**: Dán cấu hình JSON đã chuẩn bị
4. **Bấm SAVE** để hoàn tất

## **3. Kiểm tra hoạt động của Connector**

### **3.1. Kiểm tra log Debezium**
- **Tìm log theo tên connector** - Không có log error là thành công
- **Xem log trên K8s**: 
  ```
  Gen2-argocd → mcp-debezium app → chọn deployment debezium tương ứng → Xem log
  ```

### **3.2. Kiểm tra dữ liệu trên Kafka**
- Xem thông tin các topic theo `topic.prefix` đã cấu hình
- Sử dụng công cụ kafka-ui để kiểm tra chi tiết dữ liệu

---

## **Các lưu ý quan trọng**
- Đảm bảo MongoDB đã được cấu hình dưới dạng Replica Set
- Tài khoản kết nối cần có đủ quyền đọc và replication
- Kiểm tra kỹ cấu hình `collection.include.list` để đảm bảo đúng tên database và collection
- Thiết lập `snapshot.mode` là "never" để chỉ theo dõi các thay đổi mới
