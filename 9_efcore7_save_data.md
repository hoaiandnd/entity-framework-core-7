# Entity Framework Core 7.0 - Save data

## Phương thức SaveChanges()

Phương thức `DbContext.SaveChanges()` là một trong 2 kỹ thuật dùng để lưu lại các thay đổi trên CSDL với EF Core. Phương thức này được dùng cho cả 3 thao tác thêm, xóa và cập nhật dữ liệu dành cho các thực thể được theo dõi (Tracking) bởi EF Core (Xem nội dung [**Querying data**](/8_efcore7_save_data.md)).

## Thêm dữ liệu

Để thêm một thực thể vào CSDL ở mức cơ bản nhất, ta cần:

* Tạo ra một thực thể mới.

* Gọi phương thức `DbSet<TEntity>.Add()`.

* Thực thể sẽ được thêm vào CSDL khi gọi phương thức `DbContext.SaveChanges()`.

**Ví dụ:**

```cs
    // create DbContext instance
    var db = new EmployeeManagerContext();

    // create new entity
    var department = new Department() { Name = "Sales Department" };

    // add new entity to 'Department' table
    db.Departments.Add(department);

    // save changes to database
    db.SaveChanges();
```

Ngoài ra, ta có thể sử dụng phương thức `DbSet<TEntity>.AddRange()` để thêm nhiều thực thể cùng lúc với tham số nhận vào kiểu `IEnumerable<TEntity>` hoặc `TEntity[]`.

Nếu không chỉ định giá trị cho toàn bộ thuộc tính của thực thể thì:
* Giá trị `NULL` sẽ được dùng nếu như thuộc tính khai báo với kiểu tham chiếu hoặc kiểu nullable.

* Exception sẽ được ném ra nếu thuộc tính không thể chứa giá trị `NULL` (kiểu dữ liệu nguyên thủy, kiểu non-nullable, chỉ định ràng buộc với `[Required]` của [**Map Attributes**](/3_efcore_code_first_approach_map_attributes.md) hay `IsRequired()` của [**Fluent API**](/6_efcore_code_first_approach_fluent_api.md), khóa chính, ...).

* Nếu khóa chính có cài đặt `IDENTITY` trong SQL Server thì không cần chỉ định giá trị.

Ta có thể thêm các đối tượng con trong quá trình thêm thực thể cha thông qua Collection Navigation.

**Ví dụ:**

```cs
    var db = new EmployeeManagerContext();

    var department = new Department()
    { 
        Name = "Sales Department",
        Employees = new List<Employee>() // chỉ định các thực thể con sẽ thêm cùng
        {
            new Employee() { Name = "John", Salary = 16000 },
            new Employee() { Name = "David", Salary = 1500 },
            new Employee() { Name = "Mary", Salary = 25700 }
        }
    };

    db.Departments.Add(department);
    db.SaveChanges();
```

Các thực thể con được tạo cùng với thực thể cha sẽ tự động cập nhật về giá trị khóa ngoại, Reference Navigation tương ứng.

Ngược lại, ta cũng có thể thêm thực thể cha trong khi thêm thực thể con, thông qua Reference Navigation (không dùng thuộc tính khóa ngoại). Thực thể cha sẽ được lưu vào trước khi thêm thực thể con.

**Ví dụ:**

```cs
    var db = new EmployeeManagerContext();

    var employee = new Employee()
    {
         Name = "Bob",
         Salary = 10500,
         Department = new Department() { Name = "IT Department" } // add new parent entity by reference navigation
    };

    db.Employees.Add(employee);
    db.SaveChanges();
```

> [!Warning]
> Nếu muốn gán giá trị cho thuộc tính khóa ngoại, hãy chắc chắn giá trị khóa ngoại đó là hợp lệ (tức là phải tồn tại trong CSDL).

Trước khi gọi phương thức `SaveChanges()`, ta hoàn toàn có thể thực hiện nhiều thao tác với CSDL bao gồm thêm, cập nhật và xóa dữ liệu.

Một cách khác để thêm (nhiều) thực thể vào CSDL là sử dụng phương thức `Add<TEntity>()` hoặc `AddRange()` của lớp Context với cú pháp:

```cs
    Add<TEntity>(TEntity entity)

    AddRange(IEnumerable<object> entities)

    AddRange(object[] entities)
```

Sau đó gọi phương thức `SaveChanges()` để thực hiện thêm (các) thực thể vào CSDL.

**Ví dụ:**

```cs
    var db = new EmployeeManagerContext();
    var employee = new Employee()
    {
         Name = "Bob",
        Salary = 10500,
        Department = db.Departments.Single(d => d.Name == "IT Department");
    };

    db.Add<Employee>(employee); // or "db.Add(employee);"
    db.SaveChanges();
```

Ngoài ra, nếu thực thể có khóa với giá trị tự động tạo (Auto Generated Value), ta có thể sử dụng phương thức `Update<TEntity>()` hoặc `UpdateRange()` với cùng cú pháp.

Về cơ bản, phương thức `Update<TEntity>()` và `UpdateRange()` không được dùng để thêm thực thể, tuy nhiên nếu thực thể không có giá trị khóa (khóa không được đặt vì được tạo tự động) thì các phương thức trên sẽ thực hiện thêm thực thể mới thay vì cập nhật.

## Cập nhật dữ liệu

Theo mặc định, các thực thể trả về qua truy vấn LINQ sẽ được theo dõi bởi EF Core. Vì vậy, để cập nhật dữ
liệu cho thực thể, ta chỉ cần gọi ra các thuộc tính cần cập nhật và gán cho một giá trị mới. Phương thức 
`SaveChanges()` sẽ theo dõi các thay đổi và tự động thực hiện cập nhật lại trong CSDL.

**Ví dụ:**

```cs
    var db = new EmployeeManagerContext();

    var employee = db.Employees.Single(e => e.Id == 15); // find entity need updating

    employee.Salary = 18050; // update 'Salary' property
    db.SaveChanges(); // save changes to database
```

> [!Warning]
> Không thể thực hiện cập nhật trường khóa chính

Trong quá trình cập nhật dữ liệu, ta có thể thêm các dữ liệu của (các) thực thể liên quan.

**Ví dụ:**

```cs
    var db = new EmployeeManagerContext();

    var itDepartment = db.Departments.Single(d => d.Name == "IT Department");
    itDepartment.Name = "Information Technology Department"; // update department name

    // update – add new child entities
    itDepartment.Employees.Add(new Employee() { Name = "Peter", Salary = 7600 });

    db.SaveChanges();
```

Ta hoàn toàn có thể thay đổi thuộc tính khóa ngoại hoặc Navigation Property, sao cho không làm vi phạm ràng buộc tham chiếu trong CSDL.

Có thể dùng phương thức `Update<TEntity>()` và `UpdateRange()` của đối tượng Context để cập nhật dữ
liệu với cú pháp sau:

```cs
    Update<TEntity>(TEntity entity)

    UpdateRange(IEnumerable<object> entities)

    UpdateRange(object[] entities)
```

Đặc điểm của các phương thức trên là ***không cần thực thể phải được theo dõi bởi EF Core*** (No-Tracking 
Entity). Dựa vào khóa chính được cung cấp bởi thực thể mà các phương thức trên sẽ xác nhận thực thể nào 
trong CSDL sẽ được cập nhật.

**Ví dụ:**

```cs
    var employee = db.Employees.AsNoTracking().Single(e => e.Id == 15);
    employee.Name = "New name"; // update data for no tracking entity

    db.Update(employee);
    db.SaveChanges();
```

Nếu khóa chính của thực thể chưa có giá trị (thường là do khóa chính được cài đặt có giá trị tự động), các 
phương thức `Update<TEntity>()` và `UpdateRange()` sẽ sinh ra câu lệnh `INSERT` thay vì `UPDATE`.

**Entity Framework Core 7.0** giới thiệu phương thức `ExecuteUpdate()` được dùng để cập nhật dữ liệu mà 
không cần gọi `SaveChanges()` với cách biểu diễn tương tự câu lệnh `UPDATE` của SQL.
Để cập nhật dữ liệu với phương thức `ExecuteUpdate()`, ta cần sử dụng LINQ để truy vấn (các) thực thể cần 
cập nhật dữ liệu. Sau đó gọi phương thức `ExecuteUpdate()` và `SetProperty()` để chọn thuộc tính cần cập 
nhật.

Cú pháp:

```cs
    ExecuteUpdate<TSource>(Expression<Func<SetPropertyCalls<TSource>,SetPropertyCalls<TSource>>> setPropertyCalls)
```

Để trả về kiểu `SetPropertyCalls<TSource>`, ta sẽ sử dụng phương thức `SetProperty<TProperty>()` với một trong 2 cú pháp sau:

```cs
    SetProperty<TProperty>(Func<TSource,TProperty> property, Func<TSource,TProperty> value)

    SetProperty<TProperty>(Func<TSource,TProperty> property, TProperty value)
```

Trong đó:

* *`property`*: thường dùng một biểu thức Lambda để chỉ ra thuộc tính cần cập nhật dữ liệu.

* *`value`*: dùng để chỉ định giá trị mới cho thuộc tính.

**Ví dụ:**

```cs
    var updateSalesCount = db.Employees
                             .Where(e => e.Department.Name == "Sales Department")
                             .ExecuteUpdate(e => e.SetProperty(emp => emp.Salary, 50000));

    var updateITCount = db.Employees
                          .Where(e => e.Department.Name == "IT Department")
                          .ExecuteUpdate(e => e.SetProperty(emp => emp.Salary, emp => emp.Salary + 5000));
```

Ta có thể gọi lần lượt nhiều phương thức `SetProperty().SetProperty()...` để cập nhật cho nhiều thuộc tính.

> [!Warning]
> Hiện tại biểu thức Lambda trong phương thức `SetProperty()` không được hỗ trợ để gọi đến Reference Navigation hay Collection Navigation.

## Xóa dữ liệu

Ta sử dụng phương thức `DbSet<TEntity>.Remove()` để xóa dữ liệu. Nếu dữ liệu có tồn tại trong CSDL thì nó 
sẽ được xóa khỏi CSDL sau khi gọi phương thức `SaveChanges()`. Nếu không, dữ liệu được chỉ định sẽ bị loại 
khỏi trạng thái được thêm vào (Added State) và không còn thêm vào CSDL bằng `SaveChanges()`.

Nếu muốn xóa nhiều thực thể cùng lúc, hãy sử dụng phương thức `RemoveRange()` với tham số kiểu 
`TEntity[]` hoặc `IEnumerable<TEntity>`.

Để xóa dữ liệu có trong CSDL, hãy sử dụng truy vấn LINQ để xác định dữ liệu cần xóa.

**Ví dụ:**

```cs
    var employee = db.Employees.Where(e => e.Salary < 1000);
    db.Employees.RemoveRange(employee);
    db.SaveChanges();
```

**Từ EF Core 7.0**, ta có thể sử dụng phương thức `ExecuteDelete()` sau truy vấn LINQ để xóa dữ liệu mà 
không cần gọi phương thức `SaveChanges()`.

**Ví dụ:**

```cs
    db.Employees.Where(e => e.Salary < 1000).ExecuteDelete();
```

Khi thực hiện xóa thực thể trong CSDL, ta cần quan tâm đến (các) thực thể liên quan. Vấn đề này sẽ được 
trình bày chi tiết ở nội dung [**Cascade delete**]().

