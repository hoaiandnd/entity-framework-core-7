# Execute non-query SQL - User-defined function

## User-defined function trong SQL Server

Tương tự như khái niệm hàm trong các ngôn ngữ lập trình, User-defined functions (UDFs) trong SQL Server là một tập hợp các lệnh trong SQL, có thể nhận tham số để xử lý, tính toán và trả về một giá trị. Giá trị trả về có thể là một giá trị hoặc một tập dữ liệu.

> Phần cuối sẽ là nội dung so sánh UDF và Store procedure.

UDF trong SQL Server chia thành 2 loại:

- Scalar function: hàm trả về một giá trị kiểu bất ký trừ `text`, `ntext`, `image`, `cursor`, và `timestamp`.

- Table-Valued Function (TVF): trả về dữ liệu là một bảng.


