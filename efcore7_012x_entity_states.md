# Entity Framework Core 7.0 - Phần mở rộng - Các trạng thái của thực thể

## Vòng đời của đối tượng `DbContext` - DbContext lifetime

Vòng đời của một đối tượng `DbContext` bắt đầu từ lúc được khởi tạo đến khi được giải phóng (dispose).

> Nội dung tài liệu đôi khi sẽ sử dụng thuật ngữ "đối tượng context" tương ứng với đối tượng `DbContext`.

Đối tượng `DbContext` được thiết kế dành cho single unit-of-work, vì vậy mà vòng đời của đối tượng này thông thường rất ngắn.

> Single unit-of-work có thể hiểu các thực thể được theo dõi bởi đối tượng `DbContext` sẽ được commit hoặc rollback cùng nhau như một giao dịch duy nhất sau khi gọi `SaveChanges()` hoặc `SaveChangesAsync()`. Mỗi đối tượng `DbContext` chỉ quản lý một giao dịch duy nhất.

Một giao dịch điển hình khi sử dụng Entity Framework Core (EF Core) thường diễn ra như sau:

- Tạo đối tượng `DbContext`.

- Theo dõi (các) thực thể liên quan dến đối tượng context.

- Thực hiện các thay đổi trên (các) thực thể.

- Gọi `SaveChanges()` hoặc `SaveChangesAsync()`. EF Core xác định các thay đổi và lưu chúng vào cơ sở dữ liệu.

- Đối tượng `DbContext` được giải phóng.

Đối tượng context nên có thời gian sống ngắn (short-lived) và được huỷ để giúp các tài nguyên được giải phóng, tránh memory leak (rò rỉ bộ nhớ).

Trong ASP.NET Core khi cơ chế DI (Dependency Injection) được hỗ trợ, đối tượng context được DI Container quản lý từ khởi tạo đến huỷ đối tượng. 

Phương thức `AddDbContext()` mặc định đăng ký dưới dạng scope service, vì vậy context sẽ tự động huỷ sau mỗi request. Trong quá trình thực hiện request, đối tượng context có thể chia sẻ đến nhiều phương thức khác nhau trong các thiết kế nhiều lớp, như thiết kế N-tier.


> [!Warning]
> `DbContext` là một đối tượng **_not thread-safe_** và không nên được chia sẻ giữa nhiều luồng. Hãy chắc rằng sử dụng `await` trước khi tiếp tục gọi đến đối tượng context, hoặc tạo một instance mới nếu muốn tương tác trên nhiều luồng.
>
> :x: Không nên (thực ra sẽ gây ra lỗi):
>
> ```ts
> var blogAsync = dbContext.Blogs.FindAsync(1);
> var postAsync = dbContext.Posts.FirstOrDefaultAsync(p => p.BlogId = 1);
> var blog = await blogAsync;
> var post = await postAsync;
>
> /*
> System.InvalidOperationException:
>   A second operation was started on this context instance before a previous operation completed.
>   This is usually caused by different threads concurrently using the same instance of DbContext
> */
> ```
>
> :white_check_mark: Gọi `await` trước khi tiếp tục sử dụng `DbContext` instance:
>
> ```ts
> var blog = await dbContext.Blogs.FindAsync(1);
> var post = await dbContext.Posts.FindAsync(1);
> ```

Tuy có thể chia sẻ đối tượng `DbContext` trong nhiều ngữ cảnh, ta cần quan tâm đến các trạng thái theo dõi của thực thể liên quan đến đối tượng.

## Các trạng thái của thực thể - Entity States

Các trạng thái mà thực thể có thể có được đại diện bởi enum [**EntityState **](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.entitystate?view=efcore-9.0) gồm các giá trị sau:

- `Detached`: thực thể không được theo dõi (track) bởi đối tượng `DbContext`.

- `Unchanged`: thực thể chưa thực hiện thay đổi nào (các giá trị từ thuộc tính không thay đổi so với các trường trong cơ sở dữ liệu).

- `Deleted`: thực thể có tồn tại trong cơ sở dữ liệu, nhưng bị đánh dấu đã xoá khi gọi `SaveChanges()` hoặc `SaveChangesAsync()`.

- `Modified`: thực thể đã có thay đổi và sẽ được cập nhật sau khi gọi `SaveChanges()` hoặc `SaveChangesAsync()`.

- `Added`: là thực thể mới chưa được insert vào cơ sở dữ liệu và sẽ được thêm sau khi gọi `SaveChanges()` hoặc `SaveChangesAsync()`.

EF Core theo dõi các thay đổi ở mức độ thuộc tính. Ví dụ, nếu chỉ có 1 thuộc tính của thực thể bị thay đổi, EF Core chỉ thực hiện cập nhật giá trị đó cho trường tương ứng.















