# Execute non-query SQL - User-defined function

## User-defined function trong SQL Server

> - Bỏ qua nội dung này nếu đã có kiến thức về UDFs trong SQL Server.
>
> - Các lệnh T-SQL sử dụng trong các ví dụ sẽ không được giải thích chi tiết. Hãy tự tìm hiểu thêm.

Tương tự như khái niệm hàm trong các ngôn ngữ lập trình, User-defined functions (UDFs) trong SQL Server là một tập hợp các lệnh trong SQL, có thể nhận tham số để xử lý, tính toán và trả về một giá trị. Giá trị trả về có thể là một giá trị hoặc một tập dữ liệu.

> Phần cuối sẽ là nội dung so sánh UDF và Store procedure.

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
CREATE FUNCTION dbo.getPostByBlog (@blogId VARCHAR(50))
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

## User-defined function trong Entity Framework Core






