# Cấu hình DataProtection trong .NET Core 2.1 với Redis cho nhiều máy chủ IIS

## 1. Giới thiệu DataProtection

**DataProtection** là cơ chế bảo mật trong .NET Core dùng để mã hóa dữ liệu nhạy cảm như token, cookie, chuỗi bảo mật. Đặc biệt quan trọng khi triển khai ứng dụng trên nhiều máy chủ (web farm).

## 2. Cài đặt các package cần thiết

```
Install-Package Microsoft.AspNetCore.DataProtection
Install-Package Microsoft.AspNetCore.DataProtection.StackExchangeRedis
Install-Package StackExchange.Redis
```

## 3. Cấu hình lưu key vào Redis

Thêm vào `Startup.cs` trong phương thức `ConfigureServices`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Kết nối Redis
    var redis = ConnectionMultiplexer.Connect("your-redis-connection-string");
    
    // Cấu hình DataProtection
    services.AddDataProtection()
        .SetApplicationName("MISAID")  // Đảm bảo tất cả instance dùng cùng tên
        .PersistKeysToStackExchangeRedis(redis, "MISAID-DataProtection-Keys");
}
```

## 4. Cấu hình chuyển đổi giữa Redis và FileShare

Để chuyển đổi giữa Redis và FileShare dựa trên cấu hình trong appsettings.json:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Thêm DataProtection service
    var dataProtection = services.AddDataProtection()
        .SetApplicationName("MISAID");
        
    // Đọc cấu hình từ appsettings.json
    var keyStorageType = Configuration["DataProtection:KeyStorageType"];
    
    if (keyStorageType == "Redis")
    {
        var redisConnection = Configuration["DataProtection:RedisConnection"];
        var redis = ConnectionMultiplexer.Connect(redisConnection);
        dataProtection.PersistKeysToStackExchangeRedis(redis, "MISAID-DataProtection-Keys");
    }
    else if (keyStorageType == "FileShare") 
    {
        var fileSharePath = Configuration["DataProtection:FileSharePath"];
        dataProtection.PersistKeysToFileSystem(new DirectoryInfo(fileSharePath));
    }
}
```

Cấu hình tương ứng trong `appsettings.json`:

```json
{
  "DataProtection": {
    "KeyStorageType": "Redis",
    "RedisConnection": "your-redis-server:6379,password=yourpassword",
    "FileSharePath": "\\\\server\\share\\MISAID-DataProtection-Keys"
  }
}
```

## 5. Lưu ý quan trọng khi triển khai trên nhiều máy chủ IIS

1. **Vòng đời key**: Mặc định key có thời hạn 90 ngày, sau đó tự động tạo key mới.

2. **Không có TTL trên Redis**: Key DataProtection lưu trên Redis không có thời gian hết hạn (TTL) mặc định.

3. **Ảnh hưởng khi mất key**:
   - Nếu key bị mất trong Redis: Người dùng sẽ bị đăng xuất, các token/cookie cũ không thể giải mã.
   - Hệ thống sẽ tự tạo key mới và tiếp tục hoạt động, nhưng mọi phiên hiện tại sẽ bị mất.

4. **Ảnh hưởng khi mất kết nối Redis**:
   - Nếu app đang chạy: Không ảnh hưởng ngay lập tức vì key đã được cache trong bộ nhớ.
   - Nếu app restart khi Redis mất kết nối: Sẽ gặp lỗi vì không thể tải key.

5. **Số lượng key trên Redis**:
   - Với 5 máy chủ IIS chạy cùng một ứng dụng: Chỉ có một bộ key dùng chung trên Redis.
   - Số lượng key không phụ thuộc vào số instance, mà phụ thuộc vào số lần rotation (thường 90 ngày/lần).

## 6. Khuyến nghị triển khai

1. **Backup Redis** định kỳ để tránh mất key.
2. **Đảm bảo Redis HA** (High Availability) để tránh mất kết nối.
3. **Đặt ApplicationName giống nhau** trên mọi instance để đảm bảo dùng chung key.
4. **Theo dõi logs** liên quan đến DataProtection để phát hiện vấn đề sớm.
5. **Bảo mật Redis** bằng password và network ACL để tránh truy cập trái phép.

## 7. Hiệu năng

- DataProtection chỉ truy cập Redis khi:
  - Khởi động ứng dụng
  - Rotation key (mỗi 90 ngày)
  - Không truy cập Redis cho mỗi request
- Key được cache trong bộ nhớ, tác động hiệu năng rất nhỏ
