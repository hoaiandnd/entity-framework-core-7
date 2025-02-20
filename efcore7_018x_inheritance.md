# Entity Framework Core 7 - Phần mở rộng - Inheritance

## Giới thiệu

Entity Framework có thể ánh xạ hệ thống phân cấp kiểu (type hierarchy) từ .NET sang cơ sở dữ liệu. Điều này cho phép lập trình viên có thể định nghĩa các lớp thực thể như thông thường, sử dụng các lớp cơ sở đến các lớp dẫn xuất. Có thể hiểu rằng ta hoàn toàn có thể triển khai tính kế thừa trên các thực thể sẽ ánh xạ vào cơ sở dữ liệu.

Mặc định (theo convention), EF Core không tự động kiểm tra đâu là kiểu cơ sở, đâu là kiểu dẫn xuất. Điều này yêu cầu lập trình viên phải chỉ định tường minh thông qua cấu hình. Ví dụ, việc chỉ ra kiểu cơ sở không khiến EF Core ngầm định bao gồm cả các kiểu con.

**Ví dụ:**

```cs
public class User
{
  public string Id { get; set; }
  public string Name { get; set; }
  public string Password { get; set; }
}

public class Staff : User
{
  public string Role { get; set; }
}

public class MyDbContext : DbContext
{
  public virtual DbSet<User> Users { get; set; } // không tự động bao gồm kiểu `Staff`
  public virtual DbSet<Staff> Staffs { get; set; } // chỉ định tường minh
}
```

Trong EF Core, có 3 loại cấu hình cho các thực thể có tính kế thừa:

- Table-per-hierarchy (TPH).

- Table-per-type (TPT).

- Table-per-concrete-type (TPC).





























