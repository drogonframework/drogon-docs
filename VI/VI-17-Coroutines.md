## Coroutines

Drogon hỗ trợ [C++ coroutines][1] bắt đầu từ phiên bản 1.4. Chúng cung cấp một cách để làm phẳng luồng điều khiển của các lệnh gọi bất đồng bộ, tức là thoát khỏi địa ngục callback. Với nó, lập trình bất đồng bộ trở nên dễ dàng như lập trình đồng bộ.

### Thuật ngữ

Trang này không nhằm mục đích giải thích coroutine là gì hay cách thức hoạt động của nó. Mục tiêu là chỉ ra cách sử dụng coroutine trong Drogon. Thuật ngữ thông thường có xu hướng trở nên lộn xộn vì các chương trình con (hàm) sử dụng cùng thuật ngữ như coroutine, nhưng chúng có ý nghĩa hơi khác nhau. Việc coroutine C++ có thể hoạt động như thể chúng là các hàm cũng không giúp ích gì. Để giảm bớt sự nhầm lẫn, chúng ta sẽ sử dụng thuật ngữ sau - nó không hoàn hảo, nhưng nó đủ tốt.

**Coroutine** là một hàm có thể tạm dừng thực thi rồi tiếp tục. <br/>
**Return** có nghĩa là một hàm kết thúc thực thi và cung cấp giá trị trả về cho hàm gọi nó. Hoặc một coroutine tạo ra một đối tượng *có thể tiếp tục* (resumable); có thể được sử dụng để tiếp tục coroutine. <br/>
**Yield** là khi một coroutine tạo ra kết quả cho người gọi. <br/>
**co-return** có nghĩa là một coroutine yield và sau đó thoát. <br/>
**(co-)await** có nghĩa là luồng đang chờ một coroutine yield. Framework có thể tự do sử dụng luồng cho mục đích khác trong khi chờ đợi. <br/>

### Kích hoạt coroutine

Tính năng coroutine trong Drogon chỉ là tiêu đề (header-only). Điều này có nghĩa là ứng dụng có thể sử dụng coroutine ngay cả khi Drogon được build mà không có hỗ trợ coroutine. Cách bật coroutine phụ thuộc vào trình biên dịch được sử dụng. Trong GCC >= 10, nó có thể được bật bằng cách đặt `-std=c++20 -fcoroutines` trong khi với MSVC (đã thử nghiệm trên MSVC 19.25), nó có thể được bật với `/std:c++latest` và `/await` không được đặt.

Lưu ý rằng việc triển khai coroutine của Drogon sẽ không hoạt động trên Clang (kể từ Clang 12.0). GCC 11 bật coroutine theo mặc định khi C++20 được bật. Và mặc dù GCC 10 có biên dịch coroutine, nhưng nó chứa một lỗi trình biên dịch khiến các khung (frame) coroutine lồng nhau không được giải phóng; dẫn đến rò rỉ bộ nhớ tiềm ẩn.

### Sử dụng coroutine

Mỗi coroutine trong Drogon đều được thêm hậu tố `Coro`. Ví dụ: `db->execSqlSync()` trở thành `db->execSqlCoro()`. `client->sendRequest()`  trở thành `client->sendRequestCoro()`, v.v. Tất cả các coroutine trả về một đối tượng *awaitable*. Sau đó, `co_await` trên đối tượng dẫn đến một giá trị. Framework có thể tự do sử dụng luồng để xử lý I/O và các tác vụ khi nó đang chờ kết quả đến - đó là vẻ đẹp của coroutine. Mã trông đồng bộ; nhưng thực tế nó bất đồng bộ.

Ví dụ: truy vấn số lượng người dùng tồn tại trong cơ sở dữ liệu:

```c++
app.registerHandler("/num_users",
    [](HttpRequestPtr req, std::function<void(const HttpResponsePtr&)> callback) -> Task<>
    //                                     Phải đánh dấu kiểu trả về là *có thể tiếp tục* ^^^
{
    auto sql = app().getDbClient();
    try
    {
        auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
        size_t num_users = result[0][0].as<size_t>();
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(std::to_string(num_users));
        callback(resp);
    }
    catch(const DrogonDbException &err)
    {
        // Ngoại lệ hoạt động như các giao diện đồng bộ.
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(err.base().what());
        callback(resp);
    }
    // Không cần trả về bất cứ thứ gì! Đây là một coroutine trả về `void`.
    // được chỉ định bởi kiểu trả về của Task<void>
    co_return; // Nếu muốn (không bắt buộc), hãy sử dụng co_return
}
```

Lưu ý một số điểm quan trọng:

1.  Bất kỳ handler nào gọi một coroutine PHẢI trả về một *có thể tiếp tục*
    - Biến chính handler thành một coroutine
2.  `co_return` thay thế `return` trong một coroutine
3.  Hầu hết các tham số được truyền theo giá trị

*Có thể tiếp tục* là một đối tượng theo tiêu chuẩn coroutine. Đừng quá lo lắng về chi tiết. Chỉ cần biết rằng nếu bạn muốn coroutine yield một giá trị kiểu `T`, thì kiểu trả về sẽ là `Task<T>`.

Truyền hầu hết các tham số theo giá trị là hệ quả trực tiếp của việc coroutine là bất đồng bộ. Không thể theo dõi khi tham chiếu vượt ra ngoài phạm vi vì đối tượng có thể bị hủy trong khi coroutine đang chờ. Hoặc tham chiếu có thể tồn tại trên một luồng khác. Do đó, nó có thể bị hủy trong khi coroutine đang thực thi.

Sẽ hợp lý hơn khi không có callback mà sử dụng `co_return` trực tiếp. Cái nào được hỗ trợ, nhưng có thể gây ra giảm thông lượng lên đến 8% trong một số điều kiện nhất định. Vui lòng xem xét việc giảm hiệu suất và liệu nó có quá lớn đối với trường hợp sử dụng hay không. Một lần nữa, cùng một ví dụ:

```c++
app.registerHandler("/num_users",
    [](HttpRequestPtr req) -> Task<HttpResponsePtr>)
    //          Bây giờ trả về một response ^^^
{
    auto sql = app().getDbClient();
    try
    {
        auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
        size_t num_users = result[0][0].as<size_t>();
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(std::to_string(num_users));
        co_return resp;
    }
    catch(const DrogonDbException &err)
    {
        // Ngoại lệ hoạt động như các giao diện đồng bộ.
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(err.base().what());
        co_return resp;
    }
}
```

Gọi coroutine từ controller WebSocket vẫn chưa được hỗ trợ. Vui lòng mở một issue nếu bạn cần tính năng này.

### Những cạm bẫy phổ biến

Có một số cạm bẫy phổ biến mà bạn có thể gặp phải khi sử dụng coroutine.

- #### Khởi chạy coroutine với lambda capture từ một hàm

  Lambda capture và coroutine có thời gian tồn tại riêng biệt. Một coroutine tồn tại cho đến khi khung coroutine bị hủy. Trong khi lambda thường hủy ngay sau khi được gọi. Do đó, do bản chất bất đồng bộ của coroutine, thời gian tồn tại của coroutine có thể dài hơn nhiều so với lambda, chẳng hạn như trong việc thực thi SQL. Lambda hủy ngay sau khi chờ SQL hoàn thành (và quay lại vòng lặp sự kiện để xử lý các sự kiện khác), trong khi khung coroutine đang chờ SQL. Do đó, lambda sẽ bị hủy khi SQL đã hoàn thành.

  Thay vì

  ```c++
  app().getLoop()->queueInLoop([num] -> AsyncTask {
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE last_login < CURRENT_TIMESTAMP - INTERVAL $1 DAY". std::to_string(num));
      // Đối tượng lambda, do đó capture bị hủy ngay khi awaiting. Chúng bị hủy tại thời điểm này
      LOG_INFO << "Remove old customers that have no activity for more than " << num << "days"; // use-after-free
  });
  // XẤU, Điều này sẽ bị crash
  ```

  Drogon cung cấp `async_func` bao bọc lambda để đảm bảo thời gian tồn tại của nó

  ```c++
  app().getLoop()->queueInLoop(async_func([num] -> Task<void> {
  //                             ^^^^^^^^^^^^^^^^^^^^^^^^^ bao bọc bằng async_func và trả về Task<>
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE last_login < CURRENT_TIMESTAMP - INTERVAL $1 DAY". std::to_string(num));
      LOG_INFO << "Remove old customers that have no activity for more than " << num << "days";
  }));
  // Tốt
  ```

- #### Truyền/capture tham chiếu vào coroutine từ hàm

  Thực hành tốt trong C++ là truyền đối tượng bằng tham chiếu để giảm sao chép không cần thiết. Tuy nhiên, việc truyền bằng tham chiếu vào coroutine từ một hàm thường gây ra sự cố. Điều này là do coroutine thực chất là bất đồng bộ và có thể có thời gian tồn tại lâu hơn nhiều so với một hàm thông thường. Ví dụ, đoạn mã sau bị crash

  ```cpp
  void removeCustomers(const std::string& customer_id)
  {
      async_run([&customer_id] {
          //      ^^^^ KHÔNG truyền/capture đối tượng bằng tham chiếu vào coroutine
          // Trừ khi bạn chắc chắn rằng đối tượng có thời gian tồn tại lâu hơn coroutine

          auto db = app().getDbClient();
          co_await db->execSqlCoro("DELETE FROM customers WHERE customer_id = $1", customer_id);
          // `customer_id` vượt ra ngoài phạm vi ngay khi awaiting SQL. Crash ở đây
          co_await db->execSqlCoro("DELETE FROM orders WHERE customer_id = $1", customer_id);
      }
  }
  ```

  Tuy nhiên, việc truyền đối tượng làm tham chiếu từ coroutine được coi là một thực hành tốt

  ```cpp
  Task<> removeCustomers(const std::string& customer_id)
  {
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE customer_id = $1", customer_id);
      co_await db->execSqlCoro("DELETE FROM orders WHERE customer_id = $1", customer_id);
  }

  Task<> findUnwantedCustomers()
  {
      auto db = app().getDbClient();
      auto list = co_await db->execSqlCoro("SELECT customer_id from customers "
          "WHERE customer_score < 5;");
      for (const auto& customer : list)
          co_await removeCustomers(customer["customer_id"].as<std::string>());
          //                               ^^^^^^^^^^^^^^^^^
          // Điều này hoàn toàn ổn và được ưu tiên mặc dù nó là một tham chiếu const
          // vì chúng ta đang gọi nó từ một coroutine
  }
  ```

[1]: https://en.cppreference.com/w/cpp/language/coroutines

# 17 [Redis](VI-17-Redis)


