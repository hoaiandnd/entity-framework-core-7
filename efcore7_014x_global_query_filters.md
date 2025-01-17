# Global Query Filters

## Giới thiệu

Global query filters là các điều kiện truy vấn (query predicates) trong LINQ được áp dụng cho thực thể (thường đặt trong `OnModelCreating`). Các điều kiện truy vấn là các biểu thức luận lý thường được đặt vào mệnh đề `WHERE`. 

Global query filter sẽ cho phép EF Core tự động áp dụng các điều kiện truy vấn đến bất cứ truy vấn LINQ nào liên quan đến thực thể được cấu hình.

Tức là trong một kiểu thực thể, các truy vấn liên quan ta "luôn" (hoặc "thường xuyên") muốn lọc đi các dữ liệu không nên xuất hiện trong kết quả truy vấn, ta có thể đặt cho thực thể đó một "filter tự động".

Một trong các ứng dụng đơn giản nhất khi áp dụng global query filter là với các thực thể áp dụng **Soft-delete** (các thực thể sẽ chứa một thuộc tính - ví dụ như `IsDeleted` - để đánh dấu rằng thực thể này đã bị xoá nhưng không thực sự được xoá ở phía dữ liệu vật lý). Khi truy vấn trên các thực thể soft-delete, ta thường xuyên phải lọc đi các bản ghi "đã xoá", ví dụ:

```cs
var blogs = context.Blogs
  .Where(b => b.AuthorId == 1 && b.IsDeleted == false)
  .ToList();
```

Trong các nội dung bên dưới, ta sẽ sử dụng trường hợp Soft-delete để làm ví dụ.

## Cấu hình filter

Để đặt một query filter trên một kiểu thực thể, ta sẽ cấu hình trong phương thức `OnModelCreating()` và sử dụng phương thức `HasQueryFilter()`:

```ts
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
  modelBuilder.Entity<Entity>().HasQueryFilter(/* ... */);

  // hoặc
  modelBuilder.Entity<Entity>(entity =>
  {
    entity.HasQueryFilter(/* ... */);
    // các cấu hình khác ...  
  });
}
```

Phương thức `HasQueryFilter()` nhận vào một biểu thức luận lý `Expression<Func<TEntity, bool>>? filter` (tương tự biểu thức dùng cho `Where()`).

**Ví dụ:**

```ts
// tự động lọc ra các thực thể chưa bị xoá
modelBuilder.Entity<Blog>().HasQueryFilter(b => b.IsDeleted == false);
```

Biểu thức điều kiện được chỉ định cho phương thức `HasQueryFilter()` sẽ được tự động áp dụng cho bất kỳ truy vấn LINQ nào đến các thực thể được cấu hình.

> [!Note]
> Hiện tại, mỗi thực thể chỉ có thể cấu hình một query filter. Nếu có nhiều query filter, thực thể chỉ sử dụng filter được đăng ký cuối cùng. Để giải quyết vấn đề này, hãy cố gắng kết hợp các filter trong một lần cấu hình bằng cách áp dụng các toán tử logic như AND `&&` hoặc OR `||`.




