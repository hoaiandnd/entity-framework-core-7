# Entity Framework Core 7 - Phần mở rộng - SQL Queries

> Nội dung của chương này sẽ đề cập cách thao tác mang tính trực tiếp hơn thay vì sử dụng LINQ.

Entity Framework Core cho phép đặt các câu lệnh truy vấn SQL xuống các cơ sở dữ liệu quan hệ (relational database). Sử dụng câu truy vấn SQL thuần sẽ hữu ích trong trường hợp câu truy vấn không thể biểu diễn bằng cú pháp LINQ, hoặc câu truy vấn mà EF Core sinh ra không hiệu quả hoặc tối ưu như mong đợi.

## Các truy vấn SQL cơ bản

Để bắt đầu sử dụng các lệnh truy vấn SQL trong EF Core, ta sẽ sử dụng phương thức `FromSql()`.

Cú pháp:

```ts
IQueryable<TEntity> FromSql<TEntity>(FormattableString sql)
```

**Ví dụ:**

```ts
var blogs = context.Blogs
    .FromSql($"SELECT * FROM Blogs")
    .ToList();
```

Một số lưu ý đối với phương thức `FromSql()`:

- Tham số `FormattableString` phải ở dạng chuỗi định dạng tổng hợp (Composite format string). Trong nhiều trường hợp thì nó là _interpolated string_.

- Phương thức chỉ gọi trực tiếp từ một `DbSet<TEntity>`.

> [!Warning]
> Câu truy vấn SQL thậm chí không cần liên quan đến `DbSet<TEntity>`:
>
> ```ts
>var blogs = context.Blogs
>   .FromSql($"SELECT * FROM Posts") // truy vấn `Post` trên `DbSet<Blog>`
>   .ToList();
> ```
>
> Tuy nhiên, kết quả trả về vẫn tuỳ thuộc vào kiểu `TEntity`. Ở ví dụ trên, dù kết quả thực tế trả về danh sách các bản ghi kiểu `Post`, nhưng trình biên dịch xem biến `blogs` là `IQueryable<Blog>`.

> [!Note]
> Phương thức `FromSql()` được giới thiệu ở Entity Framework Core 7. Ở các phiên bản trước đó hãy sử dụng `FromSqlInterpolated()` với cú pháp tương tự.

> [!Tip]
> Cả phương thức `FromSql()` và `FromSqlInterpolated()` đều an toàn để chống lại tấn công SQL Injection.







