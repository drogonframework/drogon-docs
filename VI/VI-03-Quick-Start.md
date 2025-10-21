## Trang web tĩnh

Hãy bắt đầu với một ví dụ đơn giản giới thiệu cách sử dụng Drogon. Trong ví dụ này, chúng ta tạo một dự án bằng cách sử dụng công cụ dòng lệnh `drogon_ctl`:

```shell
drogon_ctl create project ten_du_an_cua_ban
```

Đã có một số thư mục hữu ích trong thư mục dự án:

```console
├── build                         Thư mục build
├── CMakeLists.txt                Tệp cấu hình CMake của dự án
├── config.json                   Tệp cấu hình ứng dụng Drogon
├── controllers                   Thư mục lưu trữ các tệp nguồn bộ điều khiển
├── filters                       Thư mục lưu trữ các tệp bộ lọc
├── main.cc                       Chương trình chính
├── models                        Thư mục của tệp mô hình cơ sở dữ liệu
│   └── model.json
└── views                         Thư mục lưu trữ các tệp CSP của view
```

Người dùng có thể đặt các tệp khác nhau (chẳng hạn như bộ điều khiển, bộ lọc, view, v.v.) vào các thư mục tương ứng. Để thuận tiện hơn và ít lỗi hơn, chúng tôi khuyên người dùng nên tạo các dự án ứng dụng web của riêng mình bằng lệnh `drogon_ctl`. Xem [drogon_ctl](VI-12-drogon_ctl-Command) để biết thêm chi tiết.

Hãy xem tệp `main.cc`:

```c++
#include <drogon/HttpAppFramework.h>
int main() {
    // Đặt địa chỉ và cổng của bộ lắng nghe HTTP
    drogon::app().addListener("0.0.0.0", 80);
    // Tải tệp cấu hình
    //drogon::app().loadConfigFile("../config.json");
    // Chạy framework HTTP, phương thức này sẽ chặn trong vòng lặp sự kiện nội bộ
    drogon::app().run();
    return 0;
}
```

Sau đó build dự án của bạn như dưới đây:

```shell
cd build
cmake ..
make
```

Sau khi quá trình biên dịch hoàn tất, hãy chạy mục tiêu `./ten_du_an_cua_ban`.

Bây giờ, chúng ta chỉ cần thêm một tệp tĩnh `index.html` vào đường dẫn gốc HTTP:

```shell
echo '<h1>Hello Drogon!</h1>' >> index.html
```

Đường dẫn gốc mặc định là `"./"`, nhưng cũng có thể được sửa đổi bởi `config.json`. Xem [Tệp cấu hình](VI-11-Configuration-File) để biết thêm chi tiết. Sau đó, bạn có thể truy cập trang này bằng URL `"http://localhost"` hoặc `"http://localhost/index.html"` (hoặc IP của máy chủ nơi ứng dụng web của bạn đang chạy).

![Hello Drogon!](https://drogonframework.github.io/drogon-docs/images/hellodrogon.png)

Nếu máy chủ không thể tìm thấy trang bạn đã yêu cầu, nó sẽ trả về trang 404:
![404 page](https://drogonframework.github.io/drogon-docs/images//notfound.png)

> **Lưu ý: Đảm bảo tường lửa máy chủ của bạn đã cho phép cổng 80. Nếu không, bạn sẽ không thấy các trang này. (Một cách khác là thay đổi cổng của bạn từ 80 thành 1024 (hoặc cao hơn) trong trường hợp bạn nhận được thông báo lỗi bên dưới):**

```console
FATAL Permission denied (errno=13) , Bind address failed at 0.0.0.0:80 - Socket.cc:67
```

Chúng ta có thể sao chép thư mục và tệp của một trang web tĩnh vào thư mục khởi động của ứng dụng web đang chạy này, sau đó chúng ta có thể truy cập chúng từ trình duyệt. Các loại tệp được Drogon hỗ trợ theo mặc định là:

- html
- js
- css
- xml
- xsl
- txt
- svg
- ttf
- otf
- woff2
- woff
- eot
- png
- jpg
- jpeg
- gif
- bmp
- ico
- icns

Drogon cũng cung cấp các giao diện để thay đổi các loại tệp này. Để biết chi tiết, vui lòng tham khảo [HttpAppFramework API](API-HttpAppFramework).


## Trang web động

Hãy xem cách thêm bộ điều khiển vào ứng dụng này và để bộ điều khiển phản hồi bằng nội dung.

Bạn có thể sử dụng công cụ dòng lệnh `drogon_ctl` để tạo tệp nguồn bộ điều khiển. Hãy chạy nó trong thư mục `controllers`:

```shell
drogon_ctl create controller TestCtrl
```

Như bạn có thể thấy, có hai tệp mới, `TestCtrl.h` và `TestCtrl.cc`:

`TestCtrl.h` như sau:

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

`TestCtrl.cc` như sau:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void(const HttpResponsePtr &)> &&callback)
{
    // Viết logic ứng dụng của bạn ở đây
}
```

Hãy chỉnh sửa hai tệp và để bộ điều khiển xử lý phản hồi hàm thành một "Hello World!" đơn giản

`TestCtrl.h` như sau:

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
    // Liệt kê các định nghĩa đường dẫn ở đây

    // Ví dụ
    //PATH_ADD("/path", "filter1", "filter2", HttpMethod1, HttpMethod2...);

    PATH_ADD("/", Get, Post);
    PATH_ADD("/test", Get);
    PATH_LIST_END
};
```

Sử dụng `PATH_ADD` để ánh xạ hai đường dẫn `/` và `/test` tới các hàm xử lý và thêm các ràng buộc đường dẫn tùy chọn (ở đây, các phương thức HTTP được phép).


`TestCtrl.cc` như sau:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void(const HttpResponsePtr &)> &&callback)
{
    // Viết logic ứng dụng của bạn ở đây
    auto resp = HttpResponse::newHttpResponse();
    // LƯU Ý: Hằng số enum bên dưới được đặt tên là "k200OK" (như trong 200 OK), không phải "k2000K".
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody("Hello World!");
    callback(resp);
}
```

Biên dịch lại dự án này bằng CMake, sau đó chạy mục tiêu `./ten_du_an_cua_ban`:

```shell
cd ../build
cmake ..
make
./ten_du_an_cua_ban
```

Nhập `"http://localhost/"` hoặc `"http://localhost/test"` trong thanh địa chỉ trình duyệt và bạn sẽ thấy "Hello World!" trong trình duyệt.

> **Lưu ý: Nếu máy chủ của bạn có cả tài nguyên tĩnh và động, Drogon sẽ sử dụng tài nguyên động trước. Trong ví dụ này, phản hồi cho `GET http://localhost/` là `Hello World!` (từ tệp bộ điều khiển `TestCtrl`) thay vì `Hello Drogon!` (từ tệp tĩnh `index.html`).**


Chúng ta thấy rằng việc thêm một bộ điều khiển vào một ứng dụng rất đơn giản. Bạn chỉ cần thêm tệp nguồn tương ứng. Ngay cả tệp chính cũng không cần phải sửa đổi. Thiết kế kết hợp lỏng lẻo này rất hiệu quả cho việc phát triển ứng dụng web.

> **Lưu ý: Drogon không có giới hạn về vị trí của các tệp nguồn bộ điều khiển. Bạn cũng có thể lưu chúng trong "./" (thư mục gốc của dự án), hoặc thậm chí bạn có thể định nghĩa một thư mục mới trong `CMakeLists.txt`. Nên sử dụng thư mục `controllers` để thuận tiện cho việc quản lý.**


# Tiếp theo: [Lệnh drogon_ctl](VI-04-0-Controller-Introduction)
