## HttpController

### Tạo

Bạn có thể sử dụng công cụ dòng lệnh `drogon_ctl` để nhanh chóng tạo các tệp nguồn lớp bộ điều khiển tùy chỉnh dựa trên `HttpController`. Định dạng lệnh như sau:

```shell
drogon_ctl create controller -h <[namespace::]class_name>
```

Chúng ta tạo một lớp bộ điều khiển có tên `User`, dưới namespace `demo::v1`:

```shell
drogon_ctl create controller -h demo::v1::User
```

Như bạn có thể thấy, hai tệp đã được thêm vào thư mục hiện tại, `demo_v1_User.h` và `demo_v1_User.cc`.

`demo_v1_User.h` như sau:

```c++
#pragma once

#include <drogon/HttpController.h>

using namespace drogon;

namespace demo
{
namespace v1
{
class User : public drogon::HttpController<User>
{
  public:
    METHOD_LIST_BEGIN
    // Sử dụng METHOD_ADD để thêm hàm xử lý tùy chỉnh của bạn vào đây;
    // METHOD_ADD(User::get, "/{2}/{1}", Get); // Đường dẫn là /demo/v1/User/{arg2}/{arg1}
    // METHOD_ADD(User::your_method_name, "/{1}/{2}/list", Get); // Đường dẫn là /demo/v1/User/{arg1}/{arg2}/list
    // ADD_METHOD_TO(User::your_method_name, "/absolute/path/{1}/{2}/list", Get); // Đường dẫn là /absolute/path/{arg1}/{arg2}/list

    METHOD_LIST_END
    // Khai báo hàm xử lý của bạn có thể như thế này:
    // void get(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, int p1, std::string p2);
    // void your_method_name(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, double p1, int p2) const;
};
} // namespace v1
} // namespace demo
```

`demo_v1_User.cc` như sau:

```c++
#include "demo_v1_User.h"

using namespace demo::v1;

// Thêm định nghĩa hàm xử lý của bạn vào đây
```

### Sử dụng

Hãy chỉnh sửa hai tệp:

`demo_v1_User.h` như sau:

```c++
#pragma once

#include <drogon/HttpController.h>

using namespace drogon;

namespace demo
{
namespace v1
{
class User : public drogon::HttpController<User>
{
  public:
    METHOD_LIST_BEGIN
    // Sử dụng METHOD_ADD để thêm hàm xử lý tùy chỉnh của bạn vào đây;
    METHOD_ADD(User::login, "/token?userId={1}&passwd={2}", Post);
    METHOD_ADD(User::getInfo, "/{1}/info?token={2}", Get);
    METHOD_LIST_END
    // Khai báo hàm xử lý của bạn có thể như thế này:
    void login(const HttpRequestPtr &req,
               std::function<void(const HttpResponsePtr &)> &&callback,
               std::string &&userId,
               const std::string &password);
    void getInfo(const HttpRequestPtr &req,
                 std::function<void(const HttpResponsePtr &)> &&callback,
                 std::string userId,
                 const std::string &token) const;
};
} // namespace v1
} // namespace demo
```

`demo_v1_User.cc` như sau:

```c++
#include "demo_v1_User.h"

using namespace demo::v1;

// Thêm định nghĩa hàm xử lý của bạn vào đây

void User::login(const HttpRequestPtr &req,
                 std::function<void(const HttpResponsePtr &)> &&callback,
                 std::string &&userId,
                 const std::string &password)
{
    LOG_DEBUG << "User " << userId << " login";
    // Thuật toán xác thực, đọc cơ sở dữ liệu, xác minh, xác định, v.v.
    //...
    Json::Value ret;
    ret["result"] = "ok";
    ret["token"] = drogon::utils::getUuid();
    auto resp = HttpResponse::newHttpJsonResponse(ret);
    callback(resp);
}

void User::getInfo(const HttpRequestPtr &req,
                   std::function<void(const HttpResponsePtr &)> &&callback,
                   std::string userId,
                   const std::string &token) const
{
    LOG_DEBUG << "User " << userId << " get his information";

    // Xác minh tính hợp lệ của mã thông báo, v.v.
    // Đọc cơ sở dữ liệu hoặc bộ nhớ cache để lấy thông tin người dùng
    Json::Value ret;
    ret["result"] = "ok";
    ret["user_name"] = "Jack";
    ret["user_id"] = userId;
    ret["gender"] = 1;
    auto resp = HttpResponse::newHttpJsonResponse(ret);
    callback(resp);
}
```

Mỗi lớp `HttpController` có thể định nghĩa nhiều trình xử lý yêu cầu HTTP. Vì số lượng hàm có thể lớn tùy ý, nên việc quá tải chúng bằng các hàm ảo là không thực tế. Chúng ta cần đăng ký chính trình xử lý (không phải lớp) trong framework.

- #### Ánh xạ đường dẫn

  Ánh xạ từ đường dẫn URL đến trình xử lý được thực hiện bằng macro. Bạn có thể thêm bản đồ đa đường dẫn với macro `METHOD_ADD` hoặc macro `ADD_METHOD_TO`. Tất cả các câu lệnh `METHOD_ADD` và `ADD_METHOD_TO` phải được đặt giữa các câu lệnh macro `METHOD_LIST_BEGIN` và `METHOD_LIST_END`.

  Macro `METHOD_ADD` tự động thêm tiền tố namespace và tên lớp vào bản đồ đường dẫn. Do đó, trong ví dụ này, hàm `login` được đăng ký vào đường dẫn `/demo/v1/user/token` và hàm `getInfo` được đăng ký vào đường dẫn `/demo/v1/user/{userId}/info`. Các ràng buộc tương tự như macro `PATH_ADD` của `HttpSimpleController` và không được mô tả ở đây.

  Khi bạn sử dụng macro `ADD_METHOD` và lớp thuộc về một số namespace, bạn nên thêm namespace đó vào URL truy cập. Trong ví dụ này, hãy sử dụng `http://localhost/demo/v1/user/token?userId=xxx&passwd=xxx` hoặc `http://localhost/demo/v1/user/xxxxx/info?token=xxxx`.

  Macro `ADD_METHOD_TO` thực hiện gần như giống với macro trước, ngoại trừ việc nó không tự động thêm bất kỳ tiền tố nào, tức là đường dẫn được đăng ký bởi macro là đường dẫn tuyệt đối.

  Chúng ta thấy rằng `HttpController` cung cấp một cơ chế ánh xạ đường dẫn linh hoạt hơn - chúng ta có thể đặt một lớp các hàm trong một lớp.

  Ngoài ra, bạn có thể thấy rằng các macro cung cấp một phương pháp để ánh xạ tham số. Chúng ta có thể ánh xạ các tham số truy vấn trên đường dẫn đến danh sách tham số của hàm. Số lượng tham số đường dẫn URL tương ứng với vị trí tham số của hàm, điều này rất thuận tiện. Các loại phổ biến có thể được chuyển đổi theo loại chuỗi đều có thể được sử dụng làm tham số hàm (chẳng hạn như `std::string`, `int`, `float`, `double`, v.v.) và framework Drogon sẽ tự động giúp bạn chuyển đổi loại. Điều này rất thuận tiện cho việc phát triển. Lưu ý rằng các tham chiếu lvalue phải thuộc kiểu `const`.

  Cùng một đường dẫn có thể được ánh xạ nhiều lần, phân biệt với nhau bằng Phương thức HTTP, điều này là hợp pháp và là một cách thực hành phổ biến của RESTful API, chẳng hạn như:

  ```c++
  METHOD_LIST_BEGIN
      METHOD_ADD(Book::getInfo,"/{bookId}?detail={2}", Get);
      METHOD_ADD(Book::newBook, "/{bookId}", Post);
      METHOD_ADD(Book::deleteOne, "/{bookId}", Delete);
  METHOD_LIST_END
  ```

  Các trình giữ chỗ của tham số đường dẫn có thể được viết theo nhiều cách:

  - `{}`: Vị trí trên đường dẫn là vị trí của tham số hàm, cho biết tham số đường dẫn ánh xạ tới vị trí tương ứng của các tham số trình xử lý.
  - `{1},{2}`: Các tham số đường dẫn có số trong đó được ánh xạ tới các tham số trình xử lý được chỉ định bởi số.
  - `{anystring}`: Chuỗi ở đây không có tác dụng thực tế, nhưng có thể cải thiện khả năng đọc của chương trình. Tương đương với `{}`.
  - `{1:anystring},{2:xxx}`: Số trước dấu hai chấm đại diện cho vị trí. Chuỗi sau dấu hai chấm không có tác dụng nhưng có thể cải thiện khả năng đọc của chương trình. Tương đương với `{1}` và `{2}`.

  Hai cách viết sau được khuyến nghị, và nếu các tham số đường dẫn và tham số hàm theo cùng một thứ tự, thì cách viết thứ ba là đủ. Dễ dàng nhận thấy rằng những điều sau đây là tương đương:

  - `/users/{}/books/{}`
  - `/users/{}/books/{2}`
  - `/users/{user_id}/books/{book_id}`
  - `/users/{1:user_id}/books/{2}`

  > **Lưu ý: So khớp đường dẫn không phân biệt chữ hoa chữ thường, nhưng tên tham số phân biệt chữ hoa chữ thường. Giá trị tham số có thể được trộn lẫn chữ hoa và chữ thường và được chuyển đến bộ điều khiển mà không thay đổi.**

- #### Ánh xạ tham số

  Thông qua mô tả trước đó, chúng ta biết rằng các tham số trên đường dẫn và các tham số truy vấn sau dấu hỏi chấm có thể được ánh xạ tới danh sách tham số của hàm xử lý. Kiểu của tham số đích cần đáp ứng các điều kiện sau:

  - Phải là một trong các kiểu giá trị, tham chiếu giá trị trái hằng số, hoặc tham chiếu giá trị phải không phải hằng số. Nó không thể là tham chiếu lvalue không phải hằng số. Nên sử dụng tham chiếu rvalue để người dùng có thể tùy ý xử lý nó.

  - Các kiểu cơ bản như `int`, `long`, `long long`, `unsigned long`, `unsigned long long`, `float`, `double`, `long double`, v.v. có thể được sử dụng làm kiểu tham số.

  - `std::string`

  - Bất kỳ kiểu nào có thể được gán bằng toán tử `stringstream >>`.

  > **Ngoài ra, framework Drogon cũng cung cấp một cơ chế ánh xạ từ đối tượng `HttpRequestPtr` sang bất kỳ kiểu tham số nào.**. Khi số lượng tham số ánh xạ trong danh sách tham số trình xử lý của bạn nhiều hơn số lượng tham số trên đường dẫn, các tham số bổ sung sẽ được chuyển đổi từ đối tượng `HttpRequestPtr`. Người dùng có thể định nghĩa bất kỳ loại chuyển đổi nào. Cách để định nghĩa chuyển đổi này là chuyên biệt hóa template `fromRequest` (được định nghĩa trong tệp tiêu đề `HttpRequest.h`) trong namespace `drogon`, ví dụ: giả sử chúng ta cần tạo một giao diện RESTful để tạo người dùng mới, chúng ta định nghĩa cấu trúc của người dùng như sau:

  ```c++
  namespace myapp {
  struct User {
      std::string userName;
      std::string email;
      std::string address;
  };
  } // namespace myapp

  namespace drogon
  {
  template <>
  inline myapp::User fromRequest(const HttpRequest &req)
  {
      auto json = req.getJsonObject();
      myapp::User user;
      if (json)
      {
          user.userName = (*json)["name"].asString();
          user.email = (*json)["email"].asString();
          user.address = (*json)["address"].asString();
      }
      return user;
  }
  } // namespace drogon
  ```

  Với định nghĩa và chuyên biệt hóa template ở trên, chúng ta có thể định nghĩa bản đồ đường dẫn và trình xử lý như sau:

  ```c++
  class UserController : public drogon::HttpController<UserController>
  {
  public:
      METHOD_LIST_BEGIN
          // Sử dụng METHOD_ADD để thêm hàm xử lý tùy chỉnh của bạn vào đây;
          ADD_METHOD_TO(UserController::newUser, "/users", Post);
      METHOD_LIST_END
      // Khai báo hàm xử lý của bạn có thể như thế này:
      void newUser(const HttpRequestPtr &req,
                  std::function<void(const HttpResponsePtr &)> &&callback,
                  myapp::User &&pNewUser) const;
  };
  ```

  Có thể thấy rằng tham số thứ ba của kiểu `myapp::User` không có trình giữ chỗ tương ứng trên đường dẫn ánh xạ, và framework coi nó như một tham số được chuyển đổi từ đối tượng `req` và lấy tham số này thông qua template hàm do người dùng chuyên biệt hóa. Điều này rất thuận tiện cho người dùng.

  Hơn nữa, một số người dùng không cần truy cập đối tượng `HttpRequestPtr` ngoại trừ dữ liệu kiểu tùy chỉnh của họ. Họ có thể đặt đối tượng tùy chỉnh vào vị trí của tham số đầu tiên, và framework sẽ hoàn thành chính xác ánh xạ chẳng hạn như ví dụ trên. Nó cũng có thể được viết như sau:

  ```c++
  class UserController : public drogon::HttpController<UserController>
  {
  public:
      METHOD_LIST_BEGIN
          // Sử dụng METHOD_ADD để thêm hàm xử lý tùy chỉnh của bạn vào đây;
          ADD_METHOD_TO(UserController::newUser, "/users", Post);
      METHOD_LIST_END
      // Khai báo hàm xử lý của bạn có thể như thế này:
      void newUser(myapp::User &&pNewUser,
                  std::function<void(const HttpResponsePtr &)> &&callback) const;
  };
  ```

- #### Ánh xạ nhiều đường dẫn

  Drogon hỗ trợ việc sử dụng các biểu thức chính quy trong ánh xạ đường dẫn, có thể được sử dụng bên ngoài dấu ngoặc nhọn `{}`. Ví dụ:

  ```c++
  ADD_METHOD_TO(UserController::handler1, "/users/.*", Post); /// Khớp với bất kỳ đường dẫn nào có tiền tố là `/users/`
  ADD_METHOD_TO(UserController::handler2, "/{name}/[0-9]+", Post); /// Khớp với bất kỳ đường dẫn nào được tạo thành từ một chuỗi tên và một số.
  ```

- #### Ánh xạ biểu thức chính quy

  Phương pháp trên có hỗ trợ hạn chế đối với các biểu thức chính quy. Nếu người dùng muốn sử dụng biểu thức chính quy một cách tự do, Drogon cung cấp macro `ADD_METHOD_VIA_REGEX` để đạt được điều này, chẳng hạn như:

  ```c++
  ADD_METHOD_VIA_REGEX(UserController::handler1, "/users/(.*)", Post); /// Khớp với bất kỳ đường dẫn nào có tiền tố là `/users/` và ánh xạ phần còn lại của đường dẫn đến một tham số của handler1.
  ADD_METHOD_VIA_REGEX(UserController::handler2, "/.*([0-9]*)", Post); /// Khớp với bất kỳ đường dẫn nào kết thúc bằng một số và ánh xạ số đó đến một tham số của handler2.
  ADD_METHOD_VIA_REGEX(UserController::handler3, "/(?!data).*", Post); /// Khớp với bất kỳ đường dẫn nào không bắt đầu bằng '/data'
  ```

  Như có thể thấy, ánh xạ tham số cũng có thể được thực hiện bằng cách sử dụng biểu thức chính quy, và tất cả các chuỗi được khớp bởi các biểu thức con sẽ được ánh xạ tới các tham số của trình xử lý theo thứ tự.

  > **Cần lưu ý rằng khi sử dụng biểu thức chính quy, bạn nên chú ý đến các xung đột khớp (nhiều trình xử lý khác nhau được khớp). Khi xung đột xảy ra trong cùng một bộ điều khiển, Drogon sẽ chỉ thực thi trình xử lý đầu tiên (trình xử lý được đăng ký trong framework trước). Khi xung đột xảy ra giữa các bộ điều khiển khác nhau, không chắc chắn trình xử lý nào sẽ được thực thi. Do đó, người dùng cần tránh những xung đột này.**


# Tiếp theo: [WebSocketController](VI-04-3-Controller-WebSocketController)
