# Execute non-query SQL

> Ở chương [**SQL Queries**](/efcore7_013x_sql_queries.md), ta đã tìm hiều về cách thực thi các câu truy vấn SQL và trả về dữ liệu. Ở nội dung này, ta sẽ tìm hiểu những nội dung còn lại khi thao tác với câu lệnh SQL như Store Procedure, User Function, View, ... trong SQL Server.

## Store Procedure

### Sơ lược về Store procedure trong SQL Server

> Bỏ qua nội dung này nếu đã có kiến thức về Store procedure

Store procedure trong SQL Server là một nhóm các lệnh T-SQL (Transact-SQL). Nếu ở góc nhìn C# thì có thể hiểu Store Procedure tương tự hàm/phương thức trong SQL (mặc dù SQL Server vẫn có hàm) với các đặc điểm sau:

- Có thể nhận tham số đầu vào và trả về nhiều giá trị thông qua tham số đầu ra (output parameters).

- Chứa các lệnh thao tác với database, kể cả việc gọi Store procedure khác.

- Trả về giá trị trạng thái (thành công hoặc thất bại).

#### Tạo Procedure

Cú pháp:

```sql
CREATE PROCEDURE <ProcedureName>
   @<ParameterName> <data type> [OUT | OUTPUT]
AS
BEGIN
  -- Các câu lệnh T-SQL sử dụng ở đây
END
```

Trong đó:

- `<ProcedureName>`: tên của procedure, thường kèm theo tên schema, ví dụ: `dbo.getBlogCount`.

- `<ParameterName>`, ...: các tham số của procedure. Nếu không khai báo `OUT` hoặc `OUTPUT` sẽ được hiểu là tham số đầu vào (input parameters). Một procedure có thể nhận nhiều tham số (đầu vào và đầu ra), phân cách bằng dấu phẩy

- `<data type>`: kiểu dữ liệu trong T-SQL.

> [!Tip]
> Từ khoá `PROCEDURE` có thể viết tắt là `PROC`.

Ta có thể sử dụng nhiều câu lệnh T-SQL trong procedure, đơn giản nhất là `SELECT`, `INSERT`, `UPDATE`, `DELETE`.

**Ví dụ:**

- Store procedure không có tham số:

```sql
CREATE PROCEDURE dbo.getDeletedBlogs
AS
BEGIN
   SELECT * FROM Blog WHERE is_deleted = 1
END
```

- Store procedure chỉ có tham số đầu vào:

```sql
CREATE PROCEDURE dbo.insertNewBlog
   @blogName NVARCHAR(255)
AS
BEGIN
   INSERT INTO Blog([name], is_deleted) VALUES (@blogName, 0)
END
```

- Store procedure có tham số đầu ra (output parameters):

```sql
CREATE PROCEDURE dbo.getDeletedBlogCount
   @deletedBlogCount INT OUTPUT
AS
BEGIN
   SELECT @deletedBlogCount = COUNT(id) FROM Blog WHERE is_deleted = 1
END
```

> `SELECT` trong ví dụ trên đóng vai trò gán giá trị cho biến từ câu truy vấn hiện tại.

> [!Tip]
> Có thể sử dụng lệnh `SET NOCOUNT ON` để tăng tốc thực thi cho procedure.

#### Gọi và thực thi

Để gọi và thực thi một procedure, ta sử dụng cú pháp sau:

```sql
EXECUTE <ProcedureName> @<ParameterName> = <value> [OUT | OUTPUT]
```

Trong đó: `<value>` là giá trị đối số truyền vào tham số chỉ định thẻ `@<ParameterName>`. `<value>` thường là một biến nếu như truyền vào cho tham số đầu ra.

> [!Tip]
> Từ khoá `EXECUTE` có thể viết tắt thành `EXEC`.

**Ví dụ:**

```sql
-- gọi procedure không tham số
EXECUTE dbo.getDeletedBlogs
GO

-- gọi procedure có tham số đầu vào
EXECUTE dbo.insertNewBlog @blogName = N'New Blog'
GO

-- gọi procedure có tham số đầu ra
DECLARE @count INT -- khai báo một biến nhận giá trị đầu ra
EXECUTE dbo.getDeletedBlogCount @deletedBlogCount = @count OUT
PRINT @count -- hiển thị giá trị được trả về
```

### Sử dụng Store procedure trong Entity Framework Core

Đối với góc nhìn của EF Core, ta chia Store procedure thành 2 loại:

- Có trả về dữ liệu (thực hiện truy vấn bằng `SELECT`)

- Không trả về dữ liệu (thường chứa các câu lệnh không có dữ liệu trả về như `INSERT`, `UPDATE`, `DELETE`, ...).

#### Store procedure có trả về đữ liệu

Đối với EF Core, một procedure có trả về kết quả là procedure có thực hiện truy vấn bằng lệnh `SELECT`.

🔖 Một số lưu ý:

- EF Core chỉ nhận kết quả từ truy vấn `SELECT` **đầu tiên**, trong trường hợp procedure có nhiều truy vấn (multiple result set).

```sql
CREATE PROCEDURE dbo.getDeletedBlogsMultiple
AS
BEGIN
   SELECT * FROM Blog WHERE is_deleted = 1; -- lấy dữ liệu từ truy vấn đầu tiên
   SELECT * FROM Post WHERE is_deleted = 1; -- bỏ qua
END
```

- Miễn là procedure có dùng `SELECT` để truy vấn thì EF Core sẽ sử dụng kết quả của truy vấn đó, bất kể phía sau có còn lệnh T-SQL nào khác hay không.

```sql
CREATE PROCEDURE dbo.getDeletedBlogsAndDoSomething
AS
BEGIN
   SELECT * FROM Blog WHERE is_deleted = 1; -- lấy dữ liệu từ truy vấn đầu tiên
   PRINT('Something')
END
```

Để lấy dữ liệu trả về từ Store procedure, ta sử dụng phương thức `FromSql()` hoặc một số phương thức tương tự như `FromSqlRaw()`, `SqlQuery<TResult>()`, ... như đã đề cập trong chương [**SQL Queries**](/efcore7_013x_sql_queries.md).

**Ví dụ:**

```cs
var deletedBlogs = context.Blogs.FromSql($"EXECUTE dbo.getDeletedBlogs").ToList();
```

Phương thức `FromSql()` hoặc `FromSqlRaw()` thường được sử dụng đối với những store procedure trả về toàn vẹn thực thể (tức là trả về toàn bộ thuộc tính/trường của 1 thực thể/bản ghi). 

Ngược lại nếu chỉ truy vấn trên 1 vài cột thì nên sử dụng `SqlQuery<TResult>()` hoặc `SqlQueryRaw<TResult>()`.

**Ví dụ:**

<details>
  <summary>Store procedure<br /></summary>
   
```sql
CREATE PROCEDURE dbo.getDeletedBlogsSqlQuery
AS
BEGIN
   SELECT id, name FROM Blog WHERE is_deleted = 1; -- chỉ trả về một số cột dữ liệu --> thực thể không toàn vẹn
END
```

</details>

```cs
// khai báo kiểu thực thể trả về
record GetDeletedBlogsResult(int Id, string Name);

// sử dụng kiểu thực thể đã định nghĩa để nhận kết quả từ store procedure
var deletedBlogs = context.Database.SqlQuery<GetDeletedBlogsResult>($"EXECUTE dbo.getDeletedBlogsSqlQuery").ToList();
```

Đối với store procedure yêu cầu tham số đầu vào, cách đơn giản nhất là truyền giá trị vào câu lệnh thực thi tương tự cú pháp chuỗi nội suy của C#:

<details>
  <summary>Store procedure<br /></summary>
   
```sql
CREATE PROCEDURE dbo.searchBlogs
   @name NVARCHAR(255)
AS
BEGIN
   SELECT * FROM Blog WHERE [name] LIKE '%' + @name + '%'
END
```

</details>

```cs
var searchName = "fact";
var blogs = context.Blogs.FromSql($"EXECUTE dbo.searchBlogs @name = {searchName}").ToList();
```

> [!Tip]
> Nếu Store procedure chỉ có 1 tham số đầu vào, ta có thể truyền trực tiếp ngay sau tên procedure (cú pháp của SQL - không phải do EF Core xử lý).
>
> ```cs
> var searchName = "fact";
> var blogs = context.Blogs.FromSql($"EXECUTE dbo.searchBlogs {searchName}").ToList();
> ```

Một cách khác là sử dụng đối tượng `SqlParameter`.

**Ví dụ:**

```cs
var param = new SqlParameter("name", "Blog");
var blogs = db.Blogs.FromSql($"EXECUTE dbo.searchBlog {param}").ToList();
```

Có nhiều phiên bản overload constructor của [**`SqlParameter`**](https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqlparameter?view=sqlclient-dotnet-standard-5.2), bên dưới là một số cú pháp phổ biến:

```cs
SqlParameter(string parameterName, SqlDbType dbType, int size);
SqlParameter(string parameterName, object value);
SqlParameter(string parameterName, SqlDbType dbType);
```

Trong đó:

- `parameterName`: tên của tham số (case insensitive), có thể hoặc không chứa ký tự `@`.

- `dbType`: kiểu dữ liệu của tham số tương ứng được định nghĩa bằng enum [**`SqlDbType`**](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqldbtype?view=net-9.0) như `SqlDbType.Int`, `SqlDbType.NVarChar`, ...

- `size`: kích thước của kiểu dữ liệu, ví dụ: `NVARCHAR(255)` thì `size` sẽ là `255`. Thường chỉ định cùng với `dbType`.

- `value`: giá trị của tham số.

> Kiểu `SqlParameter` còn nhiều thuộc tính khác để cấu hình cho tham số như `Scale`, `Precision` (thường dùng cho kiểu `DECIMAL` trong SQL), `Direction`, ...

**Ví dụ:**

```cs
var param = new SqlParameter("name", SqlDbType.NVarChar, 255);
param.Value = "Blog";
var blogs = db.Blogs.FromSql($"EXECUTE dbo.searchBlog {param}").ToList();
```
