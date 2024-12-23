# Entity Framework Core 7.0 - Database First Approach


* Tài liệu gốc: **https://learn.microsoft.com/en-us/ef/core/**

* Tải và cài đặt một trong hai công cụ sau:
   
   * [**Visual Studio**](https://visualstudio.microsoft.com/vs/)
   
   * [**Visual Studio Code**](https://code.visualstudio.com/) và các extension được đề cập tại [**đây**](https://github.com/hoaiandnd/aps.net-core-web-api-vscode).

* Kiến thức yêu cầu:
    
    * [**LINQ To Objects**](https://github.com/hoaiandnd/language-intergrated-query)
    
    * LINQ To Entities (LINQ To SQL) - *Không bắt buộc*

---

Mục nội dung:

- [**Sơ lược về Entity Framework Core**](#s%C6%A1-l%C6%B0%E1%BB%A3c-v%E1%BB%81-entity-framework-core)

- [**Cài đặt các gói cần thiết**](#c%C3%A0i-%C4%91%E1%BA%B7t-c%C3%A1c-g%C3%B3i-c%E1%BA%A7n-thi%E1%BA%BFt)

- [**Chuẩn bị cơ sở dữ liệu**](#chu%E1%BA%A9n-b%E1%BB%8B-c%C6%A1-s%E1%BB%9F-d%E1%BB%AF-li%E1%BB%87u)

- [**Kết nối CSDL**](#k%E1%BA%BFt-n%E1%BB%91i-csdl)

---

## Sơ lược về Entity Framework Core

***Entity Framework Core*** (EF Core) là một phiên bản mã nguồn mở, gọn nhẹ, dễ mở rộng và đa nền tảng trong công nghệ truy cập dữ liệu Entity Framework.

EF Core Là sự nâng cấp so với ***Entity Framework 6 (EF6)*** – phiên bản cuối cùng dành cho .NET Framework, sau khi .NET Core ra đời.

Gần như những gì cơ bản nhất của EF6 được giữ lại ở EF Core: 

* Là một ORM[^1] Framework (Object/Relational Mapping).

* Hỗ trợ truy vấn LINQ.

[^1]: ORM (Object/Relational Mapping) là một nền tảng ánh xạ cơ sở dữ liệu với các lớp đối tượng, cho phép lập trình viên chỉ cần thao tác với đối tượng mà không cần yêu cầu nhiều kỹ năng về SQL.

Tuy nhiên, vẫn có một số sự khác nhau giữa EF6 và EF Core. Xem chi tiết tại: **https://learn.microsoft.com/vi-vn/ef/efcore-and-ef6/**

Phiên bản EF Core được giới thiệu mới nhất là EF Core 7.0 vào tháng 11/2022 dành cho .NET 6.

EF Core hoạt động trên nhiều loại cơ sở dữ liệu như SQL Server, MySQL, PostgreSQL, ... dựa vào các [**Database Providers**](https://learn.microsoft.com/vi-vn/ef/core/providers/?tabs=dotnet-core-cli) (nhà cung cấp cơ sở dữ liệu).

EF Core hỗ trợ 2 hướng tiếp cận: *Code-first* và *Database-First*. Trong nội dung này, ta sẽ tìm hiểu về cách tiếp cận Database-First.

## Cài đặt các gói cần thiết

Để tạo ứng dụng .NET với Entity Framework Core, ta cần cài đặt 2 gói NuGet sau:

* Gói EF Core Database Provider tương ứng. Với SQL Server là `Microsoft.EntityFrameworkCore.SqlServer`.

* Gói NuGet `Microsoft.EntityFrameworkCore.Tools` hoặc `Microsoft.EntityFrameworkCore.Design`.

Tùy vào cơ sở dữ liệu muốn sử dụng mà gói Database Provider cần cài đặt sẽ khác nhau, vì vậy hãy tìm kiếm các [**gói Database Provider**](https://learn.microsoft.com/vi-vn/ef/core/providers/?tabs=dotnet-core-cli) tương ứng.

### Cài đặt bằng Package Manager Console - PMC

* Chọn **Tools > NuGet Package Manager > Package Manager Console** để mở PMC.
* Sử dụng lệnh `Install-Package` để cài đặt các gói NuGet mới nhất tìm thấy ở [**NuGet Gallery**](https://www.nuget.org/):
```console
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
```
* Nếu muốn tải các gói ở phiên bản cụ thể, hãy sử dụng tham số `-Version <version>`.

**Ví dụ:**
```console
Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 6.0.10
```

### Cài đặt bằng Manage NuGet Packages

Có 3 cách để mở Manage NuGet Packages:
* Chọn **Tools > NuGet Package Manager > Manage NuGet Packages for Solution ...**.
* Right-click vào tên Solution, chọn **Manage NuGet Packages for Solution ...**.
* Right-click vào tên Project, chọn **Manage NuGet Packages ...**.

Chọn tab **Browse** và tìm kiếm và tải 2 gói được nêu ở trên.

> [!Note]
>  Nếu đã tải các gói từ NuGet về máy, hãy đưa chúng vào thư mục `NuGet Packages` theo đường dẫn `C:\Program Files (x86)\Microsoft SDKs\NuGetPackages`.

### Cài đặt trong Visual Studio Code

Visual Studio Code có đồng thời có 2 cách tương ứng với Visual Studio:

- Sử dụng .NET CLI: Dùng lệnh `dotnet add package`

```console
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

- Sử dụng giao diện từ extension [**NuGet Gallery**](https://marketplace.visualstudio.com/items?itemName=patcx.vscode-nuget-gallery): Mở `Intergrated Terminal` trong Visual Studio Code và chuyển sang tab `NUGET`

![image](https://github.com/user-attachments/assets/0d598b91-10aa-448f-bbc4-c1b48fd4be1b)

## Chuẩn bị cơ sở dữ liệu

Với hướng tiếp cận Database First, ta cần có sẵn cơ sở dữ liệu.

Ở đây, ta sử dụng cơ sở dữ liệu đơn giản như sau:

| NhanVien | PhongBan |
| :---: | :---: |
| **manv** |  **mapb** |
| tennv | tenpb |
| luong | |
| mapb | |

Đoạn mã sau sẽ tạo và cung cấp các dữ liệu giả lập:
```SQL
    CREATE DATABASE QLNhanSu
    GO
    USE QLNhanSu
    GO
    CREATE TABLE NhanVien (
        manv VARCHAR(10) PRIMARY KEY,
        tennv NVARCHAR(50),
        luong DECIMAL,
        mapb VARCHAR(10)
    )
    GO
    CREATE TABLE PhongBan (
        mapb VARCHAR(10) PRIMARY KEY,
        tenpb NVARCHAR(50)
    )
    GO
    ALTER TABLE NhanVien ADD CONSTRAINT FK_NV_PB FOREIGN KEY(mapb) REFERENCES PhongBan(mapb)
    GO

    -- dữ liệu giả lập cho phòng ban
    INSERT INTO PhongBan VALUES('PB001', 'Phòng ban 1')
    INSERT INTO PhongBan VALUES('PB002', 'Phòng ban 2')
    INSERT INTO PhongBan VALUES('PB003', 'Phòng ban 3')
    INSERT INTO PhongBan VALUES('PB004', 'Phòng ban 4')

    -- dữ liệu giả lập cho nhân viên
    INSERT INTO NhanVien VALUES('NV00001', N'Nguyễn Văn A', 15000, 'PB001')
    INSERT INTO NhanVien VALUES('NV00002', N'Lê Thị B', 14500, 'PB002')
    INSERT INTO NhanVien VALUES('NV00003', N'Trần Thị C', 15050, 'PB003')
    INSERT INTO NhanVien VALUES('NV00004', N'Võ Văn D', 12000, 'PB004')
    INSERT INTO NhanVien VALUES('NV00005', N'Nguyễn Thị E', 5000, 'PB001')
    INSERT INTO NhanVien VALUES('NV00006', N'Trần Văn F', 3500, 'PB001')
    INSERT INTO NhanVien VALUES('NV00007', N'Đoàn Thị G', 15100, 'PB003')
    INSERT INTO NhanVien VALUES('NV00008', N'Nguyễn Văn H', 1790, 'PB004')
    INSERT INTO NhanVien VALUES('NV00009', N'Nguyễn I', 2000, 'PB002')
    INSERT INTO NhanVien VALUES('NV00010', N'Trần Văn J', 25000, 'PB001')
    INSERT INTO NhanVien VALUES('NV00011', N'Nguyễn Ngọc K', 35600, 'PB003')
    INSERT INTO NhanVien VALUES('NV00012', N'Ngô Thị L', 15000, 'PB001')
    INSERT INTO NhanVien VALUES('NV00013', N'Nguyễn Toàn M', 17000, 'PB004')
```

## Kết nối CSDL

### Visual Studio

Do EF Core được tạo ra nhằm phục vụ cho hướng tiếp cận Code-first, nên hướng tiếp cận Database-first không được hỗ trợ nhiều và đặc biệt là không thể thao tác với giao diện như EF6.

Để tiếp cận theo hướng Database-first, ta sẽ sử dụng lệnh `Scaffold-DbContext` trong **Package Manager Console**.

Lệnh `Scaffold-DbContext` có nhiều tham số dùng để chỉ định các thông tin kết nối. Ở mức độ cơ bản, ta thường dùng các tham số sau:

| Tham số | Mô tả |
| --- | --- |
| `-Connection <string>` | Chuỗi kết nối (Connection string) đến CSDL cần kết nối. **Đây là tham số bắt buộc và là tham số vị trí** |
| `-Provider <string>` | Tên gói Database Provider đang dùng. **Đây là tham số bắt buộc và là tham số vị trí** |
| `-OutputDir <string>` | Thư mục / đường dẫn chứa các lớp thực thể khi tạo ra từ CSDL. Mặc định là đường dẫn gốc của Project |
| `-ContextDir <string>` | Thư mục / đường dẫn chứa lớp Context. Mặc định dùng chung với `-OutputDir` (nếu có) hoặc đường dẫn gốc của Project |

> Xem thêm các tham số khác tại [**Scaffold-DbContext**](https://learn.microsoft.com/en-us/ef/core/cli/powershell#scaffold-dbcontext).

### Visual Studio Code

Đối với .NET CLI, ta sử dụng lệnh `dotnet ef dbcontext scaffold` với cú pháp:

```console
dotnet ef dbcontext scaffold <CONNECTION_STRING> <DATABASE_PROVIDER> [OPTIONS]
```

Trong đó:

- `<CONNECTION_STRING>` là chuỗi kết nối đến CSDL.

- `<DATABASE_PROVIDER>` là database provider tương ứng với CSDL cần kết nối.

- `[OPTIONS]` là các tham số tuỳ chọn có thể liệt kê một số như sau:

| Tham số | Alias | Mô tả |
| --- | --- | --- |
| `--data-annotations` | `-d` | Sử dụng [**Data annotation**](https://github.com/hoaiandnd/entity-framework-core-7/blob/main/3_efcore7_code_first_approach_map_attributes.md) để cấu hình các lớp thực thể khi có thể. Nếu bỏ qua sẽ sử dụng [**Fluent API**](https://github.com/hoaiandnd/entity-framework-core-7/blob/main/6_efcore7_code_first_approach_fluent_api.md) để cấu hình. |
| `--context <NAME>` | `-c` | Tên [**lớp context**](https://github.com/hoaiandnd/entity-framework-core-7/blob/main/4_efcore7_code_first_approach_context_class.md) sẽ được tạo ra. |
| `--output-dir <PATH>`| `-o` | Thư mục / đường dẫn chứa các lớp thực thể khi tạo ra từ CSDL. Mặc định là đường dẫn gốc của Project. |
| `--context-dir <PATH>`| | Thư mục / đường dẫn chứa lớp Context. Mặc định dùng chung với `--output-dir` (nếu có) hoặc đường dẫn gốc của Project. |

### Connection string

Để lấy được chuỗi kết nối một cách đơn giản trong Visual Studio, ta có thể thực hiện các bước sau:

* Chọn **Views > Server Explorer** hoặc sử dụng tổ hợp phím `Ctrl` + `Alt` + ``` ` ```.
   
* Right-click **Data Connection > Add Connection ...**.
     
* Chỉ định tên Server (Server name) và chọn Database. Có thể dùng lệnh `PRINT @@SERVERNAME` trong SQL Server để xem tên Server.
     
* Chọn **Advanced ...**, chuỗi kết nối được hiển thị phía cuối hộp thoại. Sao chép chuỗi nhận được và hủy các thao tác vừa thực hiện.

Với chuỗi kết nối nhận được, ta đã có thể dùng lệnh `Scaffold-DbContext` để kết nối với CSDL.

**Ví dụ:**

* Không chỉ định đường dẫn cho các lớp thực thể và lớp Context:

```console
   Scaffold-DbContext 'Data Source=.\sqlexpress;Initial Catalog=QLNhanSu;Integrated Security=True' Microsoft.EntityFrameworkCore.SqlServer
```

* Chỉ định đường dẫn cho các lớp thực thể và lớp Context

```console
   Scaffold-DbContext 'Data Source=.\sqlexpress;Initial Catalog=QLNhanSu;Integrated Security=True;' Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -ContextDir Data
```

> [!WARNING]
> ***Vấn đề với Entity Framework Core 7.0***
> 
> Nếu sử dụng lệnh `Scaffold-DbContext`, từ EF Core 7.0 ta sẽ nhận về thông báo lỗi sau khi gọi lệnh `Scaffold-DbContext`.
>
> ***A connection was successfully established with the server, but then an error occurred during the login process. (provider: SSL Provider, error: 0 - The certificate chain was issued by an authority that is not trusted.)***
>
> Để giải quyết lỗi trên, ta có thể thực hiện 1 trong 3 cách sau đây:
> * Cài đặt một chứng chỉ hợp lệ trên máy chủ. Cách làm này rất phức tạp vì cần một nhà cung cấp dịch vụ có thẩm quyền.
>
> * Thêm `TrustServerCertificate=True` vào chuỗi kết nối.
>
> * Thêm `Encrypt=False` vào chuỗi kết nối.
>
> Để xem thêm các vấn đề liên quan, hãy xem [**Certificate Issue**](https://learn.microsoft.com/en-us/troubleshoot/sql/database-engine/connect/certificate-chain-not-trusted?tabs=odbc-driver-18x).
