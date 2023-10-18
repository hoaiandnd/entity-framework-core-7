# Entity Framework Core 7.0 - Code First Approach - Map Attributes

Lớp thực thể trong C# không thể mô tả được hết hoặc chính xác những gì mà bảng dữ liệu trong CSDL có 
thể mô tả được. Hay nói cách khác, lớp thực thể trong ngôn ngữ hướng đối tượng như C# không thể mô 
tả các đặc điểm và ràng buộc ta muốn với bảng dữ liệu cần tạo.

Trong nội dung này, ta sẽ sử dụng các attribute để cấu hình lại lớp thực thể có thể ánh xạ thành bảng dữ
liệu trong CSDL.

Các attribute sử dụng trong tài liệu này đến từ 3 namespace sau:
* `System.ComponentModel.DataAnnotations`

* `System.ComponentModel.DataAnnotations.Schema`

* `Microsoft.EntityFrameworkCore`

## Cấu hình trên lớp

### Table

Cấu hình trên lớp dùng để cấu hình lại các thông tin về bảng dữ liệu sẽ tạo ra. Để cấu hình trên lớp, ta 
thường dùng attribute `[Table]`.

Với attribute `[Table]`, ta có thể chỉ định thuộc tính Name để quy định lại tên bảng (mặc định tên bảng sẽ
trùng với tên thuộc tính `DbSet<T>`) và thuộc tính Schema để xác định schema chứa bảng (mặc định là `dbo` với SQL Server).

Ví dụ:
```cs
    [Table(Name = "Orders")]
    public class Order
    {
        // properties ...
    }
```

### NotMapped

Attribute `[NotMapped]` dùng để tuyên bố rằng lớp đang chỉ định sẽ không ánh xạ thành bảng dữ liệu tương ứng.

**Ví dụ:**
```cs
    [NotMapped]
    class Person
    {
        // properties ...
    }
```

## Cấu hình trên thuộc tính

### NotMapped

Theo mặc định, các thuộc tính `public` cùng với các setter và getter sẽ được ánh xạ thành cột (trường) trong 
bảng dữ liệu

Không những dùng cho lớp thực thể, attribute `[NotMapped]` có thể dùng cho thuộc tính với cùng ý nghĩa - rằng thuộc tính đó
sẽ không ánh xạ thành cột (trường) trong CSDL.

**Ví dụ:**

```cs
    public class Person
    {
        // other properties ...

        [NoMapped]
        public int Age { get; set; }
    }
```

### Column

Ta sử dụng attribute `[Column]` để chỉ định một số thông tin cho thuộc tính tương ứng với cột (trường) sẽ
có trong bảng.

Một số thuộc tính có thể chỉ định kèm:
* `Name`: tên cột (mặc định tên cột cũng là tên của thuộc tính).

* `TypeName`: tên kiểu dữ liệu của cột (trong trường hợp kiểu dữ liệu của C# không biểu diễn đúng hoặc đầy đủ
kiểu dữ liệu trong CSDL).

**Ví dụ:**

```cs
    public class Person
    {
        // other properties ...

        [Column(Name = "personName", TypeName = "nvarchar(100)")]
        public string Name { get; set; }
    }
```

Bên cạnh đó, phương thức khởi tạo của `[Column]` cũng cho phép ta chỉ định tên cột – không cần thuộc tính 
`Name`.

```cs
    [Column(string name)]
```

### Các ràng buộc trên thuộc tính

Ta có các attribute sau thường dùng để ràng buộc thuộc tính

<table>
        <tr>
            <th>Attribute</th>
            <th>Mô tả</th>
        </tr>
        <tr>
            <td><code>[Required]</code></td>
            <td>Ràng buộc NOT NULL</td>
        </tr>
        <tr>
            <td>
                <code>[MinLength(int length)]</code><br>
                <code>[MaxLength(int length)]</code><br>
                <code>[StringLength(int maximumLength)]</code><br>
                <code>[StringLength(int maximumLength, MinimumLength)]</code>
            </td>
            <td>Ràng buộc độ dài chuỗi</td>
        </tr>
        <tr>
            <td>
                <code>[Range(int minimum, int maximum)]</code><br>
                <code>[Range(double minimum, double maximum)]</code><br>
                <code>[Range(Type type, string minimum, string maximum)]</code>
            </td>
            <td>Ràng buộc khoảng giá trị số</td>
        </tr>
        <tr>
            <td>
                <code>[Precision(int precision)] </code><br>
                <code>[Precision(int precision, int scale)] </code>
            </td>
            <td>Ràng buộc độ chính xác của số</td>
        </tr>
        <tr>
            <td>
                <code>[Unicode(bool unicode = true)]</code>
            </td>
            <td>Ràng buộc cột lưu trữ với Unicode hay không</td>
        </tr>
    </table>
