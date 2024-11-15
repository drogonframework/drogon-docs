## WebSocketController

Như tên gọi của nó, `WebSocketController` được sử dụng để xử lý logic WebSocket. WebSocket là một lược đồ kết nối dựa trên HTTP liên tục. Khi bắt đầu WebSocket, có một yêu cầu và phản hồi trao đổi định dạng HTTP. Sau khi kết nối WebSocket được thiết lập, tất cả các tin nhắn được truyền trên WebSocket. Tin nhắn được gói gọn trong một định dạng cố định. Không có giới hạn về nội dung tin nhắn và thứ tự truyền tin nhắn.

### Tạo

Tệp nguồn của `WebSocketController` có thể được tạo bằng công cụ `drogon_ctl`. Định dạng lệnh như sau:

```shell
drogon_ctl create controller -w <[namespace::]class_name>
```

Giả sử chúng ta muốn triển khai một hàm echo đơn giản thông qua WebSocket, tức là máy chủ chỉ cần gửi lại tin nhắn do máy khách gửi. Chúng ta có thể tạo lớp triển khai `EchoWebsock` của `WebSocketController` thông qua `drogon_ctl`, như sau:

```shell
drogon_ctl create controller -w EchoWebsock
```

Lệnh này sẽ tạo ra hai tệp `EchoWebsock.h` và `EchoWebsock.cc`, như sau:

```c++
//EchoWebsock.h
#pragma once
#include <drogon/WebSocketController.h>
using namespace drogon;
class EchoWebsock : public drogon::WebSocketController<EchoWebsock>
{
  public:
    void handleNewMessage(const WebSocketConnectionPtr&,
                          std::string &&,
                          const WebSocketMessageType &) override;
    void handleNewConnection(const HttpRequestPtr &,
                             const WebSocketConnectionPtr&) override;
    void handleConnectionClosed(const WebSocketConnectionPtr&) override;
    WS_PATH_LIST_BEGIN
    // Liệt kê các định nghĩa đường dẫn ở đây;
    WS_PATH_LIST_END
};
```

```c++
//EchoWebsock.cc
#include "EchoWebsock.h"
void EchoWebsock::handleNewMessage(const WebSocketConnectionPtr &wsConnPtr, std::string &&message, const WebSocketMessageType &type)
{
    // Viết logic ứng dụng của bạn ở đây
}
void EchoWebsock::handleNewConnection(const HttpRequestPtr &req, const WebSocketConnectionPtr &wsConnPtr)
{
    // Viết logic ứng dụng của bạn ở đây
}
void EchoWebsock::handleConnectionClosed(const WebSocketConnectionPtr &wsConnPtr)
{
    // Viết logic ứng dụng của bạn ở đây
}
```

### Sử dụng

- #### Ánh xạ đường dẫn

  Sau khi chỉnh sửa:

  ```c++
  //EchoWebsock.h
  #pragma once
  #include <drogon/WebSocketController.h>
  using namespace drogon;
  class EchoWebsock : public drogon::WebSocketController<EchoWebsock>
  {
  public:
      void handleNewMessage(const WebSocketConnectionPtr&,
                            std::string &&,
                            const WebSocketMessageType &) override;
      void handleNewConnection(const HttpRequestPtr &,
                              const WebSocketConnectionPtr&) override;
      void handleConnectionClosed(const WebSocketConnectionPtr&) override;
      WS_PATH_LIST_BEGIN
      // Liệt kê các định nghĩa đường dẫn ở đây;
      WS_PATH_ADD("/echo");
      WS_PATH_LIST_END
  };
  ```

  ```c++
  //EchoWebsock.cc
  #include "EchoWebsock.h"
  void EchoWebsock::handleNewMessage(const WebSocketConnectionPtr &wsConnPtr, std::string &&message, const WebSocketMessageType &type)
  {
      // Viết logic ứng dụng của bạn ở đây
      wsConnPtr->send(message);
  }
  void EchoWebsock::handleNewConnection(const HttpRequestPtr &req, const WebSocketConnectionPtr &wsConnPtr)
  {
      // Viết logic ứng dụng của bạn ở đây
  }
  void EchoWebsock::handleConnectionClosed(const WebSocketConnectionPtr &wsConnPtr)
  {
      // Viết logic ứng dụng của bạn ở đây
  }
  ```

  Đầu tiên, trong ví dụ này, bộ điều khiển được đăng ký vào đường dẫn `/echo` thông qua macro `WS_PATH_ADD`. Cách sử dụng macro `WS_PATH_ADD` tương tự như các macro của các bộ điều khiển khác đã giới thiệu trước đó. Người ta cũng có thể đăng ký đường dẫn với một số [Bộ lọc](VI-05-Middleware-and-Filter). Vì WebSocket được xử lý riêng biệt trong framework, nên nó có thể được lặp lại với các đường dẫn của hai bộ điều khiển đầu tiên (`HttpSimpleController` và `HttpApiController`) mà không ảnh hưởng lẫn nhau.

  Thứ hai, trong việc triển khai ba hàm ảo trong ví dụ này, chỉ có `handleNewMessage` là có nội dung, nhưng chỉ đơn giản là gửi tin nhắn nhận được trở lại máy khách thông qua giao diện gửi. Biên dịch bộ điều khiển này vào framework, bạn có thể thấy hiệu ứng, vui lòng tự mình kiểm tra.

  **Lưu ý: Giống như giao thức HTTP thông thường, HTTP WebSocket có thể bị đánh hơi. Nếu yêu cầu bảo mật, nên cung cấp mã hóa bằng HTTPS. Tất nhiên, người dùng cũng có thể hoàn thành mã hóa và giải mã ở phía máy chủ và máy khách, nhưng HTTPS thuận tiện hơn. Lớp bên dưới được xử lý bởi `Drogon` và người dùng chỉ cần quan tâm đến logic nghiệp vụ.**

  Lớp bộ điều khiển WebSocket do người dùng định nghĩa kế thừa từ template lớp `drogon::WebSocketController`. Tham số template là một kiểu lớp con. Người dùng cần triển khai ba hàm ảo sau để xử lý việc thiết lập, tắt máy và tin nhắn của WebSocket:

  ```c++
  virtual void handleNewConnection(const HttpRequestPtr &req, const WebSocketConnectionPtr &wsConn);
  virtual void handleNewMessage(const WebSocketConnectionPtr &wsConn, std::string &&message,
                               const WebSocketMessageType &type);
  virtual void handleConnectionClosed(const WebSocketConnectionPtr &wsConn);
  ```

  Dễ dàng biết được:

  - `handleNewConnection` được gọi sau khi WebSocket được thiết lập. `req` là yêu cầu thiết lập do máy khách gửi. Tại thời điểm này, framework đã trả về phản hồi. Những gì người dùng có thể làm là nhận thêm thông tin thông qua `req`, chẳng hạn như `token`. `wsConn` là một con trỏ thông minh đến đối tượng WebSocket này, và giao diện thường được sử dụng sẽ được thảo luận sau.
  - `handleNewMessage` được gọi sau khi WebSocket nhận được tin nhắn mới. Tin nhắn được lưu trữ trong biến `message`. Lưu ý rằng tin nhắn là payload của tin nhắn. Framework đã hoàn thành việc giải nén và giải mã tin nhắn. Người dùng có thể trực tiếp xử lý chính tin nhắn.
  - `handleConnectionClosed` được gọi sau khi kết nối WebSocket bị đóng, và người dùng có thể thực hiện một số công việc hoàn thiện.

### Giao diện

    Các giao diện phổ biến của đối tượng `WebSocketConnection` như sau:

    ```c++
    // Gửi một tin nhắn WebSocket, mã hóa và đóng gói
    // tin nhắn là trách nhiệm của framework
    void send(const char *msg, uint64_t len);
    void send(const std::string &msg);

    // Địa chỉ cục bộ và từ xa của WebSocket
    const trantor::InetAddress &localAddr() const;
    const trantor::InetAddress &peerAddr() const;

    // Trạng thái kết nối của WebSocket
    bool connected() const;
    bool disconnected() const;

    // Đóng WebSocket
    void shutdown(); // Đóng ghi
    void forceClose(); // Đóng

    // Thiết lập và lấy ngữ cảnh của WebSocket, và lưu trữ một số dữ liệu nghiệp vụ từ người dùng.
    // Kiểu any có nghĩa là bạn có thể lưu trữ bất kỳ kiểu đối tượng nào.
    void setContext(const any &context);
    const any &getContext() const;
    any *getMutableContext();
    ```

# Tiếp theo: [Middleware và Filter](VI-05-Middleware-and-Filter)


