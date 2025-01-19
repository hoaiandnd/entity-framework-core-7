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
  -- T-SQL statements
END
```

Trong đó:

- `<ProcedureName>`: tên của procedure, thường kèm theo tên schema, ví dụ: `dbo.getBlogCount`.

- `<ParameterName>`, ...: các tham số của procedure. Nếu không khai báo `OUT` hoặc `OUTPUT` sẽ được hiểu là tham số đầu vào (input parameters). Một procedure có thể nhận nhiều tham số (đầu vào và đầu ra), phân cách bằng dấu phẩy

- `<data tyoe>`: kiểu dữ liệu trong T-SQL.

> [!Tip]
> Từ khoá `PROCEDURE` có thể viết tắt là `PROC`.






