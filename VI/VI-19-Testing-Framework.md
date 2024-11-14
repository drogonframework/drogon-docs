## Khung Kiểm thử

`DrogonTest` là một khung kiểm thử tối thiểu được tích hợp vào Drogon để cho phép kiểm tra bất đồng bộ dễ dàng cũng như các kiểm tra đồng bộ. Nó được sử dụng cho các bài kiểm tra đơn vị và kiểm tra tích hợp của riêng Drogon. Nhưng cũng có thể được sử dụng để kiểm tra các ứng dụng được xây dựng bằng Drogon. Cú pháp của DrogonTest được lấy cảm hứng từ cả [GTest](https://github.com/google/googletest) và [Catch2](https://github.com/catchorg/Catch2).

Bạn không phải sử dụng DrogonTest cho ứng dụng của mình. Sử dụng bất cứ thứ gì bạn cảm thấy thoải mái. Nhưng nó là một lựa chọn.

### Kiểm tra cơ bản

Hãy bắt đầu với một ví dụ đơn giản. Bạn có một hàm đồng bộ tính tổng các số tự nhiên lên đến một giá trị nhất định. Và bạn muốn kiểm tra nó để biết tính chính xác.

```c++
// Yêu cầu DrogonTest tạo `test::run()`. Chỉ được xác định điều này trong tệp chính
#define DROGON_TEST_MAIN
#include <drogon/drogon_test.h>

int sum_all(int n)
{
    int result = 1;
    for(int i=2;i<n;i++) result += i;
    return result;
}

DROGON_TEST(Sum)
{
    CHECK(sum_all(1) == 1);
    CHECK(sum_all(2) == 3);
    CHECK(sum_all(3) == 6);
}

int main(int argc, char** argv)
{
    return drogon::test::run(argc, argv);
}
```

Biên dịch và chạy... Chà, nó đã vượt qua nhưng có một lỗi rõ ràng phải không. `sum_all(0)` lẽ ra phải là 0. Chúng ta có thể thêm điều đó vào bài kiểm tra của mình:

```c++
DROGON_TEST(Sum)
{
    CHECK(sum_all(0) == 0);
    CHECK(sum_all(1) == 1);
    CHECK(sum_all(2) == 3);
    CHECK(sum_all(3) == 6);
}
```

Bây giờ bài kiểm tra không thành công với:

```
In test case Sum
↳ /đường_dẫn/đến/tệp/kiểm_thử/của/bạn/main.cc:47  FAILED:
  CHECK(sum_all(0) == 0)
With expansion
  1 == 0
```

Lưu ý framework đã in bài kiểm tra không thành công và giá trị thực tế ở cả hai đầu của biểu thức. Cho phép chúng tôi thấy điều gì đang xảy ra ngay lập tức. Và giải pháp rất đơn giản:

```c++
int sum_all(int n)
{
    int result = 0;
    for(int i=1;i<n;i++) result += i;
    return result;
}
```

### Các loại khẳng định

DrogonTest đi kèm với nhiều khẳng định và hành động khác nhau. `CHECK()` cơ bản chỉ kiểm tra xem biểu thức có được đánh giá là đúng (true) hay không. Nếu không, nó sẽ in ra console. `CHECK_THROWS()` kiểm tra xem biểu thức có ném ra ngoại lệ hay không. Nếu không, in ra console. Vân vân.. Mặt khác, `REQUIRE()` kiểm tra xem một biểu thức có đúng hay không. Sau đó trả về nếu không, ngăn chặn các biểu thức sau khi kiểm tra được thực thi.

| Hành động nếu thất bại/biểu thức | is true    | throws            | does not throw     | throws certain type  |
| ------------------------- | ---------- | ----------------- | ------------------ | -------------------- |
| nothing                   | `CHECK`      | `CHECK_THROWS`      | `CHECK_NOTHROW`      | `CHECK_THROWS_AS`      |
| return                    | `REQUIRE`    | `REQUIRE_THROWS`    | `REQUIRE_NOTHROW`    | `REQUIRE_THROWS_AS`    |
| co_return                 | `CO_REQUIRE` | `CO_REQUIRE_THROWS` | `CO_REQUIRE_NOTHROW` | `CO_REQUIRE_THROWS_AS` |
| kill process              | `MANDATE`    | `MANDATE_THROWS`    | `MANDATE_NOTHROW`    | `MANDATE_THROWS_AS`    |

Hãy thử một ví dụ thực tế hơn một chút. Giả sử bạn đang kiểm tra xem nội dung của một tệp có phải là thứ bạn mong đợi hay không. Không có điểm nào để kiểm tra thêm nếu chương trình không mở được tệp. Vì vậy, chúng ta có thể sử dụng `REQUIRE` để rút ngắn và giảm mã trùng lặp.

```c++
DROGON_TEST(TestContent)
{
    std::ifstream in("data.txt");
    REQUIRE(in.is_open());
    // Thay vì
    // CHECK(in.is_open() == true);
    // if(in.is_open() == false)
    //    return;

    ...
}
```

Tương tự như vậy, `CO_REQUIRE` giống như `REQUIRE`. Nhưng dành cho coroutine. Và `MANDATE` có thể được sử dụng khi một thao tác không thành công và nó sửa đổi trạng thái toàn cục không thể phục hồi. Trong trường hợp đó, điều hợp lý duy nhất cần làm là dừng kiểm tra hoàn toàn.

### Kiểm tra bất đồng bộ

Drogon là một framework web bất đồng bộ. DrogonTest hỗ trợ kiểm tra các hàm bất đồng bộ. DrogonTest theo dõi ngữ cảnh kiểm tra thông qua biến `TEST_CTX`. Chỉ cần capture biến **theo giá trị**. Ví dụ: kiểm tra xem API từ xa có thành công hay không và trả về JSON.

```c++
DROGON_TEST(RemoteAPITest)
{
    auto client = HttpClient::newHttpClient("http://localhost:8848");
    auto req = HttpRequest::newHttpRequest();
    req->setPath("/");
    client->sendRequest(req, [TEST_CTX](ReqResult res, const HttpResponsePtr& resp) {
        // Chúng ta không thể làm gì nếu yêu cầu không đến được máy chủ
        // hoặc máy chủ tạo ra rác.
        REQUIRE(res == ReqResult::Ok);
        REQUIRE(resp != nullptr);

        CHECK(resp->getStatusCode() == k200OK);
        CHECK(resp->contentType() == CT_APPLICATION_JSON);
    });
}
```

Coroutine phải được bao bọc bên trong `AsyncTask` hoặc được gọi thông qua `sync_wait` do không có hỗ trợ nguyên bản của coroutine và khả năng tương thích C++14/17 trong framework kiểm tra.

```c++
DROGON_TEST(RemoteAPITestCoro)
{
    auto api_test = [TEST_CTX]() {
        auto client = HttpClient::newHttpClient("http://localhost:8848");
        auto req = HttpRequest::newHttpRequest();
        req->setPath("/");

        auto resp = co_await client->sendRequestCoro(req);
        CO_REQUIRE(resp != nullptr);
        CHECK(resp->getStatusCode() == k200OK);
        CHECK(resp->contentType() == CT_APPLICATION_JSON);
    };

    sync_wait(api_test());
}
```

### Khởi động vòng lặp sự kiện của Drogon

Một số bài kiểm tra cần vòng lặp sự kiện của Drogon đang chạy. Ví dụ: trừ khi được chỉ định, client HTTP chạy trên vòng lặp sự kiện toàn cầu của Drogon. Boilerplate sau xử lý nhiều trường hợp edge case và đảm bảo vòng lặp sự kiện đang chạy trước khi bất kỳ bài kiểm tra nào bắt đầu.

```c++
int main(int argc, char** argv)
{
    std::promise<void> p1;
    std::future<void> f1 = p1.get_future();

    // Khởi động vòng lặp chính trên một luồng khác
    std::thread thr([&]() {
        // Xếp hàng lời hứa được thực hiện sau khi bắt đầu vòng lặp
        app().getLoop()->queueInLoop([&p1]() { p1.set_value(); });
        app().run();
    });

    // Future chỉ được đáp ứng sau khi vòng lặp sự kiện bắt đầu
    f1.get();
    int status = test::run(argc, argv);

    // Yêu cầu vòng lặp sự kiện tắt và chờ
    app().getLoop()->queueInLoop([]() { app().quit(); });
    thr.join();
    return status;
}
```

### Tích hợp CMake

Giống như hầu hết các framework kiểm tra, DrogonTest có thể tự tích hợp vào CMake. Hàm `ParseAndAddDrogonTests` thêm các bài kiểm tra mà nó thấy trong tệp nguồn vào framework CTest của CMake.

```cmake
find_package(Drogon REQUIRED) # cũng tải ParseAndAddDrogonTests
add_executable(mytest main.cpp)
target_link_libraries(mytest PRIVATE Drogon::Drogon)
ParseAndAddDrogonTests(mytest)
```

Bây giờ bài kiểm tra có thể được chạy thông qua hệ thống build (Makefile trong trường hợp này).

```bash
❯ make test
Running tests...
Test project /đường_dẫn/đến/project/kiểm_thử/của/bạn/build/
      Start  1: Sum
 1/1  Test  #1: Sum ....................................   Passed    0.00 sec
```

