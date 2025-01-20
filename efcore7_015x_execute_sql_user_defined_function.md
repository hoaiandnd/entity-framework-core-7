# Execute non-query SQL - User-defined function

## User-defined function trong SQL Server

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

- 
