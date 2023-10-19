# Entity Framework Core - Code First Approach - Fluent API

Ở phần [**Map Attributes**](/3_efcore7_code_first_approach_map_attributes.md), ta đã tìm hiểu về cách cấu hình các lớp thực thể bằng các attribute.

Trong EF Core, ngoài sử dụng Map Attributes, ta có thể sử dụng Fluent API để cấu hình các lớp thực thể. Và nội dung của phần này sẽ trình bày cách cấu hình bằng Fluent API. Các cấu hình bằng Fluent API sẽ ghi đè các cấu hình từ nguồn khác (ưu tiên cao nhất).

Fluent API là tính năng đã có từ EF và đến EF Core đã có một số thay đổi nhỏ.

## Phương thức OnModelCreating()

Để thực hiện cấu hình bằng Fluent API, ta sẽ ghi đè lại phương thức `OnModelCreating()` trong lớp Context. 
Phương thức này nhận tham số kiểu `ModelBuilder`, là lớp sẽ cung cấp các phương thức để cấu hình các lớp 
thực thể.

```cs
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // fluent api configurations ...
    }
```

## Chuẩn bị các lớp thực thể

Trong phần này, ta sẽ khai báo 2 lớp `Employee` và `Department` để thực hành như sau:

```cs
    public class Employee
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public double Salary { get; set; }

        public int DepartmentId { get; set; }
        public Department Department { get; set; }
    }

    public class Department
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public ICollection<Employee> Employees { get; set; }
    }
```

## Cấu hình thuộc tính

Để cấu hình cho một thuộc tính, ta cần xác định lớp thực thể chứa thuộc tính cần cấu hình.

Để chỉ định lớp thực thể, ta sử dụng phương thức `Entity<TEntity>()`, với `TEntity` là lớp thực thể đang muốn gọi với 2 cú pháp thường dùng:

```cs
    Entity<TEntity>()

    Entity<TEntity>(Action<EntityTypeBuilder<TEntity>> buildAction)
```

**Ví dụ:**

```cs
    modelBuilder.Entity<Employee>()...

    modelBuilder.Entity<Department>(department =>
    {
        // configurations ...
    });
```
Trong đó, *`modelBuilder`* là đối tượng kiểu `ModelBuilder`.

Phương thức `Entity<TEntity>()` trả về kiểu `EntityTypeBuilder<TEntity>` cung cấp các phương thức truy 
cần thiết cho quá trình cấu hình ở mức thực thể (sẽ được đề cập ở phần sau để cấu hình ở mức thực thể).

Để gọi đến thuộc tính, ta sử dụng phương thức `Property()` với cú pháp:

```cs
    Property(string propertyName)

    Property(Type propertyType, string propertyName)

    Property<TProperty>(string propertyName)

    Property<TProperty>(Expression<Func<TEntity,TProperty>> propertyExpression)
```

Trong đó:
* *`propertyName`*: tên của thuộc tính cần gọi, nếu không tồn tại thì thuộc tính mới với tên chỉ định sẽ được thêm vào. Có thể sử dụng toán tử `nameof()` khi chỉ định cho tham số này.

* *`type`*: kiểu dữ liệu của thuộc tính. Có thể dùng toán tử `typeof()` khi chỉ định cho tham số này.

* *`TProperty`*: kiểu dữ liệu của thuộc tính, dùng thay thế cho tham số *`type`*.

* *`propertyExpression`*: một biểu thức Lambda dùng để chỉ ra thuộc tính muốn gọi.

**Ví dụ:**

```cs
    modelBuilder.Entity<Employee>().Property("Name")... // dùng chuỗi chỉ định tên
    modelBuilder.Entity<Employee>().Property(nameof(Employee.Name))... // dùng nameof()

    modelBuilder.Entity<Employee>().Property(typeof(string), "Name")...

    modelBuilder.Entity<Employee>().Property<string>("Name")...

    // có thể bỏ qua khai báo kiểu TProperty khi dùng với biểu thức Lambda
    modelBuilder.Entity<Employee>().Property<string>(emp => emp.Name)...
    modelBuilder.Entity<Employee>().Property(emp => emp.Name)... // bỏ qua <string>
```

Phương thức `Property()` trả về kiểu `PropertyBuilder` cung cấp các phương thức dùng để cấu hình ở mức thuộc tính.

Các phương thức thường dùng của lớp `PropertyBuilder`:

| Phương thức | Mô tả | Attribute tương đương |
| --- | --- | --- |
| `IsRequired(bool required = true)` | Ràng buộc NOT NULL | `[Required]` |
| `HasColumnName(string? name)` | Chỉ định tên cột cho thuộc tính | `[Column]` |
| `HasColumnType(string? typeName)` | Chỉ định kiểu cho cột | `[Column]` |
| `HasDefaultValue(object? value)` | Chỉ định giá trị mặc định cho cột |  |
| `HasMaxLength(int maxLength)` | Chỉ định độ dài tối đa của mảng và chuỗi | `[MaxLength]` hoặc `[StringLength]` |
| `ValueGeneratedOnAdd()` | Tạo giá trị tự động khi thêm | `[DatabaseGenerated`] |
| `IsUnicode(bool unicode = true)` | Xác định cột có nhận giá trị chuỗi Unicode hay không | `[Unicode]` |
| `HasPrecision(int precision, int scale)` | Chỉ định độ chính xác và làm tròn | `[Precision]` |
| `UseIdentityColumn(int seed, int increment = 1)` | Xác định thuộc tính sử dụng tính năng `IDENTITY` trong SQL Server. | `[DatabaseGenerated]` |


> Xem thêm các phương thức cấu hình khác tại [**PropertyBuilder**](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.metadata.builders.propertybuilder?view=efcore-7.0).

Các phương thức của lớp `PropertyBuilder` đều trả về chính kiểu `PropertyBuilder`, do đó ta có thể gọi liên tiếp, nối chuỗi các phương thức cấu hình.

**Ví dụ:**

```cs
    modelBuilder.Entity<Employee>().Property(e => e.Name)
        .IsRequired()
        .HasColumnName("EmpName")
        .HasColumnType("nvarchar(50)");

    modelBuilder.Entity<Employee>().Property(e => e.Salary)
        .HasColumnType("decimal(10, 0)")
        .HasDefaultValue(1);
```
Trong trường hợp có nhiều đoạn mã cấu hình cho một thực thể, ta có thể gom nhóm bằng cách dùng tham số `Action<EntityTypeBuilder<TEntity>>` của phương thức `Entity<TEntity>()`.

**Ví dụ:**

```cs
    modelBuilder.Entity<Employee>(employee =>
    {
        employee.Property(e => e.Name)
            .IsRequired()
            .HasColumnName("EmpName")
            .HasColumnType("nvarchar(50)");

        employee.Property(e => e.Salary)
            .HasColumnType("decimal(10, 0)")
            .HasDefaultValue(1);

    });
```

## Cấu hình mức thực thể

Để cấu hình mức thực thể, ta sẽ sử dụng các phương thức của lớp `EntityTypeBuilder<TEntity>`, hay sử
dụng cùng với phương thức `Entity<TEntity>()`.

Các phương thức thường dùng của lớp `EntityTypeBuilder<TEntity>`:

<table>
    <tr>
        <th> Phương thức </th>
        <th> Mô tả </th>
        <th> Attribute tương tự </th>
    </tr>
    <tr>
        <td>
            <code>HasKey(param string[] propertyNames)</code> <br />
            <code>HasKey(Expression<Func<TEntity, object?>> keyExpression)</code>
        </td>
        <td>
            Chỉ định (các) khóa chính của thực thể
        </td>
        <td>
            <code>[Key]</code> hoặc <code>[PrimaryKey]</code>
        </td>
    </tr>
    <tr>
        <td>
            <code>Ignore(string propertyName)</code> <br />
            <code>Ignore(Expression<Func<TEntity, object?>> propertyExpression)</code>
        </td>
        <td>
            Bỏ qua ánh xạ đối với một thuộc tính
        </td>
        <td>
            <code>[NotMapped]</code>
        </td>
    </tr>
</table>

> Xem thêm các phương thức khác tại [**EntityTypeBuilder**](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.metadata.builders.entitytypebuilder?view=efcore-7.0).


Để xác định nhiều khóa chính với tham số *`keyExpression`*, ta có thể trả về đối tượng ẩn danh gồm nhiều thuộc tính làm khóa chính.

**Ví dụ:**

```cs
    modelBuilder.Entity<MyEntity>().HasKey("Key1", "Key2");

    // hoặc

    modelBuilder.Entity<MyEntity>().HasKey(e => new { e.Key1, e.Key2 });

    // nếu sử dụng tham số Action<EntityTypeBuilder<TEntity>>
    modelBuilder.Entity<MyEntity>(entity =>
    {
        entity.HasKey(e => new { e.Key1, e.Key2 });
    });
```

## Cấu hình mối quan hệ

Xem lại nội dung về [**Relationship**](/2_efcore7_code_first_approach_relationship.md) để xem cách triển khai các mối quan hệ cho lớp 
thực thể.

Thông thường, các mối quan hệ sẽ được cấu hình tự động bởi EF Core đối với các trường hợp tuân thủ theo 
các quy tắc định sẵn (hay còn gọi là Convention). Xem chi tiết ở nội dung về [**Convention**](/7_efcore7_code_first_approach_conventions.md).

Nội dung bên dưới sẽ trình bày cách cấu hình các mối quan hệ không áp dụng Convention.

### Cấu hình quan hệ 1 - 1

Để thực hành trên mối quan hệ 1 – 1, ta sẽ xây dựng lớp `Person` và `Address` như sau:

```cs
    public class Person {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime? DateOfBirth { get; set; }

        public int AddressId { get; set; } // foreign key property
        public Address? Address { get; set; } // reference navigation
    }

    public class Address {
        public int Id { get; set; }
        public string Street { get; set; }
        public string Province { get; set; }
        public string District { get; set; }

        public Person? Person { get; set; } // reference navigation
    }
```

Để cấu hình mối quan hệ 1 – 1, ta sẽ sử dụng phương thức `HasOne<TRelatedEntity>()`, `WithOne()` và `HasForeignKey<TDependentEntity>()`.

Phương thức `HasOne<TRelatedEntity>()` dùng để chỉ định Reference Navigation của thực thể hiện tại (đang 
xét với phương thức `Entity<TEntity>()`) tham chiếu đến thực thể còn lại của mối quan hệ. 

Ta có thể dùng một trong 2 cú pháp sau của phương thức này:

```cs
    HasOne<TRelatedEntity>(Expression<Func<TEntity,TRelatedEntity?>>? navigationExpression = default)

    HasOne<TRelatedEntity>(string? navigationName)
```

Ta thường sử dụng cú pháp đầu tiên và dùng biểu thức Lambda để chỉ định Reference Navigation.

**Ví dụ:**

```cs
    modelBuilder.Entity<Address>()
        .HasOne<Person>(address => address.Person) ... // chỉ định reference navigation

    // hoặc có thể bỏ qua khai báo tham số kiểu 'Person'
    modelBuilder.Entity<Address>()
        .HasOne(address => address.Person) ...
```

Phương thức `HasOne<TRelatedEntity>()` trả về kiểu dữ liệu `ReferenceNavigationBuilder<TEntity, TRelatedEntity>`.

Sau khi gọi phương thức `HasOne<TRelatedEntity>()`, ta sẽ gọi tiếp phương thức `WithOne()` để gọi ra Reference Navigation của lớp thực thể còn lại của mối quan hệ bằng 1 trong 2 cú pháp sau:

```cs
    WithOne(Expression<Func<TRelatedEntity,TEntity?>>? navigationExpression)

    WithOne(string? navigationName = default);
```

**Ví dụ:**

```cs
    modelBuilder.Entity<Address>()
        .HasOne<Person>(address => address.Person)
    .WithOne(person => person.Address);
```
Phương thức `WithOne()` trả về kiểu dữ liệu `ReferenceReferenceBuilder<TEntity,TRelatedEntity>`.

Sau cùng, gọi phương thức `HasForeignKey<TDependentEntity>()` của lớp để chỉ ra thuộc tính khóa ngoại 
(Foreign Key Property) ở kiểu phụ thuộc – kiểu có khóa ngoại.

Cú pháp:

```cs
    HasForeignKey<TDependentEntity>(Expression<Func<TDependentEntity, object?>> foreignKeyExp)

    HasForeignKey<TDependentEntity>(params string[] foreignKeyPropertyNames)
```

**Ví dụ:**

```cs
    modelBuilder.Entity<Address>()
        .HasOne<Person>(address => address.Person) // reference navigation của lớp 'Address'
        .WithOne(person => person.Address) // reference navigation của lớp 'Person'
        .HasForeignKey<Person>(person => person.AddressId); // foreign key property
```

Nếu có nhiều thuộc tính khóa ngoại, biểu thức Lambda có thể trả về đối tượng ẩn danh gồm nhiều thuộc 
tính làm khóa ngoại, ví dụ `x => new { x.FK1, x.FK2 }`.

Phương thức `HasForeignKey()` không bắt buộc phải gọi để xác định thuộc tính khóa ngoại, nhưng EF 
Core sẽ cố gắng tìm kiếm thuộc tính khóa ngoại theo Convention. Một số trường hợp EF Core có thể sẽ
không tìm thấy hoặc không đúng với mong muốn.

Ở các ví dụ trên, ta cấu hình bắt đầu với lớp thực thể cha `Address`. Ta cũng có thể bắt đầu bằng thực thể có 
khóa ngoại `Person` như sau:

```cs
    modelBuilder.Entity<Person>()
        .HasOne<Address>(person => person.Address) // reference navigation của lớp 'Person'
        .WithOne(address => address.Person) // reference navigation của lớp 'Address'
        .HasForeignKey<Person>(person => person.AddressId); // foreign key property
```

Tóm lại, ta có thể cấu hình bắt đầu từ cả 2 lớp thực thể, nhưng thường bắt đầu từ lớp thực thể cha.

Trong trường hợp lớp thực thể không chứa thuộc tính khóa ngoại, ta cũng có thể tạo ra thuộc tính khóa 
ngoại mới bằng cú pháp thứ 2 `HasForeignKey<TDependentEntity>(params string[])`. Khi chỉ định tên thuộc 
tính khóa ngoại **không tồn tại**, EF Core sẽ tự tạo ra thuộc tính khóa ngoại mới với tên được cung cấp.

**Ví dụ:**

```cs
    // giả sử thuộc tính AddressId không có trong lớp 'Person'
    modelBuilder.Entity<Address>()
        .HasOne<Person>(address => address.Person)
        .WithOne(person => person.Address)
        .HasForeignKey<Person>("AddressId"); // foreign key property mới
```

### Cấu hình quan hệ 1 - N

Ta khai báo lớp `Teacher` và `Student` như sau để thực hành nội dung này:

```cs
    public class Student
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime? DateOfBirth { get; set; }

        public int TeacherId { get; set; } // foreign key property
        public Teacher? Teacher { get; set; } // reference navigation
    }

    public class Teacher {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Subject { get; set; }

        public ICollection<Student> Students { get; set; } // collection navigation
}
```

Để cấu hình mối quan hệ 1 – 1, ta sẽ sử dụng phương thức `HasOne<TRelatedEntity>()`, `WithMany()` và `HasForeignKey<TDependentEntity>()`.

Trong đó, phương thức `HasOne<TRelatedEntity>()` được dùng bắt đầu với lớp thực thể con – lớp chứa khóa ngoại.

**Ví dụ:**
```cs
    modelBuilder.Entity<Student>
        .HasOne<Teacher>(student => student.Teacher) ... // reference navigation của lớp 'Student'
```

Theo sau phương thức `HasOne<TRelatedEntity>()` là `WithMany()` để chỉ ra tập hợp các thực thể con từ lớp 
thực thể cha với cú pháp:

```cs
    WithMany(Expression<Func<TRelatedEntity, IEnumerable<TEntity>?>>? navigationExpression)
```

**Ví dụ:**

```cs
    modelBuilder.Entity<Student>
        .HasOne<Teacher>(student => student.Teacher)
        .WithMany(teacher => teacher.Students) ...
```

Phương thức trên trả về kiểu dữ liệu `ReferenceCollectionBuilder<TPrincipalEntity,TDependentEntity>`.

Cuối cùng gọi phương thức `HasForeignKey()` để chỉ ra thuộc tính khóa ngoại từ lớp thực thể con với cú 
pháp:

```cs
    HasForeignKey(Expression<Func<TDependentEntity, object?>> foreignKeyExpression)
    HasForeignKey (params string[] foreignKeyPropertyNames)
```

**Ví dụ:**

```cs
    modelBuilder.Entity<Student>
        .HasOne<Teacher>(student => student.Teacher)
        .WithMany(teacher => teacher.Students)
        .HasForeignKey(student => student.TeacherId);
```

Nếu có nhiều thuộc tính khóa ngoại, biểu thức Lambda có thể trả về đối tượng ẩn danh gồm nhiều thuộc 
tính làm khóa ngoại, ví dụ `x => new { x.FK1, x.FK2 }`.

Trong trường hợp lớp thực thể không chứa thuộc tính khóa ngoại, ta cũng có thể tạo ra thuộc tính khóa 
ngoại mới bằng cú pháp thứ 2 `HasForeignKey(params string[])`. Khi chỉ định tên thuộc tính khóa ngoại 
không tồn tại, EF Core sẽ tự tạo ra thuộc tính khóa ngoại mới với tên được cung cấp.

**Ví dụ:**

```cs
    // giả sử lớp 'Student' chưa có thuộc tính khóa ngoại 'TeacherId'
    modelBuilder.Entity<Student>
        .HasOne<Teacher>(student => student.Teacher)
        .WithMany(teacher => teacher.Students)
        .HasForeignKey("TeacherId");
```

Ta cũng có thể cấu hình bắt đầu từ lớp thực thể cha (lớp chứa tập hợp) bằng phương thức 
`HasMany<TRelatedEntity>()` và `WithOne()` thay cho phương thức `HasOne<TRelatedEntity>()` và `WithMany()`.

```cs
    HasMany<TRelatedEntity>(Expression<Func<TEntity,IEnumerable<TRelatedEntity>?>>? navigationExp)
    HasMany<TRelatedEntity> (string? navigationName)
```

Phương thức trên trả về kiểu `CollectionNavigationBuilder<TEntity,TRelatedEntity>`. Phương thức 
`WithOne()` dùng để chỉ định Reference Navigation cho lớp thực thể con với cú pháp:

```cs
    WithOne(Expression<Func<TRelatedEntity,TEntity?>>? navigationExpression)
```

**Ví dụ:**

```cs
    modelBuilder.Entity<Teacher>
        .HasMany<Student>(teacher => teacher.Students)
        .WithOne(student => student.Teacher)
        .HasForeignKey(student => student.TeacherId);
```

### Cấu hình quan hệ N - N

Ở nội dung này, ta sẽ sử dụng cách triển khai mối quan hệ N – N cơ bản nhất trong EF Core mà không sử
dụng lớp thực thể trung gian (hay **Join Entity Type**). Vì nếu sử dụng lớp thực thể trung gian, ta sẽ chỉ cấu 
hình 2 mối quan hệ 1 – N (Xem lại phần [**Cấu hình quan hệ 1 – N**](#cấu-hình-quan-hệ-1---N)).

Khi không sử dụng lớp trung gian, một bảng trung gian sẽ được tạo bởi EF Core đến CSDL gồm 2 trường 
là khóa của 2 bảng liên quan. Nếu như bảng trung gian cần lưu trữ một số thông tin khác ngoài 2 khóa, ta 
nên khai báo lớp trung gian.

Khai báo 2 lớp thực thể `Orders` và `Product` để thực hành mối quan hệ N – N như sau:

```cs
    public class Orders
    {
        public int Id { get; set; }
        public DateTime? DateOfOrder { get; set; }

        public ICollection<Product> Products { get; set; } // collection navigation
    }

    public class Product {
        public int Id { get; set; }
        public string Name { get; set; }
        public double Price { get; set; }

        public ICollection<Order> Orders { get; set; } // collection navigation
```

Để cấu hình mối quan hệ N – N không có lớp trung gian, ta sử dụng phương thức 
`HasMany<TRelatedEntity>()` và theo sau là phương thức `WithMany()`.

**Ví dụ:**

```cs
    modelBuilder.Entity<Orders>
        .HasMany<Product>(order => order.Products)
        .WithMany(product => product.Orders);
```

Để cấu hình đầy đủ và tường minh cho bảng trung gian sẽ được tạo ra, ta có thể tiếp tục sử dụng phương 
thức `UseEntity()`.

Xem cách sử dụng phương thức `UseEntity()` tại [**UseEntity**](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.metadata.builders.collectioncollectionbuilder.usingentity?view=efcore-7.0).

