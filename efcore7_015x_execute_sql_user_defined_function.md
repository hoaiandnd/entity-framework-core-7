# Execute non-query SQL - User-defined function

## User-defined function trong SQL Server

> - Bỏ qua nội dung này nếu đã có kiến thức về UDFs trong SQL Server.
>
> - Các lệnh T-SQL sử dụng trong các ví dụ sẽ không được giải thích chi tiết. Hãy tự tìm hiểu thêm.

Tương tự như khái niệm hàm trong các ngôn ngữ lập trình, User-defined functions (UDFs) trong SQL Server là một tập hợp các lệnh trong SQL, có thể nhận tham số để xử lý, tính toán và trả về một giá trị. Giá trị trả về có thể là một giá trị hoặc một tập dữ liệu.

UDF trong SQL Server chia thành 2 loại:

- **Scalar function**: hàm trả về một giá trị đơn (single value) với kiểu bất kỳ như `INT`, `VARCHAR`, `BIT`, ... nhưng trừ `TEXT`, `NTEXT`, `IMAGE`, `CURSOR`, và `TIMESTAMP`.

- **Table-valued function (TVF)**: trả về dữ liệu là một tập dữ liệu, hay một bảng dữ liệu (thông qua truy vấn).

### Scalar function

Cú pháp:

```sql
CREATE FUNCTION <FunctionName> ( [@Param1 datatype, @Param2 datatype, ...] )
RETURNS <return type>
AS
BEGIN
    -- Các câu lệnh T-SQL để thực hiện tính toán
    -- ...
    RETURN <value>;
END;
```

Trong đó:

- `<FunctionName>`: tên hàm, thường kèm theo tên lược đồ (schema) - `schema.function-name`, mặc định là `dbo`.

- `@Param1 datatype`: khai báo tham số cho hàm gồm tên tham số và kiểu dữ liệu của tham số.

> [!Note]
> Lưu ý rằng từ khoá `RETURNS` được dùng để khai báo kiểu trả về, từ khoá `RETURN` dùng để trả về giá trị thực sự.

**Ví dụ:**

```sql
CREATE FUNCTION dbo.getPostCount (@blogId VARCHAR(50))
RETURNS INT
AS
BEGIN
    DECLARE @count INT;
    SELECT @count = COUNT(id) FROM Post WHERE blog_id = @blogId;
    RETURN @count;
END;
```

### Table-valued function

Table-valued function (TVF) được chia thành 2 loại:

- Inline TVF: hàm chỉ chứa một câu truy vấn với `SELECT` và trả về tập dữ liệu dạng bảng.

- Multi-statements TVF: hàm có thể chứa nhiều câu lệnh T-SQL khác trước khi trả về dữ liệu dạng bảng.

#### Inline TVF

Cú pháp:

```sql
CREATE FUNCTION <FunctionName> ( [@Param1 datatype, @Param2 datatype, ...] )
RETURNS TABLE
AS
RETURN <SELECT statement>;
```

Inline TVF sẽ khai báo kiểu trả về là `TABLE` và trả về ngay một câu lệnh `SELECT`.

**Ví dụ:**

```sql
CREATE FUNCTION dbo.getPostsByBlog (@blogId INT)
RETURNS TABLE
AS
RETURN SELECT * FROM Post WHERE blog_id = @blogId;
```

#### Multi-statements TVF

Cú pháp:

```sql
CREATE FUNCTION <FunctionName> ( [@Param1 datatype, @Param2 datatype, ...] )
RETURNS @<TableVariable> TABLE (<Column Definitions>)
AS
BEGIN
    -- Các câu lệnh T-SQL để thao tác với biến bảng @TableVariable
    -- ...
    RETURN;
END;
```

Trong đó:

- `@<TableVariable>`: biến bảng, gọi biến này để thao tác với bảng sẽ chứa dữ liệu trả về.

- `<Column Definitions>`: định nghĩa các cột (trường) của bảng, chỉ ra cấu trúc của bảng sẽ được trả về.

Ta có thể sử dụng lệnh `INSERT`, `UPDATE`, `DELETE` trên biến bảng để thêm và chỉnh sửa dữ liệu cho bảng.

Biến bảng được tạo ra mặc định là bảng rỗng chỉ chứa cấu trúc, để thêm dữ liệu cho bảng này và trả về, hãy sử dụng cú pháp [**`INSERT INTO SELECT`**](https://www.w3schools.com/sql/sql_insert_into_select.asp). Cú pháp `INSERT INTO SELECT` dùng để sao chép dữ liệu trả về từ câu truy vấn `SELECT` và ghi vào bảng đã tồn tại.

**Ví dụ:**

```sql
CREATE FUNCTION dbo.getDeletedBlogs()
RETURNS @result TABLE (
    id VARCHAR(255),
    name NVARCHAR(255)
)
AS
BEGIN
    INSERT INTO @result
    SELECT id, name FROM Blog
    WHERE is_deleted = 1;
    RETURN;
END;
```

### Gọi hàm

Vì có trả về giá trị, nên UDF thường có thể sử dụng ở nhiều nơi trong câu truy vấn `SELECT`, `WHERE`, `HAVING`, ... để tính toán xử lý.

**Ví dụ:**

```sql
-- Sử dụng scalar function
SELECT dbo.getPostCount(1) -- hiển thị giá trị trả về
SELECT * FROM Blogs WHERE dbo.getPostCount(id) > 0 -- sử dụng trong câu truy vấn

-- Sử dụng table-valued function
SELECT * FROM dbo.getPostsByBlog(1)
SELECT * FROM dbo.getDeletedBlogs()
```


## User-defined function trong Entity Framework Core

### Scalar function

Các phương thức như `FromSql()`, `SqlQuery()` hay `ExecuteSql()` không thực sự hiệu quả khi gọi scalar function vì kiểu trả về của các phương thức này không phù hợp.

> Phương thức `FromSql()` và `SqlQuery()` trả về kiểu `IQueryable<T>`, còn `ExecuteSql()` trả về kiểu `int` đại diện cho số dòng bị ảnh hưởng - row affected.

#### Khai báo phương thức ánh xạ

Scalar function chỉ trả về một giá trị đơn, và EF Core cung cấp một phương pháp hiệu quả để gọi scalar function, đó là ánh xạ nó với một phương thức trong C#.

Đầu tiên, hãy khai báo một phương thức. Phương thức này phải thuộc 1 trong 2 trường hợp sau:

- Là phương thức thành viên của lớp context (kế thừa `DbContext`).

- Là một phương thức `static` (có thể đến từ một lớp khác hoặc của lớp context).

**Ví dụ:**

```cs
class BlogDbContext : DbContext
{
    // các cấu hình database ...
    public int GetPostCount(int blogId) => throw new NotImplementedException();
}
```

> [!Tip]
> EF Core khi ánh xạ sẽ không quan tâm đến phần định nghĩa của phương thức. Phần định nghĩa chỉ được gọi khi EF Core không thể ánh xạ được với UDF tương ứng trong SQL Server.

Sau khi khai báo một phương thức, có 2 cách để ánh xạ nó với UDF trong SQL Server:

- Sử dụng attribute [**`[DbFunction]`**](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbfunctionattribute?view=efcore-9.0) cho phương thức vừa khai báo.

- Cấu hình trong `OnModelCreating()` với phương thức [**`HasDbFunction()`**](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.relationalmodelbuilderextensions.hasdbfunction?view=efcore-9.0).

#### Sử dụng attribute `[DbFunction]`

**Ví dụ:**

```cs
[DbFunction]
public int GetPostCount(int blogId) => throw new NotImplementedException();
```

Theo mặc định nếu không chỉ định gì thêm, EF Core sẽ cố gắng ánh xạ với UDF trùng tên với phương thức trong C# (không phân biệt text-case, case-insensitive) và shema mặc định (với SQL Server là `dbo`).

Nếu như muốn thay đổi cho phù hợp, ta có thể chỉ định lại thông qua constructor của attribute:

```cs
DbFunctionAttribute(string name, string? schema = default)
```

**Ví dụ:**

```cs
[DbFunction("getPostCountFunc", "dbo2")]
public int GetPostCount(int blogId) => throw new NotImplementedException();
```

Các thuộc tính của attribute `[DbFunction]` có thể chỉ định kèm:

Thuộc tính | Mô tả
---|---
`IsBuiltIn` | Chỉ ra rằng hàm cần ánh xạ có phải là built-in function không.
`IsNullable` | Chỉ ra rằng hàm có trả về giá trị `NULL` hay không.
`IsNullableHasValue` | Xác định thuộc tính `IsNullable` có giá trị được chỉ định tường minh không.
`Name` | Tên của function trong database.
`Schema` | Tên schema của function.

#### Sử dụng phương thức cấu hình `HasDbFunction()`

Các cú pháp của phương thức `HasDbFunction()`:

```cs
HasDbFunction(MethodInfo methodInfo, bool fromDataAnnotation = false)
HasDbFunction(MethodInfo methodInfo, Action<DbFunctionBuilder> builderAction)
HasDbFunction(string name, Type returnType, bool fromDataAnnotation = false)
```

Trong đó:

- `methodInfo` (`System.Reflection.MethodInfo`): thông tin về phương thức C# sẽ ánh xạ đến hàm trong cơ sở dữ liệu.

- `fromDataAnnotation` (`bool`): chỉ ra rằng data annotation có được sử dụng để cấu hình không. Cụ thể ở đây là attribute `[DbFunction]`.

- `builderAction` (`Action<Microsoft.EntityFrameworkCore.Metadata.Builders.DbFunctionBuilder>`): một biểu thức lambda dùng để cấu hình cho hàm trong cơ sở dữ liệu.

- `name` (`string`): tên của hàm trong cơ sở dữ liệu.

- `returnType` (`Type`): kiểu trả về của hàm trong cơ sở dữ liệu.

> [!Tip]
> Kiểu `MethodInfo` thường được tạo ra bằng cách kết hợp cú pháp `typeof(ClassName).GetMethod("MethodName", Type[] types)`, với `types` là danh sách các kiểu tham số nhận vào của phương thức.

**Ví dụ:**

```cs
class BlogDbContext : DbContext
{
    // các cấu hình database ...
    public int GetPostCount(int blogId) => throw new NotImplementedException();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder
            .HasDbFunction(typeof(BlogDbContext).GetMethod(nameof(GetPostCount), [typeof(int)])!)
    }
}
```

#### Cách sử dụng

Sau khi khai báo và cấu hình, ta có thể gọi phương thức ánh xạ sử dụng như các phương thức thông thường khác.

> [!Important]
> Phương thức được ánh xạ phải được sử dụng trong LINQ, không được gọi bên ngoài ngữ cảnh.
>
> ```cs
> // không thực thi hàm trong database - chỉ thực thi phần định nghĩa của phương thức
> var postCount = context.GetPostCount(1);
>
> // phải gọi trong LINQ
> var postCount = context.Posts.Select(p => context.GetPostCount(p.BlogId)).First();
> ```

### Table-valued Functions

TVFs sẽ trả về một dữ liệu dạng bảng gồm nhiều bản ghi nên ta có thể sử dụng các phương thức thực thi lệnh SQL trực tiếp như `FromSql()`, `SqlQuery<TResult>()`, ...

**Ví dụ:**

```cs
// sử dụng `FromSql()`
var deletedBlogs = context.Blogs.FromSql($"SELECT * FROM dbo.getDeletedBlogs()");

// sử dụng `SqlQuery<TResult>`
record PostType(int Id, string Name);

var blogId = 1;
var posts = context.Database.SqlQuery<PostType>($"SELECT * FROM dbo.getPostsByBlog({blogId})");
```


