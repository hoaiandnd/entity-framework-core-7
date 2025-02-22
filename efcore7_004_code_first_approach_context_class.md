# Entity Framework Core 7.0 - Code First Approach - Lớp Context

Mục nội dung:

- [**Phương thức khởi tạo**](#phương-thức-khởi-tạo)

- [**Dữ liệu của bảng**](#dữ-liệu-của-bảng)

- [**Phương thức `OnConfiguring()`**](#phương-thức-onconfiguring)

    - [**Phương thức `UseSqlServer()`**](#phương-thức-usesqlserver)
 
    - [**Các phương thức khác**](#các-phương-thức-khác)

- [**Phương thức `OnModelCreating()`**](#phương-thức-onmodelcreating)

---

Lớp Context là lớp đối tượng kế thừa lớp `DbContext` đại diện cho phiên làm việc với CSDL. Lớp Context chứa 
các thuộc tính `DbSet<TEntity>` chứa dữ liệu của bảng tương ứng và các phương thức để cấu hình một số
thông tin về CSDL và lớp thực thể.

Do chứa dữ liệu của các bảng thông qua thuộc tính, nên khi làm việc với dữ liệu ta cần tạo ra đối tượng 
Context trước tiên.

## Phương thức khởi tạo

Thông thường, lớp Context được khai báo với 2 phương thức khởi tạo:

* Phương thức khởi tạo mặc định (không tham số).

* Phương thức khởi tạo nhận tham số kiểu `DbContextOptions<TContext>`. Trong đó, `TContext` là lớp Context đang định nghĩa.

**Ví dụ:**

```ts
public class QLNhanSuContext : DbContext
{
    public QLNhanSuContext() 
    {}
    public QLNhanSuContext(DbContextOptions<QLNhanSuContext> options) : base(options)
    {}
}
```

Phương thức khởi tạo nhận tham số `DbContextOptions<TContext>` thường được dùng với ASP.NET Core cùng với DI (Dependency Injection).

```ts
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));
```

## Dữ liệu của bảng

Dữ liệu của bảng đại diện bởi các thuộc tính kiểu `Microsoft.EntityFrameworkCore.DbSet<TEntity>`. Các thuộc tính này thường được đặt tên với quy tắc `<table_name>` + `s`.

Kiểu `DbSet<TEntity>` triển khai `IEnumerable` và `IQueryable`, vì vậy có thể dùng LINQ để truy vấn dữ liệu.

Trong một số trường hợp, các thuộc tính `DbSet<TEntity>` thường được khai báo với từ khóa `virtual`, cho phép tạo ra các lớp dẫn xuất ảo (Proxy). Các lớp dẫn xuất ảo này được sử dụng để theo dõi các thay đổi của đối tượng thực thể và cập nhật cơ sở dữ liệu tương ứng. 

Nếu không sử dụng từ khóa `virtual`, các lớp dẫn xuất ảo này sẽ không được tạo ra và Entity Framework sẽ không theo dõi được các thay đổi của đối tượng thực thể.

> [!Note]
> Tuy nhiên, từ khóa `virtual` có thể gây ra một số vấn đề về hiệu suất, vì vậy cần cân nhắc khi sử dụng.

**Ví dụ:**

```ts
public class QLNhanSuContext : DbContext {
    public QLNhanSuContext() 
    {}

    public QLNhanSuContext(DbContextOptions<QLNhanSuContext> options) : base(options)
    {}

    public virtual DbSet<NhanVien> NhanViens { get; set; }
    public virtual DbSet<PhongBan> PhongBans { get; set; }
}
```

## Phương thức `OnConfiguring()`

Phương thức `OnConfiguring()` cùng với tham số `DbContextOptionsBuilder`, ta có thể cấu hình các thông 
tin cần thiết (như Database Provider, sensitive-data logging, configure warning, ...) cho đối tượng Context
bằng việc ghi đè nó với từ khóa chỉ định truy cập `protected`.

```ts
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    // configuration ...
}
```

Nội dung bên dưới sẽ liệt kê một số phương thức và thuộc tính thường dùng của tham số kiểu `DbContextOptionsBuilder`.

#### Phương thức `UseSqlServer()`

Mỗi đối tượng Context phải được cấu hình với một và chỉ một Database Provider. Để cấu hình Database 
Provider, ta sử dụng phương thức `Use...()`. Phương thức `Use...()` là phương thức mở rộng có tên cụ thể
tùy thuộc vào gói Database Provider đã cài đặt.

Ở đây ta muốn thao tác với SQL Server, vì vậy sẽ sử dụng phương thức `UseSqlServer()`.

Cú pháp:

```ts
UseSqlServer(string connectionString)
```

**Ví dụ:**

```cs
public class QLNhanSuContext : DbContext
{
    // các thành phần khác ...

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"Data Source=.\sqlexpress;Initial Catalog=QLNhanSu;Integrated Security=True;Encrypt=False");
    }
}
```

> [!Tip]
> Thành phần `Encrypt=False` có thể bỏ qua trong chuỗi kết nối nếu sử dụng Entity Framework Core 6.0 trở xuống. Khuyến khích sử dụng tham số này trong môi trường phát triển.

#### Các phương thức khác

Một số phương thức thường dùng để cấu hình với kiểu `DbContextOptionsBuilder`:

| Phương thức | Mô tả |
| --- | --- |
| `UseQueryTrackingBehavior()` | Cấu hình hành vi theo dõi (Track) thực thể trên câu truy vấn (Query) |
| `LogTo()` | Lấy EF Core Log |
| `EnableSensitiveDataLogging()` | Cho phép log các thông tin nhạy cảm trong thông báo ngoại lệ hoặc logging |
| `EnableDetailedErrors()` | Cho phép lỗi truy vấn chi tiết hơn |
| `ConfigureWarnings()` | Bỏ qua hoặc ném lỗi |
| `UseLazyLoadingProxies()` | Sử dụng các proxy động để lazy-load |

>  Xem thêm các phương thức khác tại [**DbContextOptionsBuilder Class**](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder?view=efcore-7.0).

Các phương thức trên sẽ được đề cập chi tiết lần lượt trong các tài liệu sau, ở những nội dung liên quan.

## Phương thức `OnModelCreating()`

Phương thức `OnModelCreating()` được dùng để cấu hình các bảng dữ liệu sinh ra từ lớp thực thể về các 
ràng buộc, khóa chính, khóa ngoại, mối quan hệ, ...

Nội dung về phương thức `OnModelCreating()` sẽ được trình bày trong nội dung về [**Fluent API**](/6_efcore7_code_first_approach_fluent_api.md).


