# Thêm Database vào Cụm HA MSSQL (Join Only)

- **Phương pháp:** Join Only (Backup/Restore thủ công + Join vào AG)
- **Áp dụng:** MSSQL Always On Availability Groups

---

## Bước 1 — Chuyển Recovery Model sang FULL

**Thực hiện trên:** Primary (database nguồn)

**GUI:**
```
Database => Properties => Options => Recovery model: Full
```

**T-SQL:**
```sql
ALTER DATABASE [TenDatabase] SET RECOVERY FULL;
```

&gt; ⚠️ Database phải ở chế độ **FULL** trước khi có thể tham gia AG.

---

## Bước 2 — Backup Database

**Thực hiện trên:** Primary

```sql
-- Full Backup
BACKUP DATABASE [TenDatabase]
TO DISK = N'D:\Backup\TenDatabase_Full.bak'
WITH FORMAT, INIT, COMPRESSION, STATS = 10;

-- Log Backup (bắt buộc khi dùng Join Only)
BACKUP LOG [TenDatabase]
TO DISK = N'D:\Backup\TenDatabase_Log.bak'
WITH FORMAT, INIT, COMPRESSION, STATS = 10;
```

&gt; ⚠️ **Bắt buộc phải có Log Backup** khi dùng phương pháp Join Only, khác với Automatic Seeding.

---

## Bước 3 — Restore Database lên các Secondary

**Thực hiện trên:** Tất cả máy Secondary

**Lưu ý:** Copy file `.bak` sang máy Secondary trước khi restore.

```sql
-- Restore Full Backup
RESTORE DATABASE [TenDatabase]
FROM DISK = N'D:\Backup\TenDatabase_Full.bak'
WITH NORECOVERY, STATS = 10;

-- Restore Log Backup
RESTORE LOG [TenDatabase]
FROM DISK = N'D:\Backup\TenDatabase_Log.bak'
WITH NORECOVERY, STATS = 10;
```

&gt; ✅ Sau khi restore, database sẽ ở trạng thái **Restoring...** — đây là trạng thái **đúng**, không phải lỗi.

&gt; ⚠️ **Không dùng** `WITH RECOVERY` trên Secondary, database sẽ không thể join AG.

---

## Bước 4 — Thêm Database vào Availability Group (Primary)

**Thực hiện trên:** Primary

**GUI:**
```
MSSQL => Always On High Availability => Availability Groups
  => [Chọn AG] => Availability Databases => Add Database...
  => Chọn [TenDatabase] => Next => Finish
```

**T-SQL:**
```sql
ALTER AVAILABILITY GROUP [TenAG]
ADD DATABASE [TenDatabase];
```

---

## Bước 5 — Join Database vào Availability Group (Secondary)

**Thực hiện trên:** Tất cả máy Secondary

**GUI:**
```
MSSQL => Always On High Availability => Availability Groups
  => [Chọn AG] => Availability Databases
  => Chuột phải [TenDatabase] => Join to Availability Group... => OK
```

**T-SQL:**
```sql
ALTER DATABASE [TenDatabase]
SET HADR AVAILABILITY GROUP = [TenAG];
```

&gt; ✅ Sau khi join thành công, trạng thái database chuyển từ **Restoring...** sang **Synchronized / Synchronizing**.

---

## Bước 6 — Kiểm tra đồng bộ

**Trên Primary — kiểm tra trạng thái AG:**
```sql
SELECT
    ag.name              AS AG_Name,
    db.database_name,
    rs.role_desc,
    rs.synchronization_state_desc,
    rs.synchronization_health_desc
FROM sys.dm_hadr_availability_replica_states rs
JOIN sys.availability_groups ag ON rs.group_id = ag.group_id
JOIN sys.dm_hadr_database_replica_states db ON rs.replica_id = db.replica_id
WHERE db.database_name = 'TenDatabase';
```

**Kết quả mong đợi:**

| Role | Sync State | Health |
|---|---|---|
| PRIMARY | SYNCHRONIZED | HEALTHY |
| SECONDARY | SYNCHRONIZED | HEALTHY |

---

## Bước 7 — Kiểm nghiệm hoạt động HA

| Hành động | Máy | Kết quả mong đợi |
|---|---|---|
| INSERT/UPDATE/DELETE | Primary | ✅ Thành công |
| Đọc dữ liệu vừa ghi | Secondary | ✅ Dữ liệu đồng bộ |
| INSERT/UPDATE/DELETE | Secondary | ❌ Báo lỗi (read-only replica) |

&gt; ✅ Secondary báo lỗi khi ghi là **hành vi đúng** — Secondary chỉ cho phép đọc (nếu cấu hình readable secondary).
