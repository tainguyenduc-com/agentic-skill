---
name: umbraco-templating
description: Kỹ năng xây dựng Razor View, Partial View, layout, multi‑language rendering và mapping dữ liệu trong Umbraco CMS mới nhất.
type: project
---

# Skill: Templating & Rendering trong Umbraco CMS

**Mục tiêu:**
- Tạo UI sạch, tái sử dụng bằng Razor và Partial Views
- Mapping dữ liệu từ Document Types sang mô hình strongly‑typed (Models Builder)
- Hỗ trợ đa ngôn ngữ, layout chung, media handling
- Tối ưu performance rendering, cache, fallback

## Đối tượng
- Front‑end developer, Full‑stack, UI/UX engineer

---

## Quy trình thực hiện

1. **Kích hoạt Models Builder**
   - Mở `appsettings.json` → `Umbraco:CMS:ModelsBuilder:ModelsMode = SourceCode`
   - Chạy `dotnet build` → các model class được sinh trong `Models` folder

2. **Tạo Razor View (Template)**
   - Thư mục `/Views/` → tạo file `.cshtml` cho mỗi Document Type
   - Đặt `@inherits Umbraco.Cms.Web.Common.Views.UmbracoViewPage<Models.{DocumentType}>`
   - Truy cập trường: `@Model.{Alias}`

3. **Partial Views & Sections**
   - Tạo `_Header.cshtml`, `_Footer.cshtml`, `_Nav.cshtml`…
   - Gọi `@Html.Partial("_Header")` trong layout chính
   - Sử dụng `@RenderSection("scripts", required: false)` để chèn script phụ

4. **Layout chung (Master Layout)**
   - File `_Layout.cshtml` chứa `<head>`, header, footer, `@RenderBody()`
   - Đặt `Layout = "~/Views/_Layout.cshtml"` trong mỗi view

5. **Multi‑language rendering**
   - Bật `CultureVariation` trên Document Types cần đa ngôn ngữ
   - Access `@Model.Value("title", CultureInfo.CurrentCulture)` hoặc `@Model.GetCulture("en-US")`
   - Khi cần fallback: `@Model.Value("title", "en-US")`

6. **Media handling**
   - Sử dụng `UmbracoHelper.Media(mediaId)` để lấy URL
   - Render `<img src="@mediaUrl" alt="@mediaAlt" />`
   - Đối với responsive images, dùng `<picture>` và `srcset`

7. **Querying content**
   - `UmbracoHelper.ContentAtRoot()` → lặp qua root nodes
   - `UmbracoHelper.ContentSingleAtXPath("//home")`
   - `UmbracoHelper.TypedContentAtRoot()` (legacy) → chuyển sang newer APIs
   - Sử dụng `IPublishedContentQuery` cho LINQ‑style queries

8. **Cache & Performance**
   - Enable OutputCache: `[OutputCache(Duration = 300, VaryByParam = "*")]`
   - Sử dụng `UmbracoCache` cho dữ liệu không thay đổi thường xuyên
   - Buffer partial view rendering khi cần

9. **Testing & Debug**
   - Đặt breakpoint trong view (`@{ Debugger.Break(); }`) để kiểm tra model
   - Sử dụng `UmbracoDebug` middleware để introspect dữ liệu
   - Kiểm tra các fallback ngôn ngữ bằng `CultureInfo.CurrentCulture`

## Tips & Best practice
- Luôn dùng strong‑typed Models (Models Builder) → compile‑time safety
- Tách layout và partials để tránh duplication
- Đặt `@model` explicit, không dùng `dynamic`
- Khi dùng media, tránh hard‑code path, luôn qua `UmbracoHelper`
- Kiểm tra responsive image sizes, lazy‑load để giảm payload
- Đối với multilingual, luôn cung cấp fallback cho mọi field
- Đánh giá cache hit‑rate, điều chỉnh `Duration` cho phù hợp

---
