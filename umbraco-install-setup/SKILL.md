---
name: umbraco-install-setup
description: Kỹ năng cài đặt, nâng cấp Umbraco CMS, khởi tạo dự án mới trên nền .NET, cấu hình môi trường & site.
type: project
---

# Skill: Cài đặt, setup và nâng cấp Umbraco CMS

**Mục tiêu:**
- Thiết lập môi trường phát triển và sản xuất chuẩn
- Cài đặt nhanh dự án Umbraco CMS mới qua CLI hoặc Visual Studio
- Nâng cấp phiên bản Umbraco an toàn, nắm các điểm chú ý khi upgrade

## Đối tượng áp dụng
- Developer, sysadmin, devops
- Dự án mới hoặc nâng cấp hệ thống cũ

---

## Hướng dẫn thao tác

1. **Chuẩn bị môi trường**
   - Yêu cầu: .NET SDK (latest LTS), SQL Server hoặc SQLite
   - Cài đặt Visual Studio hoặc VS Code (extension C#)
   - Update dotnet CLI: `dotnet --version`

2. **Tạo dự án Umbraco mới**
   - Mở terminal, chạy: `dotnet new umbraco -n MyUmbracoSite`
   - CD vào thư mục: `cd MyUmbracoSite`
   - Restore packages: `dotnet restore`
   - Chạy thử local: `dotnet run`
   - Truy cập `https://localhost:xxxx/umbraco` để setup admin user

3. **Cấu hình & kết nối database**
   - Trong appsettings.json: chỉnh ConnectionStrings phù hợp
   - Hỗ trợ SQL Server, Azure SQL, SQLite
   - Có thể chỉnh thêm các tùy chọn SMTP, Session, Logging

4. **Cài đặt qua Visual Studio**
   - Mở project, build & run như WebApp thường
   - Hỗ trợ publish trực tiếp lên IIS/Azure

5. **Nâng cấp phiên bản Umbraco**
   - Update NuGet packages: `dotnet add package Umbraco.Cms --version x.y.z`
   - Kiểm tra breaking changes tại https://our.umbraco.com/download/releases/ 
   - Chạy lệnh migrate nếu cần, backup DB trước mọi upgrade
   - Sau upgrade, chạy lại project, làm theo hướng dẫn migration trong backoffice nếu xuất hiện

6. **Các lưu ý production**
   - Đổi secret keys, không dùng giá trị mặc định
   - Cấu hình HTTPS, logging, backup định kỳ
   - Theo dõi release note, LTS/EOL policy

## Tips & Best practice
- Luôn backup code/DB trước mọi upgrade
- Test upgrade trên môi trường staging trước production
- Tránh sửa trực tiếp thư mục core của Umbraco
- Sử dụng version management cho NuGet packages
- Tách riêng file appsettings cho mỗi môi trường

---