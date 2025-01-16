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

- Tham số `FormattableString` phải ở dạng chuỗi định dạng tổng hợp (Composite format string). Trong nhiều trường hợp thì nó là _interpolated string_ (ở đây ta chỉ xét đối với C#).

- Phương thức chỉ gọi trực tiếp từ một `DbSet<TEntity>`.

> [!Warning]
> Câu truy vấn SQL thậm chí không cần liên quan đến `DbSet<TEntity>`:
>
> ```ts
> var blogs = context.Blogs
>   .FromSql($"SELECT * FROM Posts") // truy vấn `Post` trên `DbSet<Blog>`
>   .ToList();
> ```
>
> Tuy nhiên, kết quả trả về vẫn tuỳ thuộc vào kiểu `TEntity`. Ở ví dụ trên, dù kết quả thực tế trả về danh sách các bản ghi kiểu `Post`, nhưng trình biên dịch xem biến `blogs` là `IQueryable<Blog>`.

> [!Note]
> Phương thức `FromSql()` được giới thiệu ở Entity Framework Core 7. Ở các phiên bản trước đó hãy sử dụng `FromSqlInterpolated()` với cú pháp tương tự.

## Truyền tham số vào câu truy vấn

Tham số được đưa vào chuỗi truy vấn có cú pháp tương tự như chuỗi nội suy:

```cs
int id = 1;
var blogs = context.Blogs
    .FromSql($"SELECT * FROM Blogs WHERE id = {id}")
    .FirstOrDefault();
```

> Tất nhiên, số lượng tham số truyền vào là không giới hạn.

Cú pháp truyền tham số vào chuỗi truy vấn giống với chuỗi nội suy trong C#, nhưng các giá trị được cung cấp sẽ được "wrap" bởi đối tượng `DbParameter`, thứ giúp phương thức `FromSql()` an toàn với các cuộc tấn công SQL Injection, cũng như gửi giá trị một cách hiệu quả và chính xác đến cơ sở dữ liệu.

> [!Tip]
> Cả phương thức `FromSql()` và `FromSqlInterpolated()` đều an toàn để chống lại tấn công SQL Injection.

⚠️ Lưu ý rằng ta không thể tham số hoá các thành phần của schema (như tên bảng, tên cột, ...).

**Ví dụ:**

```cs
int id = 1;
string tableName = "Blogs";
string keyColumnName = "id";
// ERROR !!!
var blogs = context.Blogs
    .FromSql($"SELECT * FROM {tableName} WHERE {keyColumnName} = {id}")
    .FirstOrDefault();
```

Muốn tham số hoá các thành phần trên, ta cần phải sử dụng phương thức `FromSqlRaw()`. Với phương thức này, ta có thể đặt trực tiếp một chuỗi truy vấn SQL để thực thi.

```cs
int id = 1;
string tableName = "Blogs";
string keyColumnName = "id";
var blogs = context.Blogs
    .FromSqlRaw($"SELECT * FROM {tableName} WHERE {keyColumnName} = {id}")
    .FirstOrDefault();
```

> [!Warning]
> Khi sử dụng phương thức `FromSqlRaw()` ta cần phải đảm bảo rằng chuỗi SQL an toàn. Phương thức này sẽ thực thi trực tiếp chuỗi SQL được chỉ định, nếu giá trị tham số truyền vào không đảm bảo, ứng dụng có thể bị tấn công SQL Injection.





