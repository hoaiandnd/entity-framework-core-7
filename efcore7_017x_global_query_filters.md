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

Trong các nội dung bên dưới, ta sẽ triển khai các filter cho các thực thể soft-delete với model sau:

```cs
class Blog
{
  public int Id { get; set; }
  public string Name { get; set; }
  public bool IsDeleted { get; set; } // Soft-delete property

  public List<Post> Posts { get; set; }
}
class Post
{
  public int Id { get; set; }
  public string Title { get; set; }
  public string Content { get; set; }
  public bool IsDeleted { get; set; } // Soft-delete property

  public string BlogId { get; set; }
  public Blog Blog { get; set; }
}
```

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
modelBuilder.Entity<Post>().HasQueryFilter(p => p.IsDeleted == false);
```

Biểu thức điều kiện được chỉ định cho phương thức `HasQueryFilter()` sẽ được tự động áp dụng cho bất kỳ truy vấn LINQ nào đến các thực thể được cấu hình.

> [!Note]
> Hiện tại, mỗi thực thể chỉ có thể cấu hình một query filter. Nếu có nhiều query filter, thực thể chỉ sử dụng filter được đăng ký cuối cùng. Để giải quyết vấn đề này, hãy cố gắng kết hợp các filter trong một lần cấu hình bằng cách áp dụng các toán tử logic như AND `&&` hoặc OR `||`.

## Sử dụng với thuộc tính điều hướng

Trong query filter, ta có thể gọi đến navigation property (reference navigation hoặc collection navigation) để đặt các điều kiện lọc.

**Ví dụ:**

```ts
// chỉ tải post khi blog chưa bị xoá
modelBuilder.Entity<Post>().HasQueryFilter(p => p.IsDeleted == false && p.Blog.IsDeleted == false); // gọi reference navigation
modelBuilder.Entity<Blog>().HasQueryFilter(b => b.Posts.Count > 0); // gọi collection navigation
```

> [!Caution]
> Đoạn mã ở ví dụ trên sẽ tạo ra vòng lặp (nếu áp dụng cả 2) do cứ mỗi truy vấn trên thực thể `Blog` đều yêu cầu lọc trên thực thể `Post` và ngược lại.
>
> Thật không may, EF Core hiện không phát hiện được vòng lặp (cycles) trong cấu hình từ global query filter. Vì thế hãy cẩn thận khi định nghĩa chúng. 

Khi sử dụng thuộc tính điều hướng trong filter, các truy vấn trên thực thể sẽ là truy vấn đệ quy (truy vấn gọi đến thực thể liên quan). 

EF Core sẽ gọi đến các thuộc tính điều hướng, query filter được định nghĩa ở thực thể liên quan cũng sẽ được áp dụng. Lúc này, điều kiện được áp dụng cho câu truy vấn sẽ là tổng hợp các điều kiện truy vấn của các thực thể liên quan được chỉ định cùng trong query filter.

**Ví dụ:**

```cs
modelBuilder.Entity<Blog>().HasQueryFilter(b => b.IsDeleted == false && b.Posts.Count > 0);
modelBuilder.Entity<Post>().HasQueryFilter(p => p.IsDeleted == false);

// truy vấn
var blog = context.Blog.ToList();
```

Ở ví dụ trên, khi truy vấn thực thể `Blog`, query filter của nó sẽ được áp dụng gồm `b.IsDeleted == false` và `b.Posts.Count > 0`.

Đối với điều kiện thứ 2, nó sẽ truy vấn đến thực thể `Post` và query filter của thực thể này cũng sẽ được áp dụng - `p.IsDeleted == false`. Do đó, điều kiện `b.Posts.Count > 0` sẽ tự động đếm các post chưa bị xoá.

Các query filter của thực thể liên quan cũng sẽ được áp dụng khi eager loading với phương thức `Include()`.

**Ví dụ:**

Giả sử query filter đang được cấu hình như sau:

```cs
modelBuilder.Entity<Blog>().HasQueryFilter(b => b.IsDeleted == false);
modelBuilder.Entity<Post>().HasQueryFilter(p => p.IsDeleted == false);
```

Các trường hợp sau sẽ áp dụng query filter từ navigation:

```cs
// chỉ tải các post chưa bị xoá
var blog = context.Blogs
  .Where(b => b.Id == 1)
  .Include(b => b.Posts) // áp dụng query filter của thực thể `Post`
  .FirstOrDefault();

// tương tự khi truy vấn theo chiều ngược lại
var post = context.Posts
  .Where(p => p.Id == 1)
  .Include(p => p.Blog) // áp dụng query filter của thực thể `Blog`
  .FirstOrDefault();
```

Đối với reference navigation, ta cần lưu ý:

- Nếu là _optional reference navigation_, khi thực thể liên quan bị lọc khỏi kết quả trả về, reference navigation sẽ nhận giá trị `null`.

- Nếu là _required reference navigation_, khi thực thể liên quan bị lọc khỏi kết quả trả về, chính thực thể con đang tải cũng sẽ bị loại khỏi kết quả trả về và nhận giá trị `null` (hệ quả của phép `INNER JOIN` - một phía thực thể không có dữ liệu phù hợp sẽ loại bản ghi phía còn lại ra khỏi kết quả trả về).

Ở trường hợp 2 trong ví dụ trên, `post` sẽ là `null` vì `Post.Blog` là một required reference navigation.

## Disabling filters

Global query filter sẽ áp dụng filter cho toàn bộ các truy vấn trên thực thể được cấu hình. Trong một số trường hợp muốn bỏ qua filter trên một số truy vấn LINQ, hãy sử dụng phương thức `IgnoreQueryFilters()`.

```cs
var blogs = await db.Blogs
    .Include(b => b.Posts)
    .IgnoreQueryFilters()
    .ToListAsync();
```

## Hạn chế của Global query filter - Limitations

Global query filter chỉ có thể định nghĩa ở kiểu gốc trong hệ thống phân cấp. Một cách đơn giản mà nói, nếu có nhiều thực thể kế thừa từ một lớp cơ sở chung thì query filter chỉ có thể định nghĩa trên kiểu cơ sở đó.

Khi query filter được định nghĩa trên lớp cơ sở, các truy vấn trên những lớp thực thể dẫn xuất (thực thể kế thừa và các cấp kế thừa sâu hơn) cũng sẽ được áp dụng query filter đó.

Tuy nhiên, đối với soft-delete, điều này là một thuận lợi. Nếu khai báo một lớp cơ sở, ví dụ `SoftDeleteEntity`, ta có thể cấu hình query filter một lần và sử dụng được cho các thực thể dẫn xuất.

**Ví dụ:**

- Khai báo một lớp cơ sở chung cho các thực thể cần chức năng soft-delete:

```cs
class SoftDeleteEntity
{
  public bool IsDeleted { get; set; }
}

public Blog : SoftDeleteEntity
{
  // các thuộc tính của blog
}

public Post : SoftDeleteEntity
{
  // các thuộc tính của post
}
```

- Ta chỉ cần cấu hình query filter trên lớp `SoftDeleteEntity`:

```cs
modelBuilder.Entity<SoftDeleteEntity>().HasQueryFilter(entity => entity.IsDeleted == false);
```



