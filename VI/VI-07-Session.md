## Phiên (Session)

`Session` là một khái niệm quan trọng của ứng dụng web. Nó được sử dụng để lưu trạng thái của máy khách trên máy chủ. Nói chung, nó phối hợp với `cookie` của trình duyệt, và Drogon cung cấp hỗ trợ cho phiên. Drogon **tắt** lựa chọn phiên theo mặc định, bạn cũng có thể tắt hoặc mở nó thông qua giao diện sau:

```c++
void disableSession();
void enableSession(const size_t timeout = 0, Cookie::SameSite sameSite = Cookie::SameSite::kNull);
```

Các phương thức trên đều được gọi thông qua singleton `HttpAppFramework`. Tham số `timeout` đại diện cho thời gian phiên không hợp lệ. Đơn vị là giây. Giá trị mặc định là 1200. Tức là, nếu người dùng không truy cập ứng dụng web trong hơn 20 phút, phiên tương ứng sẽ không hợp lệ. Đặt `timeout` thành 0 có nghĩa là Drogon sẽ giữ phiên của người dùng trong toàn bộ thời gian tồn tại;
Tham số `sameSite` thay đổi thuộc tính `SameSite` của tiêu đề phản hồi HTTP `Set-Cookie`.

Hãy chắc chắn rằng máy khách của bạn hỗ trợ cookie trước khi mở tính năng phiên. Nếu không, Drogon sẽ tạo một phiên mới cho mỗi yêu cầu không có cookie `SessionID`, điều này sẽ lãng phí bộ nhớ và tài nguyên tính toán.

### Đối tượng Session

Kiểu đối tượng phiên của Drogon là `drogon::Session`, rất giống với `HttpViewData`. Nó có thể truy cập bất kỳ kiểu đối tượng nào thông qua từ khóa; hỗ trợ đọc và ghi đồng thời; vui lòng tham khảo mô tả của lớp `Session` để biết cách sử dụng cụ thể;

Framework Drogon sẽ chuyển đối tượng phiên đến đối tượng `HttpRequest` và chuyển nó cho người dùng. Người dùng có thể lấy đối tượng `Session` thông qua giao diện sau của lớp `HttpRequest`:

```c++
SessionPtr session() const;
```

Giao diện trả về một con trỏ thông minh của đối tượng `Session`, thông qua đó có thể truy cập các đối tượng khác nhau;

### Ví dụ về phiên

Chúng tôi thêm một tính năng yêu cầu hỗ trợ phiên. Ví dụ: chúng tôi muốn giới hạn tần suất truy cập của người dùng. Sau một lần truy cập, nếu nó được truy cập lại trong vòng 10 giây, nó sẽ trả về lỗi, nếu không nó sẽ trả về ok. Chúng ta cần ghi lại thời gian truy cập cuối cùng trong phiên, sau đó so sánh nó với thời gian truy cập lần này, bạn có thể đạt được chức năng này.

Chúng tôi tạo một Filter để thực hiện chức năng này, giả sử tên lớp là `TimeFilter`, việc triển khai như sau:

```c++
#include "TimeFilter.h"
#include <trantor/utils/Date.h>
#include <trantor/utils/Logger.h>
#define VDate "visitDate"

void TimeFilter::doFilter(const HttpRequestPtr &req,
                          FilterCallback &&cb,
                          FilterChainCallback &&ccb)
{
    trantor::Date now = trantor::Date::date();
    LOG_TRACE << "";
    if (req->session()->find(VDate))
    {
        auto lastDate = req->session()->get<trantor::Date>(VDate);
        LOG_TRACE << "last:" << lastDate.toFormattedString(false);
        req->session()->modify<trantor::Date>(VDate,
                                            [now](trantor::Date &vdate) {
                                                vdate = now;
                                            });
        LOG_TRACE << "update visitDate";
        if (now > lastDate.after(10))
        {
            // 10 giây sau có thể truy cập lại;
            ccb();
            return;
        }
        else
        {
            Json::Value json;
            json["result"] = "error";
            json["message"] = "Khoảng thời gian truy cập phải ít nhất 10 giây";
            auto res = HttpResponse::newHttpJsonResponse(json);
            cb(res);
            return;
        }
    }
    LOG_TRACE << "Lần truy cập đầu tiên, chèn visitDate";
    req->session()->insert(VDate, now);
    ccb();
}
```

Sau đó, chúng tôi đăng ký một biểu thức lambda cho đường dẫn `/slow` và đính kèm `TimeFilter` bằng mã sau:

```c++
drogon::HttpAppFramework::instance()
            .registerHandler("/slow",
                            [=](const HttpRequestPtr &req,
                                std::function<void (const HttpResponsePtr &)> &&callback)
                            {
                                Json::Value json;
                                json["result"] = "ok";
                                auto resp = HttpResponse::newHttpJsonResponse(json);
                                callback(resp);
                            },
                            {Get, "TimeFilter"});
```

Gọi giao diện framework để mở phiên:

```c++
drogon::HttpAppFramework::instance().enableSession(1200);
```

Biên dịch lại toàn bộ dự án bằng CMake, chạy chương trình mục tiêu webapp, và bạn có thể thấy hiệu ứng thông qua trình duyệt.


# Tiếp theo: [Cơ sở dữ liệu](VI-08-0-Database-General)


