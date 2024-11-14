Con trỏ kiểu `HttpRequest` thường được đặt tên là `req` trong các ví dụ trong tài liệu này đại diện cho dữ liệu chứa trong yêu cầu được nhận hoặc gửi bởi Drogon, dưới đây là một số phương thức mà bạn có thể tương tác với đối tượng này:

- ### `isOnSecureConnection()`

  #### Tóm tắt:
  Hàm trả về giá trị `true` nếu yêu cầu được thực hiện trên HTTPS.

  #### Đầu vào:
  Không có.

  #### Trả về:
  Kiểu `bool`.

- ### `getMethod()`

  #### Tóm tắt:
  Hàm trả về phương thức yêu cầu. Hữu ích để phân biệt phương thức yêu cầu nếu một handler duy nhất cho phép nhiều hơn một loại.

  #### Đầu vào:
  Không có.

  #### Trả về:
    Đối tượng phương thức yêu cầu `HttpMethod`.

  #### Ví dụ:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      if (req->getMethod() == HttpMethod::Get) {
        // Làm gì đó
      } else if (req->getMethod() == HttpMethod::Post) {
        // Làm điều gì đó khác
      }
  }
  ```

- ### `getParameter(const std::string &key)`

  #### Tóm tắt:
  Hàm trả về giá trị của tham số dựa trên mã định danh. Hành vi thay đổi dựa trên loại yêu cầu GET hoặc POST.

  #### Đầu vào:
  Mã định danh tham số kiểu chuỗi.

  #### Trả về:
  Nội dung tham số ở định dạng `std::string`.

  #### Ví dụ:
  Trên kiểu GET:
  ```c++
  #include "mycontroller.h"
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // https://mysite.com/an-path/?id=5
      std::string id = req->getParameter("id");
      // Hoặc
      long id = std::strtol(req->getParameter("id").c_str(), nullptr, 10);
  }
  ```
  Hoặc
  Trên kiểu POST:
  ```c++
  #include "mycontroller.h"
  #include <string>

  using namespace drogon;

  void mycontroller::loginHandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // Yêu cầu chứa Biểu mẫu Đăng nhập
      std::string email = req->getParameter("email");
       
      std::string password = req->getParameter("password");
  }
  ```

- ### `getPath()`

  #### Tương tự:
  `path()`

  #### Tóm tắt:
  Hàm trả về đường dẫn yêu cầu. Hữu ích nếu bạn sử dụng `ADD_METHOD_VIA_REGEX` hoặc loại URL động khác trong [bộ điều khiển](VI-04-2-Controller-HttpController).

  #### Đầu vào:
  Không có.

  #### Trả về:
  Chuỗi đại diện cho đường dẫn yêu cầu.

  #### Ví dụ:
  ```c++
  #include "mycontroller.h"
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // https://mysite.com/an-path/?id=5
      std::string url = req->getPath();
      
      // url = /an-path/
  }
  ```

- ### `getBody()`

  #### Tương tự:
  `body()`

  #### Tóm tắt:
  Hàm trả về nội dung thân yêu cầu (nếu có).

  #### Đầu vào:
  Không có.

  #### Trả về:
  Chuỗi đại diện cho phần thân yêu cầu (nếu có).

- ### `getHeader(const std::string &key)`

  #### Tóm tắt:
  Hàm trả về tiêu đề yêu cầu dựa trên mã định danh.

  #### Đầu vào:
  Mã định danh tiêu đề chuỗi.

  #### Trả về:
  Nội dung của tiêu đề ở định dạng chuỗi. 

  #### Ví dụ:
  ```c++
  #include "mycontroller.h"
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      if (req->getHeader("Host") != "mysite.com") {
        // Trả về HTTP 403 
      }
  }
  ```

- ### `headers()`

  #### Tóm tắt:
  Hàm trả về tất cả các tiêu đề của yêu cầu.

  #### Đầu vào:
  Không có.

  #### Trả về:
  Một `unordered_map` chứa các tiêu đề. 

  #### Ví dụ:
  ```c++
  #include "mycontroller.h"
  #include <unordered_map>
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      for (const auto &header : req->headers()) {
        auto header_key = header.first;
        auto header_value = header.second;
      }
  }
  ```

- ### `getCookie(const std::string &key)`

  #### Tóm tắt:
  Hàm trả về cookie yêu cầu dựa trên mã định danh.

  #### Đầu vào:
  Mã định danh cookie kiểu chuỗi.

  #### Trả về:
  Giá trị của cookie ở định dạng chuỗi.

- ### `cookies()`

  #### Tóm tắt:
  Hàm trả về tất cả các cookie của yêu cầu.

  #### Đầu vào:
  Không có.

  #### Trả về:
  Một `unordered_map` chứa các cookie. 

  #### Ví dụ:
  ```c++
  #include "mycontroller.h"
  #include <unordered_map>
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      for (const auto &cookie : req->cookies()) {
        auto cookie_key = cookie.first;
        auto cookie_value = cookie.second;
      }
  }
  ```

- ### `getJsonObject()`

  #### Tóm tắt:
  Hàm chuyển đổi giá trị thân của yêu cầu thành một đối tượng JSON (thường là yêu cầu POST).

  #### Đầu vào:
  Không có.

  #### Trả về:
  Một đối tượng JSON. 

  #### Ví dụ:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // body = {"email": "test@gmail.com"}
      auto jsonData = req->getJsonObject();

      if (jsonData) {
          std::string email = (*jsonData)["email"].asString();
      }
  }
  ```

## Những điều hữu ích

###### Từ đây không phải là phương thức của đối tượng Yêu cầu HTTP, mà là một số điều hữu ích bạn có thể làm để xử lý các yêu cầu bạn sẽ nhận được

### Phân tích cú pháp yêu cầu tệp

```c++
#include "mycontroller.h"

using namespace drogon;

void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
    // Chỉ Yêu cầu POST (Biểu mẫu tệp)

    MultiPartParser file;
    file.parse(req);

    if (file.getFiles().empty()) {
      // Không tìm thấy tệp
    }

    // Lấy tệp đầu tiên và lưu sau đó
    const HttpFile &archive = file.getFiles()[0];
    archive.saveAs("/tmp/" + archive.getFileName());
  }
```
Để biết thêm thông tin về phân tích cú pháp tệp: [Trình xử lý tệp](VI-09-1-File-Handler)

# Tiếp theo: [Plugin](VI-09-1-File-Handler) 
