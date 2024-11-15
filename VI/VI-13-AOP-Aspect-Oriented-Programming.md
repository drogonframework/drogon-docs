## Lập trình hướng khía cạnh (AOP)

AOP (Lập trình hướng khía cạnh) là một mô hình lập trình nhằm mục đích tăng tính module hóa bằng cách cho phép tách biệt các mối quan tâm cắt ngang (cross-cutting concerns, trích dẫn từ Wikipedia).

Do những hạn chế của ngôn ngữ C++, Drogon không cung cấp một giải pháp AOP linh hoạt như Spring, mà là một AOP đơn giản trong đó tất cả các joinpoint được xác định trước trong framework, và bằng cách sử dụng chuỗi giao diện AOP của framework, người ta có thể đăng ký các handler (được gọi là 'advice' trong Drogon) vào các joinpoint cụ thể.

### Các joinpoint được xác định trước

Drogon cung cấp bảy joinpoint cho người dùng. Khi ứng dụng chạy đến các joinpoint, các handler do người dùng đăng ký (Advice) được gọi lần lượt. Mô tả về các joinpoint này như sau:

- **Beginning**: Như tên gọi của nó, joinpoint này nằm ở đầu chương trình. Cụ thể, tất cả các handler được đăng ký tại joinpoint này được thực thi ngay sau khi phương thức `app().run()` đã hoàn thành khởi tạo. Tại joinpoint này, tất cả các controller, filter, plugin và client cơ sở dữ liệu đã được build hoàn chỉnh, người dùng có thể nhận được tham chiếu đối tượng mong muốn hoặc thực hiện một số công việc khởi tạo khác ở đây. Advice trên joinpoint này chỉ được chạy một lần, chữ ký cuộc gọi của advice là `void()`, giao diện đăng ký là `registerBeginningAdvice`.

- **NewConnection**: Advice được đăng ký vào joinpoint này được gọi khi mỗi kết nối TCP mới được thiết lập. Chữ ký cuộc gọi của advice là `bool(const trantor::InetAddress &, const trantor::InetAddress &)`, trong đó đối số đầu tiên là địa chỉ từ xa của kết nối TCP và đối số thứ hai là địa chỉ cục bộ. Lưu ý rằng kiểu trả về là `bool`, nếu người dùng trả về `false`, kết nối tương ứng sẽ bị ngắt kết nối. Giao diện đăng ký là `registerNewConnectionAdvice`.

- **HttpResponseCreation**: Advice được đăng ký vào joinpoint này được gọi khi mỗi đối tượng HTTP Response được tạo. Chữ ký cuộc gọi của Advice là `void(const HttpResponsePtr &)`, trong đó tham số là đối tượng mới được tạo, và người dùng có thể thực hiện một số thao tác thống nhất trên tất cả các Response với joinpoint này, chẳng hạn như thêm một header đặc biệt, v.v. Joinpoint này ảnh hưởng đến tất cả các Response, bao gồm 404 hoặc bất kỳ response lỗi nội bộ nào của Drogon, đồng thời bao gồm tất cả các response do ứng dụng của người dùng tạo ra. Giao diện đăng ký là `registerHttpResponseCreationAdvice`.

- **Sync**: Joinpoint này nằm ở phía front-end của quá trình xử lý yêu cầu HTTP. Người dùng có thể chặn yêu cầu này bằng cách trả về một đối tượng Response không rỗng. Chữ ký cuộc gọi của Advice là `HttpRequestPtr(const HttpRequestPtr &)`. Giao diện đăng ký là `registerSyncAdvice`.

- **Pre-Routing**: Advice được đăng ký vào joinpoint này được gọi ngay sau khi yêu cầu được tạo và trước khi nó khớp với bất kỳ đường dẫn handler nào. Advice cho joinpoint này có hai chữ ký cuộc gọi, `void(const HttpRequestPtr &,AdviceCallback &&,AdviceChainCallback &&)` và `void(const HttpRequestPtr &)`, kiểu gọi trước giống hệt với chữ ký cuộc gọi của phương thức `doFilter` của filter. Trên thực tế, tất cả chúng đều chạy theo cùng một cách (vui lòng tham khảo [05-Filter]), người dùng có thể chặn yêu cầu của client hoặc để nó đi qua joinpoint này. Advice với chữ ký cuộc gọi thứ hai không có khả năng chặn, nhưng chi phí của nó thấp hơn, nếu người dùng không có ý định chặn yêu cầu, vui lòng chọn loại advice này. Giao diện đăng ký là `registerPreRoutingAdvice`.

- **Post-Routing**: Advice được đăng ký vào joinpoint này được gọi ngay sau khi yêu cầu khớp với đường dẫn handler. Chữ ký cuộc gọi của Advice giống như joinpoint ở trên. Giao diện đăng ký là `registerPostRoutingAdvice`.

- **Pre-Handling**: Advice được đăng ký vào joinpoint này được gọi ngay sau khi yêu cầu được tất cả các filter phê duyệt và trước khi nó được xử lý. Chữ ký cuộc gọi của Advice giống như joinpoint ở trên. Giao diện đăng ký là `registerPostRoutingAdvice`.

- **Post-Handling**: Advice được đăng ký vào joinpoint này được gọi ngay sau khi yêu cầu được xử lý và một đối tượng response được tạo bởi handler. Chữ ký cuộc gọi của Advice là `void(const HttpRequestPtr &, const HttpResponsePtr &)`. Giao diện đăng ký là `registerPostHandlingAdvice`.

### Sơ đồ AOP

Hình sau đây cho thấy vị trí của bốn joinpoint ở trên trong luồng xử lý Yêu cầu HTTP, trong đó các chấm đỏ biểu thị các joinpoint và các mũi tên màu xanh lá cây biểu thị các cuộc gọi bất đồng bộ.

![](images/AOP.png)

# 13 [Điểm chuẩn](VI-13-Benchmarks)
