# Split queries

## Single queries

Khi giao tiếp với các cơ sở dữ liệu quan hệ (relational database), để tải các thực thể liên quan thì EF Core sẽ sử dụng lệnh JOIN trong 1 câu query duy nhất. Tuy nhiên, điều đó đôi khi sẽ gây ra các vấn đề nghiêm trọng về mặt hiệu suất nếu sử dụng không đúng cách. Và vấn đề ta tìm hiểu trong nội dung này là **Cartesian explosion**.

Xét đoạn truy vấn EF sau:

```js
var blogs = ctx.Blogs
    .Include(b => b.Posts)
    .Include(b => b.Contributors)
    .ToList();
```

Mã SQL tương ứng được tạo ra:

```sql
SELECT b.Id, b.Name, p.Id, p.BlogId, p.Name, c.Id, c.BlogId
FROM Blogs AS b
LEFT JOIN Posts AS p ON b.Id = p.BlogId
LEFT JOIN Contributors AS c ON b.Id = c.BlogId
ORDER BY b.Id, p.Id
```

Trong ví dụ trên, `Posts` và `Contributors` đều là các collection navigation của `Blogs`, và chúng có **cùng cấp bậc**. Cơ sở dữ liệu sẽ thực hiện join như sau:

<img src="https://github.com/user-attachments/assets/b4ba899a-2053-40d7-9437-9306285bcb4f" width="500px" />

Ta có thể thấy, với mỗi dòng của `Posts` sẽ join với tất cả các dòng `Contributors` và tương tự với các `Posts` còn lại.

Nếu một blog chứa 10 post và 10 contributors thì có đến 100 dòng dữ liệu sẽ được trả về. Hiện tượng này đôi khi sẽ được biết là Cartesian explosion (vụ nổ Đề-các).

Tuy nhiên, hiện tượng Cartesian explosion không xảy ra nếu thực hiện join trên các thực thể không cùng cấp bậc:

```js
var blogs = ctx.Blogs
    .Include(b => b.Posts)
    .ThenInclude(p => p.Comments)
    .ToList();
```

`Comments` không cùng cấp với `Posts`. Với mỗi bản ghi trả về sẽ tương ứng với 1 comment trong post, và điều đó là hợp lý.

