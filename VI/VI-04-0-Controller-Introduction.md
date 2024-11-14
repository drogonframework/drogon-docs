## Bộ điều khiển - Giới thiệu

Bộ điều khiển rất quan trọng trong phát triển ứng dụng web. Đây là nơi chúng ta sẽ định nghĩa URL, phương thức HTTP nào được phép, [bộ lọc](VI-06-Middleware-and-Filter) nào sẽ được áp dụng và cách xử lý và phản hồi yêu cầu. Framework Drogon đã giúp chúng ta xử lý việc truyền tải mạng, phân tích giao thức HTTP, v.v. Chúng ta chỉ cần chú ý đến logic của bộ điều khiển; mỗi đối tượng bộ điều khiển có thể có một hoặc nhiều hàm xử lý (thường được gọi là handler) và giao diện của hàm thường được định nghĩa như sau:

```c++
void handlerName(const HttpRequestPtr &req,
                  std::function<void (const HttpResponsePtr &)> &&callback,
                 ...);
```

Trong đó `req` là đối tượng của yêu cầu HTTP (được bao bọc bởi con trỏ thông minh), `callback` là đối tượng hàm gọi lại mà framework truyền cho bộ điều khiển, và bộ điều khiển tạo đối tượng phản hồi (cũng được bao bọc bởi con trỏ thông minh) và sau đó truyền đối tượng cho Drogon thông qua hàm gọi lại. Sau đó, framework sẽ gửi nội dung phản hồi đến trình duyệt cho bạn. Phần cuối cùng `...` là danh sách các tham số. Drogon ánh xạ các tham số trong yêu cầu HTTP tới các tham số tương ứng theo các quy tắc ánh xạ. Điều này rất thuận tiện cho việc phát triển ứng dụng.

Rõ ràng, đây là một giao diện bất đồng bộ, người ta có thể gọi hàm gọi lại sau khi hoàn thành thao tác tốn thời gian tại các luồng khác;

Drogon có ba loại bộ điều khiển: `HttpSimpleController`, `HttpController`, và `WebSocketController`. Khi bạn sử dụng chúng, bạn cần kế thừa từ template lớp tương ứng. Ví dụ: khai báo lớp tùy chỉnh "MyClass" kế thừa từ `HttpSimpleController` như sau:

```c++

class MyClass : public drogon::HttpSimpleController<MyClass>
{
  public:
    //TestController(){}
    virtual void asyncHandleHttpRequest(const HttpRequestPtr &req,
                                         std::function<void (const HttpResponsePtr &)> &&callback) override;

    PATH_LIST_BEGIN
    PATH_ADD("/json");
    PATH_LIST_END
};
```

### Vòng đời của bộ điều khiển

Một bộ điều khiển được đăng ký với framework Drogon sẽ có nhiều nhất chỉ một phiên bản và sẽ không bị hủy trong toàn bộ quá trình chạy ứng dụng, vì vậy người dùng có thể khai báo và sử dụng các biến thành viên trong lớp bộ điều khiển. Lưu ý rằng khi handler của bộ điều khiển được gọi, nó đang ở trong một môi trường đa luồng (khi số lượng luồng IO của framework được cấu hình lớn hơn 1), nếu bạn cần truy cập các biến không phải tạm thời, vui lòng thực hiện công việc đồng bộ hóa để đảm bảo an toàn luồng.


# Tiếp theo: [HttpSimpleController](VI-04-1-Controller-HttpSimpleController)
