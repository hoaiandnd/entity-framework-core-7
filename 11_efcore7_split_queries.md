# Split queries

## Single queries

Khi giao tiếp với các cơ sở dữ liệu quan hệ (relational database), để tải các thực thể liên quan thì EF Core sẽ sử dụng lệnh JOIN trong 1 câu query duy nhất. Tuy nhiên, điều đó đôi khi sẽ gây ra các vấn đề nghiêm trọng về mặt hiệu suất nếu sử dụng không đúng cách.

### Cartesian explosion

