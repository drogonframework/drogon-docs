## HttpSimpleController

Bạn có thể sử dụng công cụ dòng lệnh `drogon_ctl` để nhanh chóng tạo các tệp nguồn lớp bộ điều khiển tùy chỉnh dựa trên `HttpSimpleController`. Định dạng lệnh như sau:

```shell
drogon_ctl create controller <[namespace::]class_name>
```

Chúng ta tạo một lớp bộ điều khiển có tên `TestCtrl`:

```shell
drogon_ctl create controller TestCtrl
```

Như bạn có thể thấy, có hai tệp mới, `TestCtrl.h` và `TestCtrl.cc`. Bây giờ, hãy xem chúng:

`TestCtrl.h`:

```c++
#pragma once
#include <drogon/HttpSimpleController.h>
using namespace drogon;
class TestCtrl : public drogon::HttpSimpleController<TestCtrl>
{
  public:
    virtual void asyncHandleHttpRequest(const HttpRequestPtr &req,
                                        std::function<void(const HttpResponsePtr &)> &&callback) override;
    PATH_LIST_BEGIN
    // Liệt kê các định nghĩa đường dẫn ở đây;
    //PATH_ADD("/path", "filter1", "filter2", HttpMethod1, HttpMethod2...);
    PATH_LIST_END
};
```

`TestCtrl.cc`:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void(const HttpResponsePtr &)> &&callback)
{
    // Viết logic ứng dụng của bạn ở đây
}
```

Mỗi lớp `HttpSimpleController` chỉ có thể định nghĩa một trình xử lý yêu cầu HTTP và nó được định nghĩa bằng cách ghi đè một hàm ảo.

Việc định tuyến (hay còn gọi là ánh xạ) từ đường dẫn URL đến trình xử lý được thực hiện bằng một macro. Bạn có thể thêm ánh xạ đa đường dẫn với macro `PATH_ADD`. Tất cả các câu lệnh `PATH_ADD` phải được đặt giữa các câu lệnh macro `PATH_LIST_BEGIN` và `PATH_LIST_END`.

Tham số đầu tiên là đường dẫn cần được ánh xạ và các tham số ngoài đường dẫn là các ràng buộc trên đường dẫn này. Hiện tại, hai loại ràng buộc được hỗ trợ. Một là kiểu enum `HttpMethod`, có nghĩa là phương thức HTTP được phép. Loại còn lại là tên của lớp `HttpFilter`. Người ta có thể cấu hình bất kỳ số lượng nào trong hai loại ràng buộc này và không có yêu cầu về thứ tự đối với chúng. Đối với Filter, vui lòng tham khảo [Middleware và Filter](VI-05-Middleware-and-Filter).

Người dùng có thể đăng ký cùng một `Simple Controller` cho nhiều đường dẫn hoặc đăng ký nhiều `Simple Controller` trên cùng một đường dẫn (sử dụng các phương thức HTTP khác nhau).

Bạn có thể định nghĩa một biến lớp `HttpResponse`, sau đó sử dụng `callback()` để trả về nó:

```c++
    // Viết logic ứng dụng của bạn ở đây
    auto resp = HttpResponse::newHttpResponse();
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody("Nội dung trang của bạn");
    callback(resp);
```

> **Việc ánh xạ từ đường dẫn ở trên đến trình xử lý được thực hiện tại thời điểm biên dịch. Trên thực tế, framework Drogon cũng cung cấp một giao diện để hoàn thành ánh xạ trong thời gian chạy. Ánh xạ thời gian chạy cho phép người dùng ánh xạ hoặc sửa đổi ánh xạ thông qua tệp cấu hình hoặc giao diện người dùng khác mà không cần biên dịch lại chương trình này (Vì lý do hiệu suất, không được phép thêm bất kỳ ánh xạ bộ điều khiển nào sau khi chạy phương thức `app().run()`).**


# Tiếp theo: [HttpController](VI-04-2-Controller-HttpController)
