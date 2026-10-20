---
name: "umbraco-content-modeling"
description: "Kỹ năng tạo, cấu trúc, tối ưu Document Types, Media Types, Data Types, Compositions, Relations trong Umbraco CMS mới nhất."
---

# Skill: Content Modeling với Umbraco CMS

**Mục tiêu:**
- Thiết kế schema dữ liệu mạnh mẽ, dễ bảo trì.
- Quản lý toàn diện Document, Media, Data Types.
- Tối ưu structure/tái sử dụng fields (Compositions & Relations).
- Mapping dữ liệu đến UI Razor template an toàn (Models Builder).

## Phạm vi áp dụng
- Khởi tạo hoặc tối ưu site Umbraco mới/đang vận hành.
- Đội ngũ Dev, Architect phụ trách logic dữ liệu.

---

## Hướng dẫn thao tác

1. **Tạo Document Type**
   - Vào Backoffice > Settings > Document Types > Create new
   - Đặt tên, alias, chọn icon
   - Thêm fields (textbox, media picker, date, ...), nhóm fields thành tab
   - Đặt constraints (bắt buộc, unique, regex...)
   - Lưu và kiểm thử tạo node mới thuộc loại vừa dựng

2. **Media Types**
   - Settings > Media Types > New
   - Tạo structure custom cho types ảnh, file đặc biệt
   - Dùng riêng cho media nhiều thuộc tính hoặc logic phân loại

3. **Data Types**
   - Settings > Data Types > New
   - Tùy biến input control (dropdown, multiselect, rich text)
   - Tái sử dụng nhiều nơi

4. **Compositions**
   - Tạo groups fields dùng chung (SEO, Meta, Settings...)
   - Vào Document Type, tab Compositions > add group
   - Lưu ý tránh vòng lặp cấu trúc

5. **Relations**
   - Định nghĩa mối quan hệ parent-child hoặc liên kết tuỳ ý giữa các node
   - Sử dụng field Picker, MultiPicker, hoặc custom Relation API

6. **Mapping sang Template**
   - Sử dụng Models Builder để sinh model mạnh kiểu (strong-typed)
   - Razor View: @Model.[FieldAlias]
   - Xử lý null, đa ngữ, media liên kết rõ ràng

7. **Kiểm thử & versioning**
   - Tạo/Chỉnh/Sửa/Xoá node mẫu
   - Kiểm tra UI backend & frontend render đúng logic mới
   - Hệ thống lớn: quản lý Data Type qua code/source control

## Tips & Best practice
- Alias & tên rõ ràng, thống nhất naming toàn project
- Sử dụng Compositions cho nhóm thuộc tính lặp lại
- Data Types nên được tối ưu/tái sử dụng, tránh lặp
- Mapping Model khi đổi schema phải build lại Models Builder
- Field quan trọng: luôn bật Required hoặc validation rõ.
- Test đa locale, nhiều loại media, quan hệ phức tạp trước khi chính thức

## Khi nào dùng skill này
- Khởi tạo dự án mới
- Khi refactor/tối ưu lại luồng content lớn
- Khi có nhu cầu mở rộng loại nội dung hoặc media đặc biệt
