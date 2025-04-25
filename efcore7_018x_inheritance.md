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

## Table-per-hierarchy configuration

Mặc định, EF sẽ ánh xạ sự kế thừa bằng cách sử dụng _table-per-hierarchy (TPH) pattern_. Về cơ bản, EF sẽ tạo ra một bảng duy nhất để lưu trữ hết các thuộc tính của các kiểu dẫn xuất (và kiểu cơ sở).

Để phân biệt cột nào của bảng nào, EF còn tạo thêm một cột dùng để phân biệt, ngầm định có tên là `Discriminator`.

![image](https://github.com/user-attachments/assets/ad858d7a-c5f9-4727-bb47-2117ac77ebb9)

> [!Important]
> Các cột dữ liệu có thể tự động đánh dấu nullable (nếu cần thiết) khi sử dụng cách cấu hình TPH. Ở ví dụ trên, cột `Role` được đánh dấu nullable vì `Role` không phải là thuộc tính của bảng `Users` thông thường.

`Discriminator` là tên thuộc tính được EF Core tự ngầm định tạo ra ở kiểu cơ sở (base type). Tuy nhiên ta có thể cấu hình lại nó bằng cách sử dụng phương thức `HasDiscriminator()`:

```ts
HasDiscriminator(string name, Type discriminatorType)
HasDiscriminator<TDiscriminator>(string name)
HasDiscriminator<TEntity,TDiscriminator>(Expression<Func<TEntity,TDiscriminator>> propertyExpression)
```

> [!Note]
> Cú pháp `HasDiscriminator<TDiscriminator>(string name)` phải chỉ định kiểu `TDiscriminator`.

Trong đó:

- `discriminatorType` (`Type`): Kiểu giá trị được lưu trong cột phân biệt.

- `TDiscriminator`: Kiểu giá trị được lưu trong cột phân biệt.

- `name`: tên cột phân biệt.

- `propertyExpression` (`Expression<Func<TEntity,TDiscriminator>>`): biểu thức chỉ ra thuộc tính được đặt làm cột phân biệt.

- `TEntity`: kiểu thực thể.

Các ví dụ bên dưới sẽ chỉ ra cách sử dụng cơ bản của các cú pháp trên:

```ts
modelBuilder.Entity<User>().HasDiscriminator("user_type", typeof(string));

modelBuilder.Entity<User>().HasDiscriminator<string>("userType");

// giả sử bảng `User` có thể thuộc tính `UserType`
modelBuilder.Entity<User>().HasDiscriminator<User, string>(user => user.UserType);
// hoặc bỏ qua đối số kiểu
modelBuilder.Entity<User>().HasDiscriminator(user => user.UserType);
```

Mặc định, giá trị của cột phân biệt (trong tài liệu này vẫn gọi là cột `Discriminator`) là tên các bảng trong hệ thống phân cấp sử dụng cấu hình TPH. Và ta hoàn toàn có thể chỉ định lại chúng bằng phương thức `HasValue()`:























