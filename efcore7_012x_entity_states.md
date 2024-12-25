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

Các trạng thái mà thực thể có thể có được đại diện bởi enum [**EntityState**](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.entitystate?view=efcore-9.0) gồm các giá trị sau:

- `Detached`: thực thể không được theo dõi (track) bởi đối tượng `DbContext`.

- `Unchanged`: thực thể chưa thực hiện thay đổi nào (các giá trị từ thuộc tính không thay đổi so với các trường trong cơ sở dữ liệu) **_kể từ lúc chúng được truy vấn từ cơ sở dữ liệu_**. Các thực thể được truy vấn từ cơ sở dữ liệu đều bắt đầu với trạng thái này.

- `Deleted`: thực thể có tồn tại trong cơ sở dữ liệu, nhưng bị đánh dấu đã xoá khi gọi `SaveChanges()` hoặc `SaveChangesAsync()`.

- `Modified`: thực thể đã có thay đổi **_kể từ lúc chúng được truy vấn từ cơ sở dữ liệu_** và sẽ được cập nhật sau khi gọi `SaveChanges()` hoặc `SaveChangesAsync()`.

- `Added`: là thực thể mới chưa được insert vào cơ sở dữ liệu và sẽ được thêm sau khi gọi `SaveChanges()` hoặc `SaveChangesAsync()`.

EF Core theo dõi các thay đổi ở mức độ thuộc tính. Ví dụ, nếu chỉ có 1 thuộc tính của thực thể bị thay đổi, EF Core chỉ thực hiện cập nhật giá trị đó cho trường tương ứng.

### Theo dõi thực thể - Entity Tracking

Mặc định, các thực thể được trả về thông qua câu truy vấn đều được EF Core theo dõi. 

> [!Note]
> Trường hợp EF Core không theo dõi đã đề cập trong chương [**Querying data - Tracking vs. No-tracking queries**](https://github.com/hoaiandnd/entity-framework-core-7/blob/main/8_efcore7_querying_data.md#tracking-vs-no-tracking-queries):
>
> - Sử dụng phương thức `AsNoTracking()` hoặc cấu hình toàn cục `UseQueryTrackingBehavior()`.
>
> - Dữ liệu trả về không chứa thực thể hoàn chỉnh.

Để xem được trạng thái hiện tại của một thực thể đối với đối tượng `DbContext` hiện tại, ta sử dụng phương thức `Entry<TEntity>()` để truy cập đến `EntityState` thông qua thuộc tính `State`.

**Ví dụ:**

```ts
var db = new MyDbContext();

var blog = await db.Blog.FindAsync(2); // theo dõi thực thể
var blogState = db.Entry(blog).State;

var newDb = new MyDbContext(); // tạo một đối tượng DbContext khác
var newBlogState = newDb.Entry(blog).State; // thực thể `blog` không được theo dõi bởi đối tượng context `newDb`

Console.WriteLine(blogState.ToString()); // "Unchanged"
Console.WriteLine(newBlogState.ToString()); // "Detached"
```

Ngay cả khi thực thể được trả về và sử dụng ở một tầm vưc khác, miễn đối tượng `DbContext` vẫn được chia sẻ đến, trạng thái của thực thể vẫn có thể xác định được thông qua đối tượng context đó.

```ts
public class BlogService
{
  private readonly MyDbContext _db = new();
  public Blog? GetBlogById(int id)
  {
    return _db.Blog.Find(id);
  }
  public string GetState(Blog blog)
  {
    return _db.Entry(blog).State.ToString();
  }
}

public class Program
{
  static void Main()
  {
    var blogService = new BlogService();
    var blog = blogService.GetBlog(1); // thực thể được trả về
    var state = blogService.GetState(blog); // sử dụng cùng một `DbContext` instance
    Console.WriteLine(state); // "Unchanged"
  }
}
```

Do chia sẻ cùng một instance, một số design pattern thường tách thành các phương thức nhỏ phục vụ cho các thao tác khác nhau trong vòng đời của `DbContext`, ví dụ:

```ts
public class BlogService
{
  private readonly MyDbContext _db = new();
  public void Add(Blog blog)
  {
    _db.Blog.Add(blog);
  }
  public void Commit()
  {
    _db.SaveChanges();
  }
}

public class Program
{
  static void Main()
  {
    var blogService = new BlogService();
    var newBlog = new Blog()
    {
      Name = "New blog"  
    };
    blogService.Add(newBlog);
    blogService.Commit();
  }
}
```

> Đoạn mã trên chỉ nhằm mục đích ví dụ, trong các tình huống cụ thể sẽ có cách triển khai riêng.

### Trạng thái `Added`

Một thực thể cần được theo dõi với trạng thái `Added` trước khi được thêm vào bởi `SaveChanges()` hoặc `SaveChangesAsync()`.

Để đối tượng `DbContext` bắt đầu theo dõi một thực thể mới tạo ra với trạng thái `Added`, cách phổ biến nhất là sử dụng các phương thức từ `DbSet<TEntity>` như `Add()`, `AddRange()`, `AddAsync()` hoặc `AddRangeAsync()`.

```ts
dbContext.Blogs.Add(newBlog);
dbContext.Posts.AddRange(postList);
```

Các phương thức kể trên sẽ bắt đầu đặt theo dõi trên (các) thực thể chỉ định và đặt trạng thái của chúng thành `Added`. Sau đó là chờ được commit.

Bên cạnh đó, các phương thức `Add...()` kể trên cũng được định nghĩa trong đối tượng `DbContext` với cú pháp và cách dùng tương tự.

```ts
dbContext.Add(newBlog);
dbContext.AddRange(postList);
```

> [!Tip]
> Trong các ngữ cảnh trừu tượng, tổng quát (generic), ta hoàn toàn có thể truy cập đến `DbSet<TEntity>` cụ thể bằng phương thức `DbContext.Set<TEntity>()`.
>
> **Ví dụ:**
>
> ```ts
> public class GenericRepository<TEntity>
> {
>   private readonly MyDbContext _db;
>   public GenericRepository(MyDbContext db) // constructor dependency injection
>   {
>     _db = db
>   }
> 
>   public void Add(TEntity entity)
>   {
>      db.Set<TEntity>.Add(entity);
>   }
> }
> ```

Các phương thức trên không chỉ theo dõi và đặt trạng thái `Added` trên thực thể được chỉ định mà còn thực hiện điều đó trên các thực thể liên quan (related entities).

**Ví dụ:**

```ts
var newBlog = new Blog()
{
  Name = "Blog 1",
  Posts = [
    new Post() { Name = "Post 1" },
    new Post() { Name = "Post 2" },
    new Post() { Name = "Post 3" },
    new Post() { Name = "Post 4" }
  ]
};
dbContext.Add(newBlog); // đặt trạng thái `Added` trên `newBlog` và cả các thực thể con
```

> [!Note]
> Sau khi gọi `SaveChanges()` hoặc `SaveChangesAsync()`, trạng thái của thực thể sẽ chuyển thành `Unchanged`, vì chúng đã có trong cơ sở dữ liệu.

### Trạng thái `Modified`

Trạng thái `Modified` được ghi nhận cho các thực thể đã được truy vấn từ cơ sở dữ liệu và bắt đầu từ trạng thái `Unchanged`. Khi thực thể có trạng thái `Modified`, EF Core sẽ thực hiện cập nhật sau khi gọi `SaveChanges()` và `SaveChangesAsync()`.

#### Thay đổi trực tiếp thuộc tính

Trong đa phần các trường hợp, trạng thái này được đánh dấu vào các thực thể được theo dõi khi có thay đổi trên ít nhất 1 thuộc tính.

```ts
var blog = dbContext.Blogs.Find(5); // trạng thái `Unchanged`
blog.Name = "Updated name"; // trạng thái `Modified`
dbContext.SaveChanges(); // lưu thay đổi
```

Các thay đổi cũng được tính trên các thuộc tính điều hướng (navigation property).

> Thuộc tính điều hướng là tên gọi chung cho 2 loại thuộc tính: **Reference navigation** và **Collection navigation**.

```ts
var post = dbContext.Posts.Include(p => p.Blog).FirstOrDefault(p => p.Id == 2);

// thay đổi trên reference navigation
post.Blog.Name = "Updated blog name";

// thay đổi trên collection navigation
post.Comments.Add(new Comment() { /* ... */ });

db.SaveChanges();
```

Các thực thể con được thêm vào khi thay đổi thực thể hiện tại sẽ có trạng thái `Added`.

#### Sử dụng các phương thức cập nhật

Cả đối tượng `DbContext` và `DbSet<TEntity>` đều có 2 phương thức dùng để bắt đầu theo dõi và đánh dấu trạng thái của thực thể là `Modified`, chính là `Update()` và `UpdateRange()`.

> [!Tip]
> Phương thức `Update()` và `UpdateRange()` sẽ bắt đầu theo dõi thực thể, tức là chúng chấp nhận các thực thể chưa được theo dõi bởi đối tượng context - trạng thái `Detached` (có thể là thực thể vừa được tạo với `new` hoặc dữ liệu từ một nguồn khác với database).

> [!Note]
> Một số lưu ý khi sử dụng các phương thức `Update()` và `UpdateRange()`:
>
> - Nếu khoá chính của thực thể là **khoá tự động tạo** (generated key) - thường dùng nhất là khoá tự động tăng (khoá `IDENTITY` trong SQL Server) thì:
>
>   - Nếu thực thể có chỉ định khoá chính thì khi gọi `Update()` hoặc `UpdateRange()`, trạng thái sẽ là `Modified`.
>  
>   ```ts
>     var blog = new Blog() { Id = 1, Name = "Updated blog name" }; // có chỉ định khoá chính `Id`
>     dbContext.Update(blog); // trạng thái `Modified`
>     db.SaveChanges(); // UPDATE
>   ```
>
>   - Ngược lại, thực thể sẽ được theo dõi trong trạng thái `Added`. Khi gọi `SaveChanges()` hoặc `SaveChangesAsync()` sẽ tạo câu `INSERT` thay vì `UPDATE`.
>
>   ```ts
>     var blog = new Blog() { Name = "New blog name" }; // không có đặt khoá chính
>     dbContext.Update(blog); // trạng thái `Added`
>     db.SaveChanges(); // INSERT
>   ```
>   
> - Nếu khoá chính không phải khoá tự động tạo, trạng thái của thực thể luôn là `Modified`.

#### Gán trạng thái thủ công

Phương thức `Update()` và `UpdateRange()` sẽ thực hiện 2 thao tác:

- Theo dõi thực thể.

- Đánh dấu trạng thái cho thực thể.

Bên cạnh đó, ta có thể thực hiện 2 thao tác trên một cách thủ công:

- Để theo dõi một thực thể mới (trạng thái `Detached`), ta sử dụng phương thức `DbContext.Attach()`.

- Thay đổi `dbContext.Entry(entity).State`.

**Ví dụ:**

```ts
var blog = new Blog() { Id = 1 };
dbContext.Attach(blog); // theo dõi thực thể - trạng thái `Unchanged`
dbContext.Entry(blog).State = EntityState.Modified; // thay đổi trạng thái
dbContext.SaveChanges();
```

> [!Warning]
> Thực thể cần chỉ định khoá chính khi thay đổi trạng thái thành `Modified`.

Ngoài ra, ta có thể chỉ định thuộc tính `IsModified` thành `true` để đánh dấu 1 thuộc tính cũng như toàn bộ thực thể là `Modified`.

```ts
var blog = new Blog() { Id = 1 };
dbContext.Attach(blog); // theo dõi thực thể - trạng thái `Unchanged`
dbContext.Entry(blog).Property(b => b.Name).IsModified = true; // thay đổi trạng thái
dbContext.SaveChanges();
```












