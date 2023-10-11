# Entity Framework Core 7.0

* Tài liệu gốc: https://learn.microsoft.com/en-us/ef/core/
* Tải và cài đặt Visual Studio 2022 tại: https://visualstudio.microsoft.com/vs/
* Kiến thức yêu cầu:
    * LINQ To Objects
    * LINQ To Entities (LINQ To SQL) - *Không bắt buộc*


## Sơ lược về Entity Framework Core

***Entity Framework Core*** (EF Core) là một phiên bản mã nguồn mở, gọn nhẹ, dễ mở rộng và đa nền tảng trong công nghệ truy cập dữ liệu Entity Framework.

EF Core Là sự nâng cấp so với ***Entity Framework 6 (EF6)*** – phiên bản cuối cùng dành cho .NET Framework, sau khi .NET Core ra đời.

Gần như những gì cơ bản nhất của EF6 được giữ lại ở EF Core: 

* Là một ORM[^1] Framework (Object/Relational Mapping).
* Hỗ trợ truy vấn LINQ.

[^1]: ORM (Object/Relational Mapping) là một nền tảng ánh xạ cơ sở dữ liệu với các lớp đối tượng, cho phép lập trình viên chỉ cần thao tác với đối tượng mà không cần yêu cầu nhiều kỹ năng về SQL.

Tuy nhiên, vẫn có một số sự khác nhau giữa EF6 và EF Core. Xem chi tiết tại: https://learn.microsoft.com/vi-vn/ef/efcore-and-ef6/

Phiên bản EF Core được giới thiệu mới nhất là EF Core 7.0 vào tháng 11/2022 dành cho .NET 6.

EF Core hoạt động trên nhiều loại cơ sở dữ liệu như SQL Server, MySQL, PostgreSQL, ... dựa vào các [Database Providers](https://learn.microsoft.com/vi-vn/ef/core/providers/?tabs=dotnet-core-cli) (nhà cung cấp cơ sở dữ liệu).

EF Core hỗ trợ 2 hướng tiếp cận: *Code-first* và *Database-First*. Trong nội dung này, ta sẽ tìm hiểu cả 2 hướng tiếp cận này.

## Cài đặt các gói cần thiết

Để tạo ứng dụng .NET với Entity Framework Core, ta cần cài đặt 2 gói NuGet sau:
* Gói EF Core Database Provider tương ứng. Với SQL Server là `Microsoft.EntityFrameworkCore.SqlServer`.
* EF Core Tools: `Microsoft.EntityFrameworkCore.Tools`.

Tùy vào cơ sở dữ liệu muốn sử dụng mà gói Database Provider cần cài đặt sẽ khác nhau, vì vậy hãy tìm kiếm các [gói Database Provider](https://learn.microsoft.com/vi-vn/ef/core/providers/?tabs=dotnet-core-cli) tương ứng.

### Cài đặt bằng Package Manager Console - PMC

* Chọn **Tools > NuGet Package Manager > Package Manager Console** để mở PMC.
* Sử dụng lệnh `Install-Package` để cài đặt các gói NuGet mới nhất tìm thấy ở [NuGet Gallery](https://www.nuget.org/):
```console
    Install-Package Microsoft.EntityFrameworkCore.SqlServer
    Install-Package Microsoft.EntityFrameworkCore.Tools
```
* Nếu muốn tải các gói ở phiên bản cụ thể, hãy sử dụng tham số `-Version <version>`.

**Ví dụ:**
```console
    Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 6.0.10
```

### Cài đặt bằng Manage NuGet Packages

Có 3 cách để mở Manage NuGet Packages:
* Chọn **Tools > NuGet Package Manager > Manage NuGet Packages for Solution ...**.
* Right-click vào tên Solution, chọn **Manage NuGet Packages for Solution ...**.
* Right-click vào tên Project, chọn **Manage NuGet Packages ...**.

Chọn tab **Browse** và tìm kiếm và tải 2 gói được nêu ở trên.

**Gợi ý:** Nếu đã tải các gói từ NuGet về máy, hãy đưa (Copy hoặc Cut - Move) chúng vào thư mục `NuGet Packages` theo đường dẫn `C:\Program Files (x86)\Microsoft SDKs\NuGetPackages`.

## Hướng tiếp cận Database First

### Chuẩn bị cơ sở dữ liệu

Với hướng tiếp cận Database First, ta cần có sẵn cơ sở dữ liệu.

Ở đây, ta sử dụng cơ sở dữ liệu đơn giản như sau:

| NhanVien | PhongBan |
| :---: | :---: |
| **manv** |  **mapb** |
| tennv | tenpb |
| luong | |
| mapb | |

Đoạn mã sau sẽ tạo và cung cấp các dữ liệu giả lập:
```SQL
    CREATE DATABASE QLNhanSu
    GO
    USE QLNhanSu
    GO
    CREATE TABLE NhanVien (
        manv VARCHAR(10) PRIMARY KEY,
        tennv NVARCHAR(50),
        luong DECIMAL,
        mapb VARCHAR(10)
    )
    GO
    CREATE TABLE PhongBan (
        mapb VARCHAR(10) PRIMARY KEY,
        tenpb NVARCHAR(50)
    )
    GO
    ALTER TABLE NhanVien ADD CONSTRAINT FK_NV_PB FOREIGN KEY(mapb) REFERENCES PhongBan(mapb)
    GO

    -- dữ liệu giả lập cho phòng ban
    INSERT INTO PhongBan VALUES('PB001', 'Phòng ban 1')
    INSERT INTO PhongBan VALUES('PB002', 'Phòng ban 2')
    INSERT INTO PhongBan VALUES('PB003', 'Phòng ban 3')
    INSERT INTO PhongBan VALUES('PB004', 'Phòng ban 4')

    -- dữ liệu giả lập cho nhân viên
    INSERT INTO NhanVien VALUES('NV00001', N'Nguyễn Văn A', 15000, 'PB001')
    INSERT INTO NhanVien VALUES('NV00002', N'Lê Thị B', 14500, 'PB002')
    INSERT INTO NhanVien VALUES('NV00003', N'Trần Thị C', 15050, 'PB003')
    INSERT INTO NhanVien VALUES('NV00004', N'Võ Văn D', 12000, 'PB004')
    INSERT INTO NhanVien VALUES('NV00005', N'Nguyễn Thị E', 5000, 'PB001')
    INSERT INTO NhanVien VALUES('NV00006', N'Trần Văn F', 3500, 'PB001')
    INSERT INTO NhanVien VALUES('NV00007', N'Đoàn Thị G', 15100, 'PB003')
    INSERT INTO NhanVien VALUES('NV00008', N'Nguyễn Văn H', 1790, 'PB004')
    INSERT INTO NhanVien VALUES('NV00009', N'Nguyễn I', 2000, 'PB002')
    INSERT INTO NhanVien VALUES('NV00010', N'Trần Văn J', 25000, 'PB001')
    INSERT INTO NhanVien VALUES('NV00011', N'Nguyễn Ngọc K', 35600, 'PB003')
    INSERT INTO NhanVien VALUES('NV00012', N'Ngô Thị L', 15000, 'PB001')
    INSERT INTO NhanVien VALUES('NV00013', N'Nguyễn Toàn M', 17000, 'PB004')
```

### Kết nối CSDL

Do EF Core được tạo ra nhằm phục vụ cho hướng tiếp cận Code-first, nên hướng tiếp cận Database-first không được hỗ trợ nhiều và đặc biệt là không thể thao tác với giao diện như EF6.

Để tiếp cận theo hướng Database-first, ta sẽ sử dụng lệnh `Scaffold-DbContext` trong **Package Manager Console**.

Lệnh `Scaffold-DbContext` có nhiều tham số dùng để chỉ định các thông tin kết nối. Ở mức độ cơ bản, ta thường dùng các tham số sau:

| Tham số | Mô tả |
| --- | --- |
| `-Connection <string>` | Chuỗi kết nối đến CSDL cần kết nối. **Đây là tham số bắt buộc và là tham số vị trí** |
| `-Provider <string>` | Tên gói Database Provider đang dùng. **Đây là tham số bắt buộc và là tham số vị trí** |
| `-OutputDir <string>` | Thư mục / đường dẫn chứa các lớp thực thể khi tạo ra từ CSDL. Mặc định là đường dẫn gốc của Project |
| `-ContextDir <string>` | Thư mục / đường dẫn chứa lớp Context. Mặc định dùng chung với `-OutputDir` (nếu có) hoặc đường dẫn gốc của Project |

> Xem thêm các tham số khác tại [Scaffold-DbContext](https://learn.microsoft.com/en-us/ef/core/cli/powershell#scaffold-dbcontext).

---
#### Chuỗi kết nối (Connection string) trong Visual Studio 2022
---

* Chọn **Views > Server Explorer** hoặc sử dụng tổ hợp phím `Ctrl` + `Alt` + ``` ` ```.
   
* Right-click **Data Connection > Add Connection ...**.
     
* Chỉ định tên Server (Server name) và chọn Database. Có thể dùng lệnh `PRINT @@SERVERNAME` trong SQL Server để xem tên Server.
     
* Chọn **Advanced ...**, chuỗi kết nối được hiển thị phía cuối hộp thoại. Sao chép chuỗi nhận được và hủy các thao tác vừa thực hiện.

Với chuỗi kết nối nhận được, ta đã có thể dùng lệnh `Scaffold-DbContext` để kết nối với CSDL.

**Ví dụ:**
* Không chỉ định đường dẫn cho các lớp thực thể và lớp Context:
```console
   Scaffold-DbContext 'Data Source=.\sqlexpress;Initial Catalog=QLNhanSu;Integrated Security=True' Microsoft.EntityFrameworkCore.SqlServer
```
* Chỉ định đường dẫn cho các lớp thực thể và lớp Context
```console
   Scaffold-DbContext 'Data Source=.\sqlexpress;Initial Catalog=QLNhanSu;Integrated Security=True;' Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -ContextDir Data
```

---
#### Vấn đề với Entity Framework Core 7.0
---

Nếu sử dụng lệnh `Scaffold-DbContext` [phần trên](#chuỗi-kết-nối-connection-string-trong-visual-studio-2022), với EF Core 7.0 ta sẽ nhận về thông báo lỗi sau khi gọi lệnh `Scaffold-DbContext`.

> *A connection was successfully established with the server, but then an error occurred during the login process. (provider: SSL Provider, error: 0 - The certificate chain was issued by an authority that is not trusted.)*

Để giải quyết lỗi trên, ta có thể thực hiện 1 trong 3 cách sau đây:
* Cài đặt một chứng chỉ hợp lệ trên máy chủ. Cách làm này rất phức tạp vì cần một nhà cung cấp dịch vụ có thẩm quyền.

* Thêm `TrustServerCertificate=True` vào chuỗi kết nối.

* Thêm `Encrypt=False` vào chuỗi kết nối.

**Lưu ý:** Cách 2 và 3 chỉ nên thực hiện khi đang trong môi trường phát triển.

Để xem thêm các vấn đề liên quan, hãy xem [Certificate Issue](https://learn.microsoft.com/en-us/troubleshoot/sql/database-engine/connect/certificate-chain-not-trusted?tabs=odbc-driver-18x).

## Hướng tiếp cận Code First

### Relationship

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

Trong các tài liệu của Microsoft, Navigation Property thường được chia thành 2 loại:

* **Reference Navigation**: dùng để chỉ những Navigation Property tham chiếu đến 1 thực thể liên quan (trong mối quan hệ 1 - 1 và 1 - N).
* **Collection Navigation**: dành cho các Navigation Property dạng tập hợp, danh sách các thực thể liên quan (trong mối quan hệ 1 - N và N - N).

Các Navigation Property sẽ thể hiện rõ ràng mối quan hệ giữa 2 thực thể, cung cấp phương thức truy cập dữ liệu của thực thể liên quan một cách nhanh nhất, không cần thông qua truy vấn.

---
#### Mối quan hệ 1 - 1
---
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

* Nếu Reference Navigation **của thực thể con** có kiểu `Nullable<T>` (hay `T?`) thì được gọi là **Optional Reference Navigation** (điều hướng tùy chọn). Điều này đồng nghĩa với việc khóa ngoại khi được tạo ra hoặc xóa thực thể cha được phép gán giá trị `NULL`.
```csharp
    // principal (parent)
    public class A
    {
        public B? B_obj { get; set; }
    }

    // dependent (child)
    public class B
    {
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

---
#### Mối quan hệ 1 - N
---

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

Đôi khi ta cũng có thể khai báo thêm thuộc tính khóa ngoại.

**Ví dụ:**
```csharp
    public class Student
    {
        // other propreties ...
        public int? TeacherId { get; set; }
        public Teacher? Teacher { get; set; }
    }
```

Mối quan hệ 1 - N cũng có các khái niệm về **Required Reference Navigation** và **Optional Reference Navigation** tương tự [mối quan hệ 1 - 1](#mối-quan-hệ-1---1).


