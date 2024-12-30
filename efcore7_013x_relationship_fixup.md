# Entity Frameowork Core 7.0 - Phần mở rộng - Relationship fixup

## Relationship fixup

Relationship fixup là quá trình EF Core giữ cho các thuộc tính điều hướng (navigation property) phù hợp với khoá ngoại (foreign key) và ngược lại. Bất kỳ thay đổi nào trên khoá ngoại hay thuộc tính điều hướng đều yêu cầu EF Core cập nhật thành phần còn lại cho tương ứng.

## Fixup by query

Quá trình fixup đầu tiên diễn ra trong quá trình truy vấn từ cơ sở dữ liệu. Cơ sở dữ liệu chỉ lưu trữ giá trị khoá ngoại, vì vậy EF Core cần dựa vào giá trị khoá ngoại đó xác định và khởi tạo phiên bản thực thể tương ứng, đặt reference navigation cho thực thể đó và đưa thực thể đó vào collection navigation.

**Ví dụ:**

```ts
var blogs = context.Blogs
    .Include(e => e.Posts)
    .ToList();
```

Với mỗi blog, EF Core sẽ tạo một `Blog` instance trước tiên. Sau đó đã tạo các `Blog` instance tương ứng với các bản ghi trong cơ sở dữ liệu, với mỗi post được tải lên, reference navigation của nó - `Post.Blog` - sẽ được trỏ đến blog tương ứng, đồng thời `Post` instance đó sẽ được thêm vào collection navigation của blog - `Blog.Posts`.



