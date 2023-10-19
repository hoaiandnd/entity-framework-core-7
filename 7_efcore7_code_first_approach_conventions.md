# Entity Framework Core 7.0 - Code First Approach - Conventions

**Conventions** (Quy ước) là một tập các quy ước định sẵn mà EF Core dùng để cấu hình các lớp thực thể
thành bảng dữ liệu. Nếu tuân theo các quy ước, lớp thực thể sẽ ngầm định đạt được các cấu hình nhất định.

Conventions là một trong 3 cách cấu hình lớp thực thể bên cạnh [**Map Attributes**](/3_efcore7_code_first_approach_map_attributes.md) và [**Fluent API**](/6_efcore7_code_first_approach_fluent_api.md).

Nội dung phần này sẽ đề cập đến 2 vấn đề:

* Conventions cấu hình thực thể

* Conventions cấu hình mối quan hệ

## Conventions cấu hình thực thể

### Khóa chính

Theo quy ước, các thuộc tính `public` có tên là `Id` hoặc `<entity type>Id` sẽ được tự động cấu hình làm khóa 
chính (không phân biệt chữ hoa, thường).

**Ví dụ:**

```cs
    public class Book
    {
        public int Id { get; set; } // or 'ID', 'id'
        // or
        public int BookId { get; set; } // or 'bookId', 'BookID', ...
        // other properties ...
    }
```

Các khóa chính được cấu hình bằng Conventions mà có kiểu dữ liệu số (`int`, `double`, `float`, ...) sẽ tự động được cài đặt tự động tăng `IDENTITY` trong SQL Server.

EF Core hỗ trợ thuộc tính khóa chính sử dụng kiểu dữ liệu nguyên thủy (Primitive type), bao gồm cả kiểu 
`string`, `Guid`, `byte[]` và các kiểu khác tương tự. Tuy nhiên, không phải các CSDL đều hỗ trợ tất cả kiểu cho 
khóa chính, nên đôi khi sẽ phải thực hiện cấu hình tường minh (bằng [**Map Attributes**](/3_efcore7_code_first_approach_map_attributes.md) hoặc [**Fluent API**](/6_efcore7_code_first_approach_fluent_api.md)).

### Khóa ngoại

Một thuộc tính được cấu hình thành khóa ngoại nếu như:

* Kiểu dữ liệu của thuộc tính khóa ngoại tương thích với kiểu khóa chính của lớp thực thể cha (tương thích ở đây có thể hiểu là kiểu `T` hoặc `T?`).

* Tên thuộc tính phù hợp với một trong các định dạng quy ước sau:

    * `<navigation name><principal key name>`

    * `<navigation name>Id`

    * `<principal entity name><principal key name>`

    * `<principal entity name>Id`

**Ví dụ:**

```cs
    public class Book
    {
        public int Id { get; set; }
        public string BookName { get; set; }

        public int AuthorId { get; set; } // foreign key property
        public Author? Author { get; set; } // navigation property
    }
    public class Author
    {
        public int Id { get; set; }
        public string Name { get; set; }

        public ICollection<Book> Books { get; set; } // collection navigation
    }

```

### Bảng dữ liệu, Schema, Column

* Mặc định, tên của bảng dữ liệu sẽ trùng với tên thuộc tính `DbSet<TEntity>` được khai báo trong lớp Context.

* EF Core mặc định sẽ ánh xạ đến schema `dbo`.

* Tên cột (trường) mặc định trùng với tên thuộc tính được khai báo trong lớp thực thể.

### Kiểu dữ liệu

Kiểu dữ liệu cho cột trong bảng dữ liệu tùy thuộc vào cách mà Database Provider ánh xạ kiểu dữ liệu .NET 
thành kiểu dữ liệu cho CSDL tương ứng.

Bảng bên dưới là danh sách các kiểu dữ liệu C# và kiểu tương ứng được ánh xạ sang SQL Server:

| Kiểu dữ liệu C# | Kiểu ánh xạ trong SQL Server |
| --- | --- |
| `int` | `INT` |
| `string` | `NVARCHAR(MAX)` hoặc `NVARCHAR(450)` dành cho khóa chính |
| `decimal` | `DECIMAL`, `MONEY`, `NUMERIC` |
| `float` | `REAL` |
| `byte[]` | `BINARY`, `VARBINARY(MAX)`, `VARBINARY`, `IMAGE`, `TIMESTAMP` |
| `DateTime` | `DATE`, `DATETIME`, `DATETIME2`, `SMALLDATETIME` |
| `bool` | `BIT` |
| `byte` | `TINYINT` |
| `short` | `SMALLINT` |
| `long` | `BIGINT` |
| `double` | `FLOAT` |

> Tham khảo thêm tại [**SQL Server Data Type Mappings - ADO.NET | Microsoft Learn**](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql-server-data-type-mappings).

Với các thuộc tính có kiểu dữ liệu tham chiếu hoặc kiểu nullable (`int?`, `string?`, ...), cột tương ứng sẽ có 
thể chứa giá trị `NULL`. Ngược lại các kiểu dữ liệu nguyên thủy sẽ tạo ra cột `NOT NULL`.

### Conventions cấu hình mối quan hệ

#### Reference Navigation

Một thuộc tính được xem là Reference Navigation khi:

* Có phạm vi truy cập là `public`.

* Phải có **Setter** và **Getter**. **Setter** có thể không bắt buộc phải công khai mà có thể dùng với `private`
hoặc `protected`. Ngoài ra có thể thay `set` bằng `init`.

* Kiểu dữ liệu của thuộc tính phải là kiểu tham chiếu (Reference type) trừ `string`.

* Thuộc tính không chỉ định với `static` và cũng không phải là **Indexer**.

Reference Navigation trong hầu hết các trường hợp đều sử dụng kiểu của thực thể liên quan trong mối quan hệ.

#### Collection Navigation

Collection Navigation được dùng để chỉ Navigation Property tham chiếu đến nhiều thực thể con trong mối 
quan hệ 1 – N hoặc N – N.

Một thuộc tính được xem là Collection Navigation khi:

* Có phạm vi truy cập là `public`.

* Phải có **Getter**. Có thể có thêm **Setter** nhưng không bắt buộc.

* Kiểu dữ liệu của thuộc tính phải triển khai `IEnumerable<TEntity>`, trong đó `TEntity` phải là kiểu 
tham chiếu trừ `string`.

* Thuộc tính không chỉ định với `static` và cũng không phải là **Indexer**.

### Quan hệ N - N

Mối quan hệ 1 – 1 và 1 – N được quy ước sẵn bởi Reference Navigation và Collection Navigation. Do đó, ta 
sẽ tìm hiểu về Conventions trong mối quan hệ N – N.

Trước **EF Core 5.0**, ta không thể cấu hình mối quan hệ N – N nếu như không định nghĩa lớp thực thể trung 
gian (**Join Entity Type**) và cấu hình bằng [**Fluent API**](/6_efcore7_code_first_approach_fluent_api.md).

Từ phiên bản **EF Core 5.0**, Conventions cho phép ta không cần phải khai báo lớp thực thể trung gian. Các 
lớp thực thể cần tạo mối quan hệ N – N chỉ cần cung cấp 2 Collection Navigation tương ứng trong từng 
lớp.

Conventions cho mối quan hệ N – N có nội dung như sau:

* Lớp thực thể trung gian sẽ tự động tạo ra với tên `<left entity name><right entity name>` (kết 
nối tên của 2 thực thể chính).

* Lớp thực thể sẽ chứa 2 khóa chính với tên `<navigation name><principal key name>` theo 2 lớp 
thực thể chính với kiểu non-nullable, yêu cầu sự hiện diện của cả 2 thực thể chính.

### Cascade Delete

Xem nội dung về [**Cascade Delete**]().
