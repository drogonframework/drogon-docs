# Hiểu Mô hình Luồng của Drogon

Drogon là một framework ứng dụng web C++ nhanh. Một phần lý do khiến nó nhanh là do nó không trừu tượng hóa mô hình luồng bên dưới. Tuy nhiên, điều này cũng gây ra một số nhầm lẫn. Không có gì lạ khi thấy các vấn đề và thảo luận về lý do tại sao phản hồi chỉ được gửi sau một số lệnh gọi chặn, tại sao gọi một hàm mạng chặn trên cùng một vòng lặp sự kiện chặn lại gây ra deadlock, v.v. Trang này nhằm mục đích giải thích chính xác điều kiện gây ra chúng và cách tránh chúng.

## Vòng lặp sự kiện và luồng

Drogon chạy trên một nhóm luồng, trong đó mỗi luồng có vòng lặp sự kiện (event loop) riêng. Và các vòng lặp sự kiện là cốt lõi của Drogon. Mọi ứng dụng Drogon đều có ít nhất 2 vòng lặp sự kiện: một vòng lặp chính (main loop) và một vòng lặp worker. Miễn là bạn không làm bất cứ điều gì kỳ lạ, vòng lặp chính luôn chạy trên luồng chính (luồng đã khởi động `main`). Và nó chịu trách nhiệm khởi động tất cả các vòng lặp worker. Lấy ứng dụng "Hello World" làm ví dụ, dòng `app().run()` khởi động vòng lặp chính trên luồng chính. Sau đó, nó sẽ tạo ra 3 luồng/vòng lặp worker.

```cpp
#include <drogon/drogon.h>
using namespace drogon;

int main()
{
    app().registerHandler("/", [](const HttpRequest& req
        , std::function<void (const HttpResponsePtr &)> &&callback) {
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody("Hello world");
        callback(resp);
    });
    app().addListener("0.0.0.0", 80800);
    app().setNumThreads(3);
    app().run();
}
```

Hệ thống phân cấp luồng trông như thế này:

```
            .-------------.
            | app().run() | <luồng chính>
            :-------------:
                  |
            .-----v------.
            | MAIN LOOP  |
            :------------:
                  | Tạo ra
      .-----------+--------------.
      |           |              |
.-----v----.   .--v-------.  .---v----.
| Worker 1 |   | Worker 2 |  | etc... |
:----------:   :----------:  :--------:
 <luồng 1>     <luồng 2>   <luồng ...>
```

Số lượng vòng lặp worker phụ thuộc vào nhiều biến. Cụ thể là bao nhiêu luồng được chỉ định cho máy chủ HTTP, bao nhiêu kết nối cơ sở dữ liệu và NoSQL không nhanh (non-fast) được tạo - chúng ta sẽ tìm hiểu về kết nối nhanh so với kết nối không nhanh sau. Chỉ cần biết rằng Drogon có nhiều luồng hơn là chỉ các luồng máy chủ HTTP. Mỗi vòng lặp sự kiện về cơ bản là một hàng đợi tác vụ với các chức năng sau:

- Đọc các tác vụ từ hàng đợi tác vụ và thực thi chúng. Bạn có thể submit tác vụ để chạy trên một vòng lặp từ bất kỳ luồng nào khác. Việc submit tác vụ hoàn toàn không có khóa (nhờ cấu trúc dữ liệu không có khóa - lock-free data structure) và sẽ không gây ra xung đột dữ liệu (data race) trong mọi trường hợp. Vòng lặp sự kiện xử lý các tác vụ từng cái một. Do đó, các tác vụ có thứ tự thực thi được xác định rõ ràng. Nhưng đồng thời, các tác vụ được xếp hàng sau một tác vụ lớn, chạy lâu sẽ bị trì hoãn.
- Lắng nghe và điều phối các sự kiện mạng mà nó quản lý.
- Thực thi bộ hẹn giờ (timer) khi chúng hết thời gian (thường do người dùng tạo).

Khi không có điều nào ở trên xảy ra, vòng lặp sự kiện/luồng sẽ chặn và chờ chúng.

```cpp
// Xếp hàng hai tác vụ trên vòng lặp chính
trantor::EventLoop* loop = app().getLoop();
loop->queueInLoop([]{
    std::cout << "task1: Tôi sẽ đợi 5 giây\n";
    std::this_thread::sleep_for(5s);
    std::cout << "task1: xin chào!\n";
});
loop->queueInLoop([]{
    std::cout << "task2: thế giới!\n";
});
```

Hy vọng rằng điều này đã giải thích rõ ràng tại sao việc chạy đoạn mã trên sẽ dẫn đến `task1: Tôi sẽ đợi 5 giây` xuất hiện ngay lập tức. Tạm dừng trong 5 giây và sau đó cả `task1: xin chào!` và `task2: thế giới!` xuất hiện.

> Vì vậy, **mẹo 1**: **Không gọi I/O chặn trong vòng lặp sự kiện. Các tác vụ khác phải đợi I/O đó.**

## I/O mạng trong thực tế

Hầu hết mọi thứ trong Drogon đều được liên kết với một vòng lặp sự kiện. Điều này bao gồm luồng/kết nối TCP, client HTTP, client cơ sở dữ liệu và bộ nhớ đệm dữ liệu. Để tránh race condition, tất cả các I/O đều được thực hiện trong vòng lặp sự kiện được liên kết. Nếu một lệnh gọi I/O được thực hiện từ một luồng khác, thì các tham số được lưu trữ và submit dưới dạng một tác vụ đến vòng lặp sự kiện thích hợp. Điều này có một số hàm ý. Ví dụ: khi gọi điểm cuối từ xa từ bên trong trình xử lý HTTP hoặc thực hiện lệnh gọi cơ sở dữ liệu. Callback từ client có thể không nhất thiết (thực tế, thường không) chạy trên cùng một luồng với trình xử lý.

```cpp
app().registerHandler("/send_req", [](const HttpRequest& req
    , std::function<void (const HttpResponsePtr &)> &&callback) {
    // Trình xử lý này sẽ chạy trên một trong các luồng máy chủ HTTP

    // Tạo một client HTTP chạy trên vòng lặp chính
    auto client = HttpClient::newHttpClient("https://drogon.org", app().getLoop());
    auto request = HttpRequest::newHttpRequest();
    client->sendRequest(request, [](ReqResult result, const HttpResponse& resp) {
        // Callback này chạy trên luồng chính
    });
});
```

Do đó, có thể làm tắc nghẽn vòng lặp sự kiện nếu bạn không nhận thức được mã của mình thực sự đang làm gì. Giống như tạo hàng tấn client HTTP trên vòng lặp chính và gửi tất cả các yêu cầu gửi đi. Hoặc chạy các hàm tính toán nặng trong một callback cơ sở dữ liệu, ngăn chặn các truy vấn cơ sở dữ liệu khác đang được xử lý.

```
 Worker 1       Main Loop        Worker2
.---------.    .----------.     .---------.
|         |    |          |     |         |
| req 1-. |    |----------|  .--+--req 2  |
|       :-+----+->        |  |  |         |
|         |    | gửi HTTP |  |  |---------|
|---------| a.-|   req 1  |  |  |         |
|other req| s| |----------|  |  |         |
|---------| y| |      <---+--:  |         |
|         | n| |gửi HTTP |     |         |
|         | c| |  req 2   |-.   |         |
|         |  | |----------| |a  |---------|
|         |  | |HTTP resp1| |s  |other req|
|         |  :-|>tính toán  | |y  |---------|
|         |    |          | |n  |         |
|         | .--+-tạo | |c  |         |
|         | |  | phản hồi | |   |         |
|         | |  |----------| |   |         |
|         | |  |HTTP resp2|<:   |         |
|---------| |  | tính toán  |     |---------|
|phản hồi<|-:  |          |-----|>        |
|gửi lại|    | tạo |     |gửi phản hồi|
|         |    | phản hồi |     | lại    |
:---------:    :----------:     :---------:
```

Nguyên tắc tương tự cũng đúng với máy chủ HTTP. Nếu phản hồi được tạo từ một luồng riêng biệt (ví dụ: trong một callback cơ sở dữ liệu). Sau đó, phản hồi được xếp hàng trên luồng được liên kết để gửi thay vì gửi ngay lập tức.

> **Mẹo 2**: **Hãy lưu ý nơi bạn đặt các phép tính của mình. Chúng cũng có thể gây hại cho thông lượng nếu không cẩn thận.**

## Deadlock vòng lặp sự kiện

Với sự hiểu biết về cách Drogon được thiết kế, không khó để nhận ra cách tạo ra deadlock cho một vòng lặp sự kiện. Bạn chỉ cần submit một yêu cầu I/O từ xa và đợi nó trên cùng một vòng lặp - API đồng bộ chính xác là như vậy. Nó submit một yêu cầu I/O và đợi callback.

```cpp
app().registerHandler("/dead_lock", [](const HttpRequest& req
    , std::function<void (const HttpResponsePtr &)> &&callback) {
    auto currentLoop = app().getIOLoops()[app().getCurrentThreadIndex()];
    auto client = HttpClient::newHttpClient("https://drogon.org", currentLoop);
    auto request = HttpRequest::newHttpRequest();
    auto resp = client->sendRequest(resp); // DEADLOCK! Gọi giao diện đồng bộ
});
```

Có thể được hình dung như:

```
     Một số vòng lặp
   .------------.
   | client mới |
   | yêu cầu mới |
   | gửi yêu cầu |
 .-> CHỜ phản hồi ---+-.
 | |  ....      | |
?| |            | |
?| |------------| |
?| |            | |
?| |            | |
?| |            | |
?| |            | |
 | |            | |
 | |            | |
 | |            | |
 | |            | |
 | |            | |
 | |            | |
 | |------------| |
 | | đọc phản hồi | |
 :-+-         <-+-:
   | oops       |
   | deadlock   |
   :------------:
```

Điều tương tự cũng đúng với mọi thứ khác. Cơ sở dữ liệu, callback NoSQL, v.v. May mắn thay, client cơ sở dữ liệu non-fast chạy trên các luồng riêng của chúng; mỗi client nhận được luồng riêng của mình. Do đó, an toàn để thực hiện các truy vấn cơ sở dữ liệu đồng bộ từ một handler HTTP. Tuy nhiên, bạn không nên chạy các truy vấn cơ sở dữ liệu đồng bộ bên trong callback của cùng một client. Nếu không thì điều tương tự sẽ xảy ra.

> **Mẹo 3**: **API đồng bộ gây hại cho cả hiệu suất và sự an toàn. Tránh chúng như bệnh dịch. Nếu bạn phải làm vậy, hãy đảm bảo rằng bạn chạy client trên một luồng riêng biệt.**

### Client Cơ sở dữ liệu Nhanh (Fast DB Client)

Drogon được thiết kế ưu tiên hiệu suất, sau đó là dễ sử dụng. Client cơ sở dữ liệu nhanh là client cơ sở dữ liệu chia sẻ các luồng máy chủ HTTP. Điều này cải thiện hiệu suất bằng cách loại bỏ nhu cầu submit yêu cầu đến một luồng khác và tránh chuyển đổi ngữ cảnh (context switch) bởi hệ điều hành. Tuy nhiên, do thực tế đó, bạn không thể thực hiện các truy vấn đồng bộ trên chúng, vì nó sẽ gây ra deadlock cho vòng lặp sự kiện.

## Coroutine để Giải cứu

Một tình huống khó xử trong việc phát triển và sử dụng Drogon là API bất đồng bộ hiệu quả hơn nhưng khó sử dụng. Trong khi API đồng bộ có thể gặp vấn đề và chậm, nhưng chúng dễ dàng lập trình hơn. Khai báo Lambda có thể dài dòng và tẻ nhạt, và cú pháp không đẹp mắt. Bên cạnh đó, mã không chạy từ trên xuống dưới; nó đầy đủ các callback. API đồng bộ gọn gàng hơn nhiều so với API bất đồng bộ, nhưng lại đánh đổi bằng hiệu suất.

```cpp
// API cơ sở dữ liệu bất đồng bộ của Drogon
auto db = app().getDbClient();
db->execSqlAsync("INSERT......", [db, callback](auto result){
    db->execSqlAsync("UPDATE .......", [callback](auto result){
        // Xử lý thành công
    },
    [callback](const DbException& e) {
        // xử lý lỗi
    })
},
[callback](const DbException& e){
    // xử lý lỗi
})
```

so với

```cpp
// API đồng bộ của Drogon. Ngoại lệ có thể được xử lý tự động bởi framework
db->execSqlSync("INSERT.....");
db->execSqlSync("UPDATE.....");
```

Phải có cách để có cả hai, phải không? Hãy đến với coroutine của C++20. Về cơ bản, chúng là một trình bao bọc hạng nhất, được trình biên dịch hỗ trợ xung quanh các callback để làm cho mã của bạn trông giống như đồng bộ. Nhưng thực tế là bất đồng bộ theo mọi cách. Đây là cùng một mã trong coroutine.

```cpp
co_await db->execSqlCoro("INSERT.....");
co_await db->execSqlCoro("UPDATE.....");
```

Nó giống hệt API đồng bộ! Nhưng tốt hơn theo hầu hết mọi cách. Bạn nhận được tất cả lợi ích của bất đồng bộ nhưng vẫn tiếp tục sử dụng giao diện đồng bộ. Cách thức hoạt động thực sự của nó nằm ngoài phạm vi của bài viết này. Những người bảo trì khuyến khích sử dụng coroutine khi bạn có thể (GCC >= 11, MSVC >= 16.25). Tuy nhiên, nó không hoàn toàn là ma thuật. Nó không giải quyết được các vòng lặp sự kiện tắc nghẽn và race condition cho bạn. Tuy nhiên, việc gỡ lỗi và hiểu mã bất đồng bộ với coroutine dễ dàng hơn.

**Mẹo 4**: **Sử dụng coroutine khi bạn có thể.**

## Tóm tắt

- Sử dụng coroutine C++20 và kết nối `Fast DB` khi bạn có thể.
- API đồng bộ có thể làm chậm hoặc thậm chí deadlock vòng lặp sự kiện.
- Nếu bạn phải sử dụng API đồng bộ, hãy đảm bảo rằng chúng được liên kết với một luồng khác.

