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




