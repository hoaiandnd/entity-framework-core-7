# Entity Framework Core 7.0 - Relationship

Với hướng tiếp cận Code-First trong EF Core, thao tác đầu tiên sẽ là khai báo các lớp thực thể (Entity Classes) 
dùng để tạo cơ sở dữ liệu tương ứng.

Lớp thực thể thực chất là các lớp đối tượng C# chứa các thuộc tính tương ứng với cột (trường dữ liệu - Field) trong SQL Server. Việc khai báo lớp đối tượng trong C# đã là một thao tác quá quen thuộc, vì vậy sẽ không được đề cập chi tiết ở đây. Thay vào đó, tài liệu này sẽ cung cấp các nội dung về Relationship – cách biểu diễn mối quan hệ giữa 2 thực thể.

Giữa 2 thực thể thường sẽ xảy ra 1 trong 3 loại mối quan hệ sau:

* Quan hệ 1 – 1.
* Quan hệ 1 – N.
* Quan hệ N – N.

Với các RDBMS[^2], các quan hệ được biểu diễn bằng cách sử dụng khóa ngoại (Foreign Key) tham chiếu đến khóa chính (Primary Key) trong đa phần các trường hợp. Tuy nhiên, trong các ngôn ngữ hướng đối tượng như C#, ta sử dụng các Navigation Property để biểu diễn mối quan hệ giữa 2 lớp.

[^2]: Relational Database Management System (Hệ quản trị cơ sở dữ liệu quan hệ)

**Navigation Property** (Thuộc tính điều hướng) là các thuộc tính không chứa giá trị mà chỉ chứa một tham chiếu đến (các) thực thể liên quan ở vế còn lại của quan hệ.

Trong các tài liệu từ Microsoft, Navigation Property thường được chia thành 2 loại:

* **Reference Navigation**: dùng để chỉ những Navigation Property tham chiếu đến 1 thực thể liên quan (trong mối quan hệ 1 - 1 và 1 - N).
* **Collection Navigation**: dành cho các Navigation Property dạng tập hợp, danh sách các thực thể liên quan (trong mối quan hệ 1 - N và N - N).

Các Navigation Property sẽ thể hiện rõ ràng mối quan hệ giữa 2 thực thể, cung cấp phương thức truy cập dữ liệu của thực thể liên quan một cách nhanh nhất, không cần thông qua truy vấn.

## Mối quan hệ 1 - 1

Mối quan hệ 1 – 1 được sử dụng để mô tả rằng với một thực thể này chỉ liên kết với tối đa một thực thể khác. Với quan hệ 1 – 1, thuộc tính điều hướng chỉ là một thuộc tính đơn tham chiếu đến một thực thể
khác. Với thiết kế thuần hướng đối tượng, ta có thể không cần phải định nghĩa thuộc tính đại diện cho khóa ngoại, chỉ cần Navigation Property.

Mối quan hệ 1 - 1 trong OOP thường được thiết kế như sau:
```csharp
    // principal (parent)
    public class A
    {
        // properties ...
        public B B_obj { get; set; }
    }

    // dependent (child)
    public class B
    {
        // properties ...
        public A A_obj { get; set; }
    }
```

**Ví dụ:**
```csharp
    public class Person
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }

        public Contact? Contact { get; set; } // reference navigation
    }
    public class Contact {
        public int Id { get; set; }
        public string Address { get; set; }
        public string Phone { get; set; }
        public string Email { get; set; }

        public Person Person { get; set; } // reference navigation
    }
```

Bên cạnh đó ta cũng có thể khai báo thêm thuộc tính khóa ngoại cho lớp thực thể con (tương ứng với bảng sẽ chứa khóa ngoại). Không chỉ mối quan hệ 1 - 1 mà mối quan hệ 1 - N cũng có thể khai báo thêm thuộc tính khóa ngoại, gọi là **Foreign Key Property**.

**Ví dụ:**
```csharp
    public class Contact
    {
        // other properties ...
        public int PersonId { get; set; } // foreign key property
        public Person Person { get; set; } // reference navigation
    }
```

* Nếu Reference Navigation **của thực thể con** có kiểu `Nullable<T>` (hay `T?`) thì được gọi là **Optional Reference Navigation** (điều hướng tùy chọn). Điều này đồng nghĩa với việc khóa ngoại khi được tạo ra hoặc xóa thực thể cha được phép gán giá trị `NULL`. Chú ý rằng điều hướng tùy chọn chỉ có thể chỉ định cùng với thuộc tính khóa ngoại nullable (nếu có).
```csharp
    // principal (parent)
    public class A
    {
        public B? B_obj { get; set; }
    }

    // dependent (child)
    public class B
    {
        public type? A_Id { get; set; } // optional foreign key property
        public A? A_obj { get; set; } // optional reference navigation
    }
```
* Ngược lại thì được gọi là **Required Reference Navigation** (điều hướng bắt buộc). Khi tạo mới thực thể con phải chỉ ra thực thể cha.
```csharp
    // principal (parent)
    public class A
    {
        public B? B_obj { get; set; }
    }

    // dependent (child)
    public class B
    {
        public A A_obj { get; set; } // required reference navigation
    }
```
Để bắt buộc Navigation Property của thực thể con phải tham chiếu đến thực thể cha và không thể để rỗng (NULL), ta có thể sử dụng giá trị `null!`.

**Ví dụ:**
```csharp
    public Person Person { get; set; } = null!;
```

> Xem thêm các trường hợp khác trong thiết kế quan hệ 1 – 1 trong EF Core tại [One-to-One](https://learn.microsoft.com/en-us/ef/core/modeling/relationships/one-to-one).


## Mối quan hệ 1 - N

Mối quan hệ 1 – N được dùng để khi một thực thể này liên kết với nhiều (hoặc không) thực thể khác. Trong thiết kế lớp thực thể, Navigation Property của thực thể cha sẽ chứa một tập hợp các thực thể con liên quan (còn được gọi là Collection Navigation). Một số kiểu dữ liệu thường dùng cho Collection Navigation như `ICollection<T>` (thường thấy ở các lớp thực thể được tạo theo hướng tiếp cận Database First), `IEnumerable<T>`, `List<T>`, ...

Mối quan hệ 1 - N trong OOP thường được thiết kế như sau:

```csharp
    // dependent (child)
    public class A
    {
        // properties ...
        public B B_obj { get; set; }
    }

    // principal (parent)
    public class B
    {
        // properties ...
        public List<A> A_list { get; set; }
    }
```

**Ví dụ:**

```csharp
    public class Student
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public Teacher? Teacher { get; set; } // optional reference navigation
    }
    public class Teacher
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Salary { get; set; }

        // contain 'Student' entities
        public ICollection<Student> Students { get; set; } = new List<Student>(); // collection navigation
    }
```

Thiết kế mối quan hệ 1 - N có thể khai báo thêm thuộc tính khóa ngoại.

**Ví dụ:**
```csharp
    public class Student
    {
        // other propreties ...
        public int? TeacherId { get; set; } // foreign key property
        public Teacher? Teacher { get; set; } // reference navigation
    }
```

Mối quan hệ 1 - N cũng có các khái niệm về **Required Reference Navigation** và **Optional Reference Navigation** tương tự [mối quan hệ 1 - 1](#mối-quan-hệ-1---1).


> Xem thêm các trường hợp khác trong thiết kế quan hệ 1 – N tại: [One-to-Many](https://learn.microsoft.com/en-us/ef/core/modeling/relationships/one-to-many).

## Mối quan hệ N - N

Mối quan hệ N – N được dùng mô tả rằng nhiều thực thể này có thể liên kết với nhiều (hoặc không) thực thể khác. Có thể hiểu là một thực thể A có thể liên kết với nhiều thực thể B và ngược lại.

Mối quan hệ N – N khác biệt so với 2 loại quan hệ trước. Trong cơ sở dữ liệu, mối quan hệ này không thể biểu diễn chỉ bằng khóa ngoại và thường phải có thêm bảng trung gian (Join Table). 
Ta cũng có thể khai báo một lớp trung gian (Join Entity Type) để hiện thực mối quan hệ N – N.

**Ví dụ:**
```cs
    public class Orders
    {
        // other properties ...
        public ICollection<OrderDetail> Details { get; set; }
    }
    public class Product
    {
        // other properties ...
        public ICollection<OrderDetail> Details { get; set; }
    }

    // join entity type
    public class OrderDetail 
    { 
        // other properties ...
        public Order? Order { get; set; }
        public Product? Product { get; set; }
    }
```

Khi sử dụng kiểu trung gian, mối quan hệ N – N sẽ trở thành 2 quan hệ 1 – N giữa kiểu trung gian và 2 kiểu chính.

Tuy nhiên, EF Core có thể giấu đi kiểu trung gian và có thể kiểm soát ngầm định. Vì vậy ta không cần định nghĩa thêm kiểu trung gian và dùng các quan hệ 1 – N.

**Ví dụ:**
```cs
    public class Orders
    {
        // other properties ...
        public ICollection<Product> Products { get; set; }
    }
    public class Product
    {
        // other properties ...
        public ICollection<Orders> Orders { get; set; }
    }
```

Việc thiết kế lớp thực thể và quan hệ như trên là một trong các thiết kế cơ bản nhất trong mối quan hệ N – N.

> Xem thêm các trường hợp khác trong thiết kế quan hệ N – N tại: [Many-to-Many](https://learn.microsoft.com/en-us/ef/core/modeling/relationships/many-to-many).
