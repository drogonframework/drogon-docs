## Drogon

**Drogon** là một framework ứng dụng HTTP dựa trên C++17/20. Drogon có thể được sử dụng để dễ dàng xây dựng nhiều loại chương trình máy chủ ứng dụng web khác nhau bằng C++.

**Drogon** là tên của một con rồng trong bộ phim truyền hình Mỹ "Game of Thrones" mà tôi rất thích.

Nền tảng ứng dụng chính của Drogon là Linux. Nó cũng hỗ trợ Mac OS, FreeBSD và Windows.

Các tính năng chính của nó như sau:

* Sử dụng thư viện mạng I/O không chặn dựa trên epoll (kqueue trên macOS/FreeBSD) để cung cấp I/O mạng hiệu năng cao, đồng thời. Vui lòng truy cập [Kết quả kiểm tra TFB](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite) để biết thêm chi tiết;
* Cung cấp chế độ lập trình hoàn toàn bất đồng bộ;
* Hỗ trợ HTTP 1.0/1.1 (phía máy chủ và phía máy khách);
* Dựa trên template, một cơ chế reflection đơn giản được triển khai để tách rời hoàn toàn framework chương trình chính, bộ điều khiển (controller) và view;
* Hỗ trợ cookie và phiên làm việc tích hợp;
* Hỗ trợ render back-end, bộ điều khiển tạo dữ liệu cho view để tạo trang HTML. View được mô tả bởi các tệp template CSP, mã C++ được nhúng vào các trang HTML thông qua các thẻ CSP. Công cụ dòng lệnh drogon tự động tạo các tệp mã C++ để biên dịch;
* Hỗ trợ tải động trang view (biên dịch và tải động trong thời gian chạy);
* Cung cấp giải pháp định tuyến thuận tiện và linh hoạt từ đường dẫn đến trình xử lý bộ điều khiển;
* Hỗ trợ chuỗi bộ lọc (filter chain) để tạo điều kiện thuận lợi cho việc thực thi logic thống nhất (chẳng hạn như xác minh đăng nhập, xác minh ràng buộc phương thức HTTP, v.v.) trước khi xử lý các yêu cầu HTTP;
* Hỗ trợ HTTPS (dựa trên OpenSSL);
* Hỗ trợ WebSocket (phía máy chủ và phía máy khách);
* Hỗ trợ yêu cầu và phản hồi định dạng JSON, rất thân thiện với việc phát triển ứng dụng Restful API;
* Hỗ trợ tải xuống và tải lên tệp;
* Hỗ trợ truyền nén gzip, brotli;
* Hỗ trợ pipelining;
* Cung cấp một công cụ dòng lệnh gọn nhẹ, `drogon_ctl`, để đơn giản hóa việc tạo các lớp khác nhau trong Drogon và tạo mã view;
* Hỗ trợ đọc và ghi cơ sở dữ liệu bất đồng bộ dựa trên I/O không chặn (cơ sở dữ liệu PostgreSQL và MySQL (MariaDB));
* Hỗ trợ đọc và ghi cơ sở dữ liệu SQLite3 bất đồng bộ dựa trên thread pool;
* Hỗ trợ kiến trúc ARM;
* Cung cấp một triển khai ORM gọn nhẹ, tiện lợi hỗ trợ ánh xạ hai chiều đối tượng-cơ sở dữ liệu thông thường;
* Hỗ trợ các plugin có thể được cài đặt bởi tệp cấu hình tại thời điểm tải;
* Hỗ trợ AOP với các joinpoint tích hợp sẵn.

# Tiếp theo: [Cài đặt drogon](VI-02-Installation)