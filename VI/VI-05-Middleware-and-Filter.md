## Middleware và Filter

Trong [ví dụ](VI-04-2-Controller-HttpController) về `HttpController`, phương thức `getInfo` nên kiểm tra xem người dùng đã đăng nhập hay chưa trước khi trả về thông tin của người dùng. Chúng ta có thể viết logic này trong phương thức `getInfo`, nhưng rõ ràng, việc kiểm tra tư cách thành viên đăng nhập của người dùng là logic chung sẽ được sử dụng bởi nhiều giao diện, nó nên được trích xuất riêng biệt và được cấu hình trước khi gọi handler, đó là những gì filter thực hiện.

Middleware của Drogon sử dụng mô hình onion (hành tây). Sau khi framework hoàn thành việc khớp đường dẫn URL, nó sẽ gọi tuần tự middleware đã đăng ký cho đường dẫn đó. Trong mỗi middleware, người dùng có thể chọn chặn hoặc chuyển qua yêu cầu và thêm logic xử lý trước và xử lý sau.

Nếu một middleware chặn một yêu cầu, nó sẽ không tiếp tục đến các lớp bên trong của onion, và handler tương ứng sẽ không được gọi. Tuy nhiên, nó vẫn sẽ trải qua logic xử lý sau của middleware lớp ngoài.

Filter, trên thực tế, là middleware bỏ qua thao tác xử lý sau. Middleware và filter có thể được sử dụng kết hợp khi đăng ký đường dẫn.

### Middleware/Filter tích hợp sẵn

Drogon chứa các filter phổ biến sau:

- `drogon::IntranetIpFilter`: chỉ cho phép các yêu cầu HTTP từ IP mạng nội bộ, hoặc trả về trang 404.
- `drogon::LocalHostFilter`: chỉ cho phép các yêu cầu HTTP từ 127.0.0.1 hoặc ::1, hoặc trả về trang 404.

### Middleware/Filter tùy chỉnh

- #### Định nghĩa Middleware

  Người dùng có thể tùy chỉnh middleware, bạn cần kế thừa từ template lớp `HttpMiddleware`, kiểu template là kiểu lớp con, ví dụ: nếu bạn muốn bật hỗ trợ cross-origin cho một số route, bạn có thể định nghĩa nó như sau:

  ```c++
  class MyMiddleware : public HttpMiddleware<MyMiddleware>
  {
  public:
      MyMiddleware(){};  // Không bỏ qua hàm tạo

      void invoke(const HttpRequestPtr &req,
                  MiddlewareNextCallback &&nextCb,
                  MiddlewareCallback &&mcb) override
      {
          const std::string &origin = req->getHeader("origin");
          if (origin.find("www.some-evil-place.com") != std::string::npos)
          {
              // Chặn trực tiếp
              mcb(HttpResponse::newNotFoundResponse(req));
              return;
          }
          // Làm gì đó trước khi gọi middleware tiếp theo
          nextCb([mcb = std::move(mcb)](const HttpResponsePtr &resp) {
              // Làm gì đó sau khi middleware tiếp theo trả về
              resp->addHeader("Access-Control-Allow-Origin", origin);
              resp->addHeader("Access-Control-Allow-Credentials", "true");
              mcb(resp);
          });
      }
  };
  ```

  Bạn cần ghi đè hàm ảo `invoke` của lớp cha để triển khai logic của middleware;

  Hàm ảo này có ba tham số, đó là:

  - **req**: Yêu cầu HTTP;
  * **nextCb**: Hàm gọi lại để vào lớp bên trong của onion. Gọi hàm này có nghĩa là gọi middleware tiếp theo hoặc handler cuối cùng. Khi gọi `nextCb`, nó chấp nhận một hàm khác làm tham số. Hàm này sẽ được gọi khi trả về từ các lớp bên trong, và `HttpResponsePtr` được trả về từ các lớp bên trong sẽ được chuyển làm đối số cho hàm này.
  * **mcb**: Hàm gọi lại để quay lại lớp trên của onion. Gọi hàm này có nghĩa là quay lại lớp ngoài của onion. Nếu `nextCb` bị bỏ qua và chỉ gọi `mcb`, thì có nghĩa là chặn yêu cầu và trực tiếp quay lại lớp trên.

- #### Định nghĩa Filter

  Tất nhiên, người dùng có thể tùy chỉnh filter, bạn cần kế thừa từ template lớp `HttpFilter`, kiểu template là kiểu lớp con, ví dụ: nếu bạn muốn tạo `LoginFilter`, bạn có thể định nghĩa nó như sau:

  ```c++
  class LoginFilter : public drogon::HttpFilter<LoginFilter>
  {
  public:
      void doFilter(const HttpRequestPtr &req,
                    FilterCallback &&fcb,
                    FilterChainCallback &&fccb) override;
  };
  ```

  Bạn có thể tạo filter bằng lệnh `drogon_ctl`, xem [drogon_ctl](VI-11-drogon_ctl-command#Filter-creation).

  Bạn cần ghi đè hàm ảo `doFilter` của lớp cha để triển khai logic của filter;

  Hàm ảo này có ba tham số, đó là:

  - **req**: Yêu cầu HTTP;
  - **fcb**: Hàm gọi lại filter, kiểu hàm là `void(HttpResponsePtr)`, khi filter xác định rằng yêu cầu không hợp lệ, phản hồi cụ thể sẽ được trả về trình duyệt thông qua hàm gọi lại này;
  - **fccb**: Hàm gọi lại chuỗi filter, kiểu hàm là `void()`, khi filter xác định rằng yêu cầu là hợp lệ, hãy thông báo cho Drogon gọi filter tiếp theo hoặc handler cuối cùng thông qua hàm gọi lại này;

  Việc triển khai cụ thể có thể tham khảo việc triển khai filter tích hợp sẵn của Drogon.

- #### Đăng ký Middleware/Filter

  Việc đăng ký middleware/filter luôn đi kèm với việc đăng ký bộ điều khiển. Các macro (`PATH_ADD`, `METHOD_ADD`, v.v.) đã đề cập trước đó có thể thêm tên của một hoặc nhiều middleware/filter ở cuối; ví dụ: chúng ta thay đổi dòng đăng ký của phương thức `getInfo` trước đó thành dạng sau:

  ```c++
  METHOD_ADD(User::getInfo,"/{userId}/info?token={token}",Get,"LoginFilter","MyMiddleware");
  ```

  Sau khi đường dẫn được khớp thành công, phương thức `getInfo` sẽ chỉ được gọi khi đáp ứng các điều kiện sau:

  1. Yêu cầu phải là yêu cầu HTTP GET;
  2. Bên yêu cầu phải đã đăng nhập;

  Như bạn có thể thấy, việc cấu hình và đăng ký middleware/filter rất đơn giản. Tệp nguồn bộ điều khiển đăng ký middleware không cần include tệp tiêu đề của middleware. Middleware và bộ điều khiển được tách rời hoàn toàn.

  > **Lưu ý: Nếu middleware/filter được định nghĩa trong namespace, bạn phải viết đầy đủ namespace khi đăng ký.**


# Tiếp theo: [View](VI-06-View)


