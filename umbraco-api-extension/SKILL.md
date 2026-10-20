---
name: umbraco-api-extension
description: Kỹ năng viết custom API, tạo dashboard, property editors, event hooks, quản lý package/extensions trong Umbraco CMS mới nhất.
type: project
---

# Skill: API, Extension & Customization trong Umbraco CMS

**Mục tiêu:**
- Tạo API custom bằng ASP.NET Core trong dự án Umbraco
- Xây dựng back‑office dashboard, property editor tùy chỉnh
- Đăng ký event hooks, Composer, Surface Controller
- Quản lý packages, cài đặt, cập nhật, phát hành

## Đối tượng
- Developer chuyên sâu, plugin author, back‑office engineer

---

## Các bước thực hiện

1. **Tạo dự án API trong solution**
   - Mở solution Umbraco, thêm project Class Library (`dotnet new classlib -n MyUmbraco.Api`).
   - Thêm reference tới `Umbraco.Cms.Web.Common` và `Microsoft.AspNetCore.Mvc`.
   - Đăng ký controller trong `Startup.cs` hoặc `Program.cs`:
   ```csharp
   services.AddUmbraco(_ => _.AddWebApi());
   ```

2. **Viết Custom API Controller**
   - Kế thừa `UmbracoApiController` hoặc `ControllerBase`.
   - Đánh dấu `[Route("backoffice/api/[controller]")]`.
   ```csharp
   public class ContentApiController : UmbracoApiController {
       private readonly IContentService _contentService;
       public ContentApiController(IContentService contentService) => _contentService = contentService;

       [HttpGet("{id}")]
       public IActionResult Get(int id) {
           var content = _contentService.GetById(id);
           if (content == null) return NotFound();
           return Ok(new { id = content.Id, name = content.Name, url = content.Url() });
       }
   }
   ```

3. **Dashboard custom**
   - Tạo file `dashboard.config` trong `/App_Plugins/MyDashboard/`.
   - Định nghĩa `<dashboard>` với `<section alias="dashboard" view="/App_Plugins/MyDashboard/dashboard.html" />`.
   - Viết HTML/JS (Angular/React) để gọi API ở bước 2 và hiển thị dữ liệu.

4. **Property Editor tùy chỉnh**
   - Tạo thư mục `/App_Plugins/MyEditor/`.
   - Thêm `manifest.json`:
   ```json
   {
     "propertyEditors": [{
       "alias": "My.CustomEditor",
       "name": "Custom Editor",
       "editor": "/App_Plugins/MyEditor/editor.html",
       "icon": "icon-settings",
       "config": {}
     }]
   }
   ```
   - `editor.html` chứa UI (Angular/React) và sử dụng `Umbraco.Sys.ServerVariables` để gửi dữ liệu.

5. **Event Hooks (Composer, Component, MutatingContent)**
   - Tạo lớp Composer:
   ```csharp
   public class MyComposer : IComposer {
       public void Compose(IUmbracoBuilder builder) {
           builder.Components().Append<MyComponent>();
       }
   }
   public class MyComponent : IComponent {
       public void Initialize() => ContentService.Saving += OnSaving;
       private void OnSaving(IContentService sender, SaveEventArgs<IContent> e) {
           foreach (var c in e.SavedEntities) {
               // custom logic, e.g., auto‑populate fields
           }
       }
       public void Terminate() { }
   }
   ```

6. **Package / Extension management**
   - Tạo `package.manifest` trong `/App_Plugins/MyPackage/` với metadata (name, version, author).
   - Đóng gói `.zip` chứa `App_Plugins/`, `bin/` dlls, `manifest.json`.
   - Cài đặt qua Backoffice → Packages → Install local package.
   - Để cập nhật, tăng `version` trong manifest, upload lại.

7. **Testing & Debugging**
   - Use `ILogger<T>` in controllers/components.
   - Run site in Development mode → `dotnet run`.
   - Check logs at `App_Data/Logs/`.
   - Use Postman or Swagger (`services.AddSwaggerGen()`) to test API endpoints.

8. **Security considerations**
   - Protect API với `[Authorize]` hoặc custom permission (`UmbracoAuthorizeAttribute`).
   - Validate input, tránh SQL injection qua `IContentService`.
   - Kiểm tra anti‑forgery token khi sử dụng Surface Controllers.

---

## Tips & Best practice
- Sử dụng **Strongly‑typed Models Builder** cho content trong API để tránh magic strings.
- Tách dashboard UI ra thành SPA (Angular/React) độc lập, chỉ giao tiếp qua API.
- Đặt **cache** cho read‑only endpoints (`[ResponseCache(Duration = 300)]`).
- Đăng ký **custom permissions** trong `permissions.config` để kiểm soát ai có thể dùng API.
- Khi phát hành package, luôn cung cấp **README** và **upgrade notes**.
- Kiểm tra **unit tests** cho component, controller.
- Sử dụng **dependency injection** cho services, không tạo instance trực tiếp.

---
