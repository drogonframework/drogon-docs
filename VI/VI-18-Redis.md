## Redis

Drogon hỗ trợ Redis, một kho dữ liệu trong bộ nhớ rất nhanh. Có thể được sử dụng như một bộ nhớ đệm cơ sở dữ liệu hoặc một trình trung chuyển thông báo (message broker). Giống như mọi thứ trong Drogon, các kết nối Redis là bất đồng bộ. Điều này đảm bảo Drogon chạy với tính đồng thời rất cao ngay cả khi tải nặng.

Hỗ trợ Redis phụ thuộc vào thư viện `hiredis`. Hỗ trợ Redis sẽ không khả dụng nếu `hiredis` không khả dụng khi build Drogon.

### Tạo client

Client Redis có thể được tạo và truy xuất một cách linh hoạt thông qua `app()`.

```c++
app().createRedisClient("127.0.0.1", 6379);
...
// Sau app().run()
RedisClientPtr redisClient = app().getRedisClient();
```

Client Redis cũng có thể được tạo thông qua tệp cấu hình.

```json
    "redis_clients": [
        {
            // name: Tên của client, mặc định là 'default'
            //"name": "",
            // host: IP máy chủ, mặc định là 127.0.0.1
            "host": "127.0.0.1",
            // port: Cổng máy chủ, mặc định là 6379
            "port": 6379,
            // passwd: '' theo mặc định
            "passwd": "",
            // db: chỉ mục cơ sở dữ liệu, mặc định là 0
            "db": 0,
            // is_fast: false theo mặc định, nếu là true, client sẽ nhanh hơn nhưng người dùng không thể gọi bất kỳ giao diện đồng bộ nào của nó và không thể sử dụng nó bên ngoài các luồng I/O và luồng chính.
            "is_fast": false,
            // number_of_connections: 1 theo mặc định, nếu 'is_fast' là true, số lượng là số lượng kết nối cho mỗi luồng I/O, nếu không thì đó là tổng số tất cả các kết nối.
            "number_of_connections": 1,
            // timeout: -1.0 theo mặc định, tính bằng giây, thời gian chờ để thực thi lệnh.
            // giá trị bằng không hoặc âm có nghĩa là không có thời gian chờ.
            "timeout": -1.0
        }
    ]
```

### Sử dụng Redis

`execCommandAsync` thực thi các lệnh Redis một cách bất đồng bộ. Nó nhận ít nhất 3 tham số, tham số đầu tiên và thứ hai là callback sẽ được gọi khi lệnh Redis thành công hoặc thất bại. Thứ ba là bản thân lệnh. Lệnh có thể là một chuỗi định dạng kiểu C. Và phần còn lại là đối số cho chuỗi định dạng. Ví dụ: để đặt khóa `name` thành `drogon`:

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {},
    [](const std::exception &err) {
        LOG_ERROR << "Có lỗi xảy ra!!! " << err.what();
    },
    "set name drogon");
```

Hoặc đặt `myid` thành `587d-4709-86e4`

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {},
    [](const std::exception &err) {
        LOG_ERROR << "Có lỗi xảy ra!!! " << err.what();
    },
    "set myid %s", "587d-4709-86e4");
```

Cùng một `execCommandAsync` cũng có thể truy xuất dữ liệu từ Redis.

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {
        if (r.type() == RedisResultType::kNil)
            LOG_INFO << "Không thể tìm thấy biến được liên kết với khóa 'name'";
        else
            LOG_INFO << "Name is " << r.asString();
    },
    [](const std::exception &err) {
        LOG_ERROR << "Có lỗi xảy ra!!! " << err.what();
    },
    "get name");
```

### Giao dịch

Giao dịch Redis cho phép nhiều lệnh được thực thi trong một bước duy nhất. Tất cả các lệnh trong một giao dịch được thực thi theo thứ tự, không có lệnh nào của các client khác có thể được thực thi **ở giữa** của một giao dịch. Lưu ý rằng một giao dịch không phải là nguyên tử. Điều này có nghĩa là sau khi nhận được lệnh `EXEC`, giao dịch sẽ được thực thi, Nếu bất kỳ lệnh nào trong giao dịch không thực thi được, các lệnh còn lại vẫn sẽ được thực thi. Giao dịch Redis không hỗ trợ các hoạt động rollback.

Phương thức `newTransactionAsync` tạo một giao dịch mới. Sau đó, giao dịch có thể được sử dụng giống như một `RedisClient` thông thường. Cuối cùng, phương thức `RedisTransaction::execute` thực thi giao dịch đã nói.

```c++
redisClient->newTransactionAsync([](const RedisTransactionPtr &transPtr) {
    transPtr->execCommandAsync(
        [](const drogon::nosql::RedisResult &r) { /* Lệnh này hoạt động */ },
        [](const std::exception &err) { /* Lệnh này thất bại */ },
        "set name drogon");

    transPtr->execute(
        [](const drogon::nosql::RedisResult &r) { /* Giao dịch hoạt động */ },
        [](const std::exception &err) { /* Giao dịch thất bại */ });
});
```

### Coroutine

Client Redis hỗ trợ coroutine. Người ta nên sử dụng GCC 11 hoặc trình biên dịch mới hơn và sử dụng `cmake -DCMAKE_CXX_FLAGS="-std=c++20"` để bật nó. Xem phần (coroutine) [VI-16-Coroutines] để biết thêm thông tin.

```c++
try
{
    auto transaction = co_await redisClient->newTransactionCoro();
    co_await transaction->execCommandCoro("set zzz 123");
    co_await transaction->execCommandCoro("set mening 42");
    co_await transaction->executeCoro();
}
catch(const std::exception& e)
{
    LOG_ERROR << "Redis thất bại: " << e.what();
}
```

# 18 [Khung Kiểm thử](VI-18-Testing-Framework)


