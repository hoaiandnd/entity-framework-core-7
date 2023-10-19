# Entity Framework Core 7.0 - Querying Data

> [!Note]
> *Nội dung này dành cho cả Database First Approach và Code First Approach*

> [!Important]
> Nội dung yêu cầu kiến thức về LINQ (Language Intergrated Query)

EF Core sử dụng **Language-Integrated Query** (LINQ) để truy vấn dữ liệu từ CSDL. Nguồn dữ liệu sử dụng là 
các thuộc tính `DbSet<TEntity>` trong lớp Context.

Kiểu `DbSet<TEntity>` có triển khai cả `IEnumerable<T>` và `IQueryable<T>`, do đó mà ta có thể sử dụng LINQ để
truy vấn một cách bình thường.

**Ví dụ:**

```cs
    var db = new BusinessContext(); // đối tượng dbcontext

    // sử dụng LINQ query syntax
    var employees = from emp in db.Employees
                    where emp.Salary >= 1500
                    select new { emp.Name, emp.Salary };

    // hoặc sử dụng LINQ method syntax
    var employees = db.Employees
                        .Where(e => e.Salary)
                        .Select(e => new { emp.Name, emp.Salary });
    
    // hiển thị kết quả
    foreach(var e in result)
    {
        Console.WriteLine($"{e.Name}: {e.Salary}");
    }
```

Nội dung của phần này sẽ không đề cập về cách truy vấn dữ liệu với LINQ. Thay vào đó, ta sẽ tìm hiểu 
một số vấn đề với thao tác truy vấn dữ liệu trong Entity Framework Core 7.0.

## Tracking vs. No-Tracking Queries

Theo mặc định, (các) thực thể được trả về bởi câu truy vấn sẽ được **theo dõi** (Tracking). EF Core sẽ theo dõi 
bất kỳ sự thay đổi nào trên (các) thực thể và đồng bộ dữ liệu sau khi gọi phương thức `SaveChanges()`.

Phương thức `SaveChanges()` là phương thức sẵn có ở các lớp Context – lớp kế thừa của `DbContext`, được 
dùng để lưu lại các thay đổi (thêm, cập nhật hoặc xóa) trên thực thể vào CSDL bằng EF Core. Xem chi tiết ở
phần [**Insert, Update and Delete**]().

**No-tracking queries** (Truy vấn không theo dõi) sẽ có ích nếu như kết quả truy vấn được dùng với mục đích 
chỉ đọc (Read-only). Nó sẽ làm câu truy vấn thực thi nhanh hơn vì không cần phải cài đặt theo dõi thực thể.

Tóm lại, nếu như dữ liệu được truy vấn mà không có nhu cầu cập nhật, thì ta nên sử dụng truy vấn không 
theo dõi.

Để câu truy vấn không theo dõi (các) thực thể khi trả về kết quả, ta sử dụng phương thức `AsNoTracking()`
từ thuộc tính `DbSet<TEntity>`.

**Ví dụ:**

```cs
    var db = new BusinessContext();

    // sử dụng với LINQ method syntax
    var emp_1 = db.Employees.AsNoTracking().Select(emp => emp.Name);

    // sử dụng với LINQ query syntax
    var emp_2 = from emp in db.Employees.AsNoTracking()
                select emp.Name;
```

Hoặc ta có thể thay đổi hành vi theo dõi (Tracking Behavior) mặc định bằng đối tượng Context.

**Ví dụ:**

```cs
    var db = new BusinessContext();
    db.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;
```

Khi thay đổi hành vi theo dõi cho đối tượng Context như ví dụ trên, các truy vấn sẽ không theo dõi thực thể nữa. Để thực hiện truy vấn theo dõi, ta sẽ sử dụng phương thức `AsTracking()` trong câu truy vấn.

Phương thức `OnConfiguring()` trong lớp Context cũng có thể cấu hình hình vi theo dõi mặc định cho truy vấn.

```cs
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(@"<connection string>")
                      .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
    }
```

> [!Note]
> Nếu kết quả truy vấn không trả về thực thể ***hoàn chỉnh*** nào thì Entity Framework Core sẽ không theo dõi truy vấn đó.
> **Ví dụ:**
> ```csharp
>    var tracking = db.Employees.Select(e => new { Employee = e, DepName = e.Department.Name });
> 
>    var noTracking = db.Employees.Select(e => new { e.Name, e.Salary }); // no entity return
> ```
> Ở ví dụ trên, khi trả về kết quả `new { e.Name, e.Salary }`, không có thực thể hoàn chỉnh nào được trả về, vì vậy mà câu truy vấn này không được theo dõi.


## Load Related Data

EF Core cho phép ta sử dụng Navigation Property để tải và sử dụng các dữ liệu của (các) thực thể liên quan.

Đối với O/RM framework nói chung, có 3 cách để tải dữ liệu của (các) thực thể liên quan:

* **Eager loading**: tải tất cả dữ liệu của (các) thực thể liên quan ngay khi truy vấn.

* **Explicit loading**: tải tất cả dữ liệu của (các) thực thể liên quan sau truy vấn (lúc khác).

* **Lazy loading**: tải dữ liệu của (các) thực thể liên quan sẽ được hoãn lại cho đến khi có yêu cầu về dữ liệu đó.

Trước khi tìm hiểu về 3 cách tải dữ liệu trên, ta cùng tìm hiểu xem hành vi tải dữ liệu mặc định của câu truy vấn trong EF Core.

Ở nội dung này, ta sẽ sử dụng mối quan hệ 1 – N để thực hành tải dữ liệu, vì mối quan hệ này được sử dụng nhiều nhất và bao gồm cả trường hợp tải 1 thực thể và tải nhiều thực thể. Giả sử ta có 2 lớp thực thể sau:

```cs
    public class Employee
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public double Salary { get; set; }

        public int DepartmentId { get; set; } // foreign key property
        public Department Department { get; set; } // reference navigation
    }

    public class Department {
        public int Id { get; set; }
        public string Name { get; set; }

        public ICollection<Employee> Employees { get; set; } // collection navigation
    }
```

### Default Loading Behavior

Khi tải dữ liệu của một thực thể bằng truy vấn LINQ, các dữ liệu liên quan sẽ không được tải cùng dù là một 
thực thể (với Reference Navigation) hay nhiều thực thể (với Collection Property).

**Sau khi thực hiện truy vấn**, các Reference Navigation hay Collection Navigation đều có giá trị `null`, mặc dù khóa 
ngoại (đối với thực thể con) vẫn có giá trị.


**Ví dụ:**

```cs
    var db = new HumanResourceContext();
    var employee = db.Employees.Single(e => e.Id == 1);
    Console.WriteLine(employee.Department.Name); // exception, 'Department' is null
    Console.WriteLine(employee.DepartmentId); // OK
```

Ta sẽ nhận được kết quả của Reference Navigation `Department` là `null`, mặc dù thuộc tính khóa ngoại `DepartmentId` vẫn có giá trị.

EF Core cũng không thực hiện tải tự động các dữ liệu vào Collection Navigation (trong quan hệ 1 – N, N - N) tương tự như trên.

**Tuy nhiên**, nếu Reference Navigation hay Collection Navigation **được gọi ngay trong câu truy vấn**, EF Core bắt buộc phải tải dữ liệu cần thiết để hoàn thành câu truy vấn. Nhưng tương tự như trên, sau câu truy vấn thì các dữ liệu liên quan sẽ mất đi (trừ khi giữ chúng lại bằng `Select()` hoặc `select`).

**Ví dụ:**

```cs
    var db = new HumanResourceContext();

    // EF Core will load data to collection navigation
    var dep = db.Departments.Where(d => d.Employees.Count > 0).Select(d => d).First();

    Console.WriteLine(dep.Employees.Count); // exception, 'Employees' is null
```

Nếu muốn giữ lại các thực thể liên quan, ta cần trả về theo câu truy vấn với `Select()` hoặc `select`, ví dụ:

```cs
    var dep = from d in db.Departments
              where d.Employees.Count > 0 select new { d, d.Employees }; // trả về thực thể liên quan 'Employees'
```

> **Tóm lại, EF Core sẽ:**
>
> * Tự động tải dữ liệu liên quan nếu như phần dữ liệu đó được yêu cầu (được gọi thông qua 
Reference Navigation hay Collection Property). Các dữ liệu liên quan này không được trả về theo thực thể
chính (thực thể được truy vấn chính).
>
> * Không tự động tải dữ liệu liên quan nếu không được yêu cầu hoặc bên ngoài câu truy vấn.
Về cơ bản, hành vi mặc định này của EF Core tương tự cơ chế **Lazy loading**. Nội dung về Lazy loading ở
phần sau sẽ trình bày cách tải dữ liệu sau khi thực hiện truy vấn.

### Eager Loading

**Eager Loading** (*tạm dịch*: Tải tích cực) là cơ chế tải toàn bộ dữ liệu liên quan và trả về kết quả truy vấn.

Để chỉ định cơ chế Eager Loading, ta sử dụng phương thức `Include()` với cú pháp:

```cs
    Include<TEntity,TProperty>(Expression<Func<TEntity,TProperty>> navigationPropertyPath)
```

**Ví dụ:**

```cs
    var db = new HumanResourceContext();

    var emp = db.Employees.Single(e => e.Id == 1).Include(e => e.Department); // load 'Department' data

    Console.WriteLine(emp.Department.Name); // OK
```

Tuy nhiên, không có từ khóa nào thay thế cho phương thức `Include()`, vì vậy nếu sử dụng Query syntax, ta cần phải kết hợp với Method syntax.

**Ví dụ:**

```cs
    var department = (from d in db.Departments 
                      where d.Employees.Count > 0
                      select d).Include(d => d.Employees);
```
---
#### Multiple relationship including
---

Ta có thể tải nhiều dữ liệu liên quan cho từng mối quan hệ bằng cách gọi nhiều phương thức `Include()`.

Mỗi phương thức thức `Include()` sẽ tải các dữ liệu từ các mối quan hệ khác mà thực thể chính có.

