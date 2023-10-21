# Entity Framework Core 7.0 - Cascade Delete

## Conventions for Cascading

Cascade là cơ chế lan truyền sự chuyển đổi trên thực thể đến các thực thể liên quan khác.

Các mối quan hệ trong EF Core được thể hiện thông qua khóa ngoại. Một thực thể chứa khóa ngoại được 
gọi là thực thể con (**Child Entity**) hay thực thể phụ thuộc (**Dependent Entity**) trong mối quan hệ. Giá trị của 
khóa ngoại phải trùng khớp với giá trị khóa của thực thể cha (**Parent Entity**) hay thực thể chủ thể (**Principal 
Entity**).

Khi thực thể cha bị xóa khỏi CSDL, giá trị khóa ngoại của (các) thực thể con sẽ không còn trùng khớp với 
bất kỳ thực thể cha nào trong bảng dữ liệu. Trong đa phần các CSDL, vi phạm ràng buộc tham chiếu (Referential constraint 
violation) sẽ xảy ra và ngăn cản việc xóa thực thể cha.
Trong trường hợp trên, ta có 2 lựa chọn giải quyết:
* Đưa tất cả các giá trị khóa ngoại về giá trị `NULL` – **Cascading Nulls**.

* Xóa tất cả các thực thể con liên quan – **Cascade Delete**.

Mặc định khi ta xóa thực thể cha ra khỏi CSDL thì:

* Nếu giữa thực thể cha và (các) thực thể con là **mối quan hệ ràng buộc** (Required relationship) thì (các) thực thể con sẽ bị **loại bỏ cùng với thực thể cha khi xóa**.

* Nếu giữa thực thể cha và (các) thực thể con là **mối quan hệ tùy chọn** (Optional relationship) thì
giá trị khóa ngoại của (các) thực thể con sẽ được gán với giá trị `NULL` khi xóa thực thể cha.

Trong đó:

* **Mối quan hệ ràng buộc** (Required relationship) được xác định khi thuộc tính khóa ngoại được khai báo với kiểu non-nullable.

* **Mối quan hệ tùy chọn** (Optional relationship) được xác định khi thuộc tính khóa ngoại được khai báo với kiểu cho phép giá trị `NULL` (kiểu tham chiếu, kiểu Nullable, ...).

**Ví dụ:**

```cs
    public int DepartmentId { get; set; } // required

    public int? DepartmentId { get; set; } // optional
```

> [!Important]
> **Cascade Delete** được cấu hình mặc định bởi [**Conventions**](./7_efcore7_code_first_approach_conventions.md). Nếu khai báo thuộc tính khóa ngoại 
với kiểu nullable (mối quan hệ tùy chọn) thì hành vi Cascade mặc định không còn hiệu lực nữa. Muốn sử
dụng **Cascading nulls**, ta phải tải (các) thực thể con bằng cơ chế **Eager Loading** với phương thức `Include()`
hoặc **Explicit Loading** với phương thức `Entry<TEntity>()`. Nếu không, một ngoại lệ sẽ được ném ra.

**Ví dụ:**

```cs
    var d = db.Departments.Include(d => d.Employees).Single(d => d.Id == 1); // tải cùng (các) thực thể con

    db.Departments.Remove(d);

    db.SaveChanges();
```

## Configure Cascade Behaviors

Hành vi Cascade có thể được cấu hình bằng cách sử dụng [**Fluent API**]() với phương thức `OnDelete()`.

**Ví dụ:**

```cs
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>()
                    .HasOne(e => e.Department)
                    .WithMany(d => d.Employees)
                    .OnDelete( /* DeleteBehavior */ );
    }
```

Trong đó, [`DeleteBehavior`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.deletebehavior?view=efcore-7.0) là một kiểu `enum` với 2 giá trị thường được dùng là `DeleteBehavior.Cascade` và 
`DeleteBehavior.SetNull`.

> [!Warning]
> Giá trị `DeleteBehavior.SetNull` sẽ ném ra ngoại lệ nếu như thuộc tính khóa ngoại không thể nhận giá trị `NULL` (Non-nullable Property).

>[!Note]
> Phương thức `OnDelete()` có thể gọi ra từ phương thức `WithOne()` hoặc `WithMany()`.

