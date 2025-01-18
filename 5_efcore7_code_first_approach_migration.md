# Entity Framework Core 7.0 - Code First Approach - Migration

**Migration** là một tính năng của EF Core cung cấp cách cập nhật từng bước lược đồ quan hệ (Database 
Schema) để giữ nó được đồng bộ với dữ liệu của ứng dụng trong khi vẫn bảo quản các dữ liệu sẵn có từ
CSDL.

EF Core Migration là tập các lệnh có thể thực thi trong **Package Manager Console** trong Visual Studio. Các nội 
dung bên dưới sẽ lần lượt những lệnh có thể dùng trong EF Core Migration.

## Tạo Migration đầu tiên và Database

Sau khi thực hiện xong các cấu hình trên lớp thực thể và đối tượng Context, ta sử dụng lệnh: 

- **Package Manager Console**:

```console
Add-Migration [Migration name]
```

- **.NET CLI**:

```console
dotnet ef migrations add [Migration name]
```

Trong đó, `[Migration name]` là một định danh tùy chọn, tuy nhiên định danh này sẽ được dùng làm tên `class`, vì vậy nếu được hãy đặt theo định dạng **PascalCase**.

EF Core sẽ tự động tạo ra thư mục tên `Migrations` trong dự án hiện tại cùng với một số file `*.cs`.

**Ví dụ:**

```console
    Add-Migration dbcreation
```

Trong đó, các file sẽ được tạo trong thư mục `Migration`:

* `<timestamp>_<migration name>.cs`: là file Migration chính chứa 2 phương thức `Up()` và `Down()`.

    * Phương thức `Up()` dùng để tạo hoặc cập nhật CSDL với lệnh `Update-Database`.

    * Phương thức `Down()` được dùng để hoàn tác lại thao tác của `Up()` bằng các sử dụng lệnh `Update-Database [Migration name]`.

* `<timestamp>_<migration name>.Design.cs`: Chứa các thông tin được sử dụng bởi EF Core.

* `<context class>ModelSnapshot.cs`: chứa snapshot cho các thực thể hiện tại, dùng để xác định sự thay đổi khi tạo một Migration khác.

Lúc này, CSDL và các bảng (tương ứng với các lớp thực thể) vẫn chưa được tạo ra. Để tạo Database, ta sẽ tiếp tục sử dụng lệnh sau:

```console
    Update-Database
```

Lệnh `Update-Database` sẽ gọi phương thức `Up()` của lớp Migration gần nhất.
Như vậy, CSDL đã sẵn sàng sử dụng cho ứng dụng hiện tại, mặc dù chưa có dữ liệu trên các bảng.

## Cập nhật lớp thực thể

Khi phát triển ứng dụng, đôi khi ta sẽ khai báo thêm/loại bỏ một hoặc một số thuộc tính cho lớp thực thể. 
Trong trường hợp đó, lớp thực thể và bảng dữ liệu đã không còn đồng bộ nữa.

Để thực hiện tái đồng bộ, ta sẽ tạo ra một Migration mới và cập nhật lại CSDL:

```console
    Add-Migration [New migration name]
    Update-Database
```

## Xóa Migration gần nhất

Để xóa Migration gần nhất (được tạo lần cuối) trong trường hợp muốn hiệu chỉnh lại các lớp thực thể trước 
khi xác nhận, ta có thể sử dụng lệnh sau:

```console
    Remove-Migration
```

> [!WARNING]
> Ta chỉ có thể xóa Migration gần nhất nếu như Migration đó chưa được xác nhận, chưa được ghi 
nhận trong CSDL (chưa gọi lệnh `Update-Database`). Nếu không, ta sẽ nhận được thông báo lỗi tương tự như
sau:
> 
> *The migration '...' has already been applied to the database. Revert it and try again. If the migration has been applied to other databases, consider reverting its changes using a new migration instead.*

## Cập nhật CSDL theo Migration chỉ định - Revert migration

Lệnh `Update-Database` được dùng để cập nhật vào CSDL theo Migration gần nhất.

Để cập nhật CSDL theo Migration được chỉ định, ta có thể chỉ định thêm tên Migration.

```console
    Update-Database [Migration name]
```

Trong đó, `[Migration name]` là phần tên của Migration (không bao gồm phần timestamp phía trước).

> [!NOTE]
>
> * Sau khi cập nhật, Migration được chỉ định không trở thành Migration gần nhất.
>
> * Toàn bộ các phương thức `Down()` từ Migration hiện tại đến Migration được chỉ định sẽ được gọi và thực thi (Rollback).



