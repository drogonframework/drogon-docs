## Tệp cấu hình

Bạn có thể kiểm soát các hành vi khác nhau của máy chủ HTTP bằng cách cấu hình các tham số khác nhau thông qua nhiều giao diện của instance `DrogonAppFramework`. Tuy nhiên, sử dụng tệp cấu hình là một cách tốt hơn vì những lý do sau:

- Sử dụng tệp cấu hình thay vì mã nguồn có thể xác định hành vi của ứng dụng trong thời gian chạy (runtime) chứ không phải trong thời gian biên dịch (compile time), điều này chắc chắn là một cách thuận tiện và linh hoạt hơn;
- Sử dụng tệp cấu hình có thể làm cho tệp chính ngắn gọn hơn;

Dựa trên những lợi ích bổ sung này, bạn nên sử dụng tệp cấu hình để cấu hình các tham số khác nhau của ứng dụng.

Tệp cấu hình có thể được tải rất đơn giản bằng cách gọi giao diện `loadConfigFile()` trước khi gọi giao diện `run()`. Tham số của phương thức `loadConfigFile()` là tên tệp của tệp cấu hình, ví dụ:

```c++
int main()
{
    drogon::app().loadConfigFile("config.json");
    drogon::app().run();
}
```

Chương trình trên tải tệp cấu hình `config.json` và sau đó chạy ứng dụng. Cổng lắng nghe (listening port) cụ thể, output log, cấu hình cơ sở dữ liệu, v.v. có thể được cấu hình bởi tệp cấu hình. Trên thực tế, chương trình này về cơ bản có thể là toàn bộ mã của tệp chính (main file) của ứng dụng web.

### Chi tiết tệp cấu hình

Một ví dụ về tệp cấu hình nằm ở cấp cao nhất của thư mục nguồn, `config.example.json`. Nếu bạn sử dụng lệnh `drogon_ctl create project` để tạo dự án, bạn cũng có thể tìm thấy tệp `config.json` có nội dung tương tự trong thư mục dự án. Vì vậy, về cơ bản bạn không cần tạo một tệp cấu hình mới mà chỉ cần thực hiện một số thay đổi đối với tệp này để hoàn tất cấu hình của ứng dụng web.

Tệp ở định dạng `JSON` và hỗ trợ comment. Bạn có thể comment bỏ qua các mục cấu hình không cần thiết bằng các ký hiệu comment C++ `/**/` và `//`.

Sau khi comment bỏ qua một tùy chọn cấu hình, framework sẽ khởi tạo nó với các giá trị mặc định. Giá trị mặc định cho mỗi tùy chọn có thể được tìm thấy trong các comment trong tệp cấu hình.

### Định dạng được hỗ trợ
* `json`
* `yaml`, nên cài đặt thư viện `yaml-cpp` để cung cấp trình phân tích cú pháp (parser) tệp YAML.

- ### `ssl`

  Tùy chọn `ssl` là để cấu hình các tệp SSL của dịch vụ HTTPS như sau:

  ```json
    "ssl": {
      "cert": "../../trantor/trantor/tests/server.pem",
      "key": "../../trantor/trantor/tests/server.pem",
      "conf":[
        ["Options", "Compression"],
        ["min_protocol", "TLSv1.2"]
      ]
    }
  ```

  Trong đó `cert` là đường dẫn của tệp chứng chỉ (certificate file) và `key` là đường dẫn của tệp khóa riêng (private key file). Nếu một tệp chứa cả chứng chỉ và khóa riêng, thì hai đường dẫn có thể được tạo giống nhau. Tệp ở định dạng mã hóa PEM.

  `conf` là các tùy chọn SSL tùy chọn được truyền trực tiếp vào [SSL_CONF_cmd](https://www.openssl.org/docs/manmaster/man3/SSL_CONF_cmd.html) để cho phép cấu hình mã hóa ở mức thấp. Các tùy chọn phải là mảng một hoặc hai phần tử.

- ### `listeners`

  Như tên gọi của nó, tùy chọn `listeners` là để cấu hình các listener cho ứng dụng web. Nó là một kiểu mảng JSON. Mỗi đối tượng JSON đại diện cho một listener. Cấu hình cụ thể như sau:

  ```json
  "listeners": [
      {
        "address": "0.0.0.0",
        "port": 80,
        "https": false
      },
      {
        "address": "0.0.0.0",
        "port": 443,
        "https": true,
        "cert": "",
        "key": ""
      }
    ]
  ```

  Trong đó:

  - `address`: Với kiểu chuỗi cho biết địa chỉ IP cần được lắng nghe. Nếu tùy chọn này không khả dụng, giá trị mặc định "0.0.0.0" sẽ được sử dụng.
  - `port`: Một tùy chọn kiểu số nguyên cho biết cổng cần được lắng nghe. Nó phải là một số cổng hợp lệ. Không có giá trị mặc định.
  - `https`: Kiểu boolean, cho biết có sử dụng HTTPS hay không, giá trị mặc định là `false`, có nghĩa là sử dụng HTTP.
  - `cert` và `key`: Kiểu chuỗi, có hiệu lực khi `https` là `true`, cho biết chứng chỉ và khóa riêng của HTTPS. Giá trị mặc định là một chuỗi rỗng, cho biết tệp chứng chỉ và khóa riêng được cấu hình bởi tùy chọn toàn cục `ssl`;

- ### `db_clients`

  Tùy chọn này được sử dụng để cấu hình database client. Nó là một kiểu mảng JSON. Mỗi đối tượng JSON đại diện cho một database client riêng biệt. Cấu hình cụ thể như sau:

  ```json
    "db_clients":[
      {
        "name":"",
        "rdbms": "postgresql",
        "host": "127.0.0.1",
        "port": 5432,
        "dbname": "test",
        "user": "",
        "passwd": "",
        "is_fast": false,
        "connection_number": 1,
        "filename": ""
      }
    ]
  ```

  Trong đó:

  - `name`: Kiểu chuỗi, tên client, giá trị mặc định là "default", `name` là dấu hiệu để nhà phát triển ứng dụng nhận database client từ framework, nếu có nhiều client, trường `name` phải khác nhau;
  - `rdbms`: Một chuỗi cho biết loại máy chủ cơ sở dữ liệu. Hiện tại hỗ trợ "postgresql", "mysql" và "sqlite3", không phân biệt chữ hoa chữ thường;
  - `host`: Chuỗi, địa chỉ máy chủ cơ sở dữ liệu, `localhost` là giá trị mặc định;
  - `port`: Một số nguyên đại diện cho số cổng của máy chủ cơ sở dữ liệu;
  - `dbname`: Chuỗi, tên cơ sở dữ liệu;
  - `user`: Chuỗi, tên người dùng;
  - `passwd`: Chuỗi, mật khẩu;
  - `is_fast`: Bool, mặc định là `false`, cho biết client có phải là [`FastDbClient`](VI-08-4-Database-FastDbClient) hay không
  - `connection_number`: Một số nguyên cho biết số lượng kết nối đến máy chủ cơ sở dữ liệu, ít nhất là 1, giá trị mặc định cũng là 1, ảnh hưởng đến hiệu suất concurrency của việc đọc và ghi dữ liệu; Nếu `is_fast` là `true`, số lượng là số lượng kết nối cho mỗi vòng lặp sự kiện, nếu không thì đó là tổng số tất cả các kết nối.
  - `filename`: Tên tệp của cơ sở dữ liệu SQLite3;

- ### `threads_num`

  Tùy chọn con thuộc tùy chọn `app`, một số nguyên, giá trị mặc định là 1, cho biết số lượng luồng I/O, điều này có tác động rõ ràng đến concurrency mạng. Con số này không lớn nhất có thể. Những người dùng hiểu nguyên tắc I/O không chặn nên biết rằng giá trị này nên giống với số lượng bộ xử lý mà họ mong đợi I/O mạng sẽ chiếm dụng. Nếu giá trị được đặt thành 0, số lượng luồng I/O sẽ là số lượng tất cả các lõi phần cứng.

  ```json
  "threads_num": 16,
  ```

  Ví dụ trên cho thấy I/O mạng sử dụng 16 luồng và có thể chạy tối đa 16 lõi CPU trong điều kiện tải cao.

- ### `session`

  Các tùy chọn liên quan đến session cũng là con của tùy chọn `app`, kiểm soát xem session có được sử dụng hay không và thời gian chờ session. Ví dụ:

  ```json
  "enable_session": true,
  "session_timeout": 1200,
  ```

  Trong đó:

  - `enable_session`: Giá trị boolean cho biết có sử dụng session hay không. Mặc định là `false`. Nếu client không hỗ trợ cookie, hãy đặt thành `false` vì framework sẽ tạo một session mới cho mỗi yêu cầu mà không có session cookie, điều này sẽ dẫn đến mất tài nguyên và hiệu suất hoàn toàn không cần thiết;
  - `session_timeout`: Giá trị số nguyên cho biết khoảng thời gian chờ (timeout) của session, tính bằng giây. Giá trị mặc định là 0, cho biết giá trị vĩnh viễn. Chỉ hoạt động nếu `enable_session` là `true`.

- ### `document_root`

  Tùy chọn con của tùy chọn `app`, một chuỗi, cho biết đường dẫn tài liệu tương ứng với thư mục gốc HTTP, và là đường dẫn gốc của quá trình tải xuống tệp tĩnh. Giá trị mặc định là "./", cho biết đường dẫn hiện tại của chương trình đang chạy. Ví dụ:

  ```json
  "document_root": "./",
  ```

- ### `upload_path`

  Con của tùy chọn `app`, một chuỗi, đại diện cho đường dẫn mặc định để upload tệp. Giá trị mặc định là "uploads". Nếu giá trị không bắt đầu bằng `/`, `./` hoặc `../`, và giá trị này không phải là `.` hoặc `..`, thì đường dẫn này là đường dẫn tương đối của mục `document_root` trước đó, nếu không thì đó là một đường dẫn tuyệt đối hoặc một đường dẫn tương đối đến thư mục hiện tại. Ví dụ:

  ```json
  "upload_path": "uploads",
  ```

- ### `client_max_body_size`

  Con của đối tượng `app`, một chuỗi, đại diện cho kích thước tối đa tổng thể của phần thân của yêu cầu.
  Bạn có thể sử dụng các hậu tố `k`, `m`, `g` để chỉ định kilobyte, megabyte hoặc gigabyte (byte * 1024), trên bản build 64 bit, bạn cũng có thể sử dụng `t` cho terabyte. Các hậu tố có thể là chữ thường hoặc chữ hoa.

  ```json
  "client_max_body_size": "10M",
  ```

- ### `client_max_memory_body_size`

  Con của đối tượng `app`, một chuỗi, đại diện cho kích thước bộ đệm tối đa để lưu trữ phần thân yêu cầu trước khi lưu vào bộ nhớ cache vào một tệp.
  Bạn có thể sử dụng các hậu tố `k`, `m`, `g` để chỉ định kilobyte, megabyte hoặc gigabyte (byte * 1024), trên bản build 64 bit, bạn cũng có thể sử dụng `t` cho terabyte. Các hậu tố có thể là chữ thường hoặc chữ hoa.

  ```json
  "client_max_memory_body_size": "50K"
  ```

- ### `file_types`

  Tùy chọn con của tùy chọn `app`, một mảng các chuỗi, với các giá trị mặc định như sau, cho biết loại tải xuống tệp tĩnh được hỗ trợ bởi framework. Nếu phần mở rộng tệp tĩnh được yêu cầu nằm ngoài các loại này, framework sẽ trả về lỗi 404.

  ```json
  "file_types": [
        "gif",
        "png",
        "jpg",
        "js",
        "css",
        "html",
        "ico",
        "swf",
        "xap",
        "apk",
        "cur",
        "xml"
      ],
  ```

- ### `mime`

  Tùy chọn con của tùy chọn ứng dụng, một từ điển từ chuỗi đến chuỗi hoặc một mảng chuỗi. Khai báo cách ánh xạ phần mở rộng tệp với các kiểu MIME mới (tức là cho những kiểu không được nhận dạng theo mặc định) khi gửi tệp tĩnh. Lưu ý rằng tùy chọn này chỉ đăng ký MIME. Framework vẫn gửi 404 nếu phần mở rộng không có trong `file_types` được mô tả ở trên.

  ```json
  "mime": {
    "text/markdown": "md",
    "text/gemini": ["gmi", "gemini"]
  }
  ```

- ### Kiểm soát Số lượng Kết nối

  Con của tùy chọn `app` có hai tùy chọn, như sau:

  ```json
  "max_connections": 100000,
  "max_connections_per_ip": 0,
  ```

  Trong đó:

  - `max_connections`: Số nguyên, giá trị mặc định là 100000, có nghĩa là số lượng kết nối đồng thời tối đa; khi số lượng kết nối do máy chủ duy trì đạt đến con số này, yêu cầu kết nối TCP mới sẽ bị từ chối trực tiếp.
  - `max_connections_per_ip`: Số nguyên, giá trị mặc định là 0, có nghĩa là số lượng kết nối tối đa cho một IP client duy nhất, và 0 có nghĩa là không giới hạn.

- ### Tùy chọn Log

  Con của mục `app`, một đối tượng JSON, kiểm soát hành vi của output log như sau:

  ```json
  "log": {
        "log_path": "./",
        "logfile_base_name": "",
        "log_size_limit": 100000000,
        "log_level": "TRACE"
      },
  ```

  Trong đó:

  - `log_path`: Chuỗi, giá trị mặc định là một chuỗi rỗng, cho biết đường dẫn nơi tệp log được lưu trữ. Nếu nó là một chuỗi rỗng, tất cả các log sẽ được xuất ra đầu ra tiêu chuẩn.
  - `logfile_base_name`: Một chuỗi cho biết `basename` của tệp log. Giá trị mặc định là một chuỗi rỗng có nghĩa là basename sẽ là `drogon`.
  - `log_size_limit`: Một số nguyên, tính bằng byte. Giá trị mặc định là 100000000 (100MB). Khi kích thước của tệp log đạt đến giá trị này, tệp log sẽ được chuyển đổi (rotate).
  - `log_level`: Một chuỗi, giá trị mặc định là "DEBUG", cho biết mức thấp nhất của output log. Các giá trị tùy chọn là từ thấp đến cao: "TRACE", "DEBUG", "INFO", "WARN", trong đó mức TRACE chỉ có hiệu lực khi biên dịch ở chế độ DEBUG.

  > **Lưu ý: Log tệp của Drogon sử dụng cấu trúc output không chặn có thể đạt được output log hàng triệu dòng mỗi giây và có thể được sử dụng một cách tự tin.**

- ### Kiểm soát ứng dụng

  Chúng cũng là con của tùy chọn `app` và có hai tùy chọn, như sau:

  ```json
      "run_as_daemon": false,
      "relaunch_on_error": false,
  ```

  Trong đó:

  - `run_as_daemon`: Giá trị boolean, giá trị mặc định là `false`. Khi nó là `true`, ứng dụng sẽ là một tiến trình con của tiến trình số 1 dưới dạng daemon chạy trong nền của hệ thống.
  - `relaunch_on_error`: Giá trị boolean, giá trị mặc định là `false`. Khi nó là `true`, ứng dụng sẽ tự khởi chạy lại khi xảy ra lỗi.

- ### `use_sendfile`

  Tùy chọn con của tùy chọn `app`, boolean, cho biết có sử dụng system call `sendfile` của Linux hay không khi gửi tệp. Giá trị mặc định là `true`. Sử dụng `sendfile` có thể cải thiện hiệu quả gửi và giảm mức sử dụng bộ nhớ của các tệp lớn. Ví dụ:

  ```json
  "use_sendfile": true,
  ```

  > **Lưu ý: Ngay cả khi tùy chọn này là `true`, system call `sendfile` sẽ không được sử dụng, vì việc sử dụng `sendfile` cho các tệp nhỏ không nhất thiết phải hiệu quả về chi phí, và framework sẽ quyết định có nên áp dụng nó hay không theo chiến lược tối ưu hóa của riêng nó.**

- ### `use_gzip`

  Tùy chọn con `app`, boolean, giá trị mặc định là `true`, cho biết phần thân của phản hồi HTTP có sử dụng truyền nén hay không. Khi nó là `true`, nén được sử dụng trong các trường hợp sau:

  - Client hỗ trợ nén gzip;
  - Phần thân HTTP là kiểu text;
  - Độ dài của phần thân lớn hơn một giá trị nhất định;

  Ví dụ cấu hình như sau:

  ```json
  "use_gzip": true,
  ```

- ### `static_files_cache_time`

  Tùy chọn con `app`, giá trị số nguyên, tính bằng giây, cho biết thời gian lưu trữ (cache) của tệp tĩnh, tức là đối với yêu cầu lặp lại cho tệp trong thời gian này, framework sẽ trả về phản hồi trực tiếp từ bộ nhớ mà không cần đọc hệ thống tệp. Giá trị mặc định là 5 giây, 0 có nghĩa là luôn cache (chỉ đọc hệ thống tệp một lần, hãy sử dụng thận trọng), giá trị âm có nghĩa là không cache. Ví dụ:

  ```json
  "static_files_cache_time": 5,
  ```

- ### `simple_controllers_map`

  Tùy chọn con `app`, một mảng các đối tượng JSON, mỗi đối tượng đại diện cho một ánh xạ từ đường dẫn HTTP đến `HttpSimpleController`, cấu hình này chỉ là một tùy chọn thay thế, không nhất thiết phải được cấu hình ở đây, hãy xem [`HttpSimpleController`](VI-04-1-Controller-HttpSimpleController).
  Cấu hình cụ thể như sau:

  ```json
  "simple_controllers_map": [
        {
          "path": "/path/name",
          "controller": "controllerClassName",
          "http_methods": ["get","post"],
          "filters": ["FilterClassName"]
        }
      ],
  ```

  Trong đó:

  - `path`: Chuỗi, đường dẫn HTTP;
  - `controller`: Chuỗi, tên của `HttpSimpleController`;
  - `http_methods`: Một mảng các chuỗi đại diện cho các phương thức HTTP được hỗ trợ. Các yêu cầu nằm ngoài danh sách này sẽ bị lọc ra, trả về lỗi 405.
  - `filters`: Mảng chuỗi, danh sách các filter trên đường dẫn, xem [Middleware và Filter](VI-05-Middleware-and-Filter);

- ### Kiểm soát Thời gian chờ Kết nối Nhàn rỗi

  Tùy chọn con `app`, giá trị số nguyên, tính bằng giây, giá trị mặc định là 60. Khi kết nối vượt quá thời gian này mà không có bất kỳ thao tác đọc và ghi nào, kết nối sẽ bị ngắt kết nối cưỡng bức. Ví dụ:

  ```json
  "idle_connection_timeout": 60
  ```

- ### Tải View Động

  Các tùy chọn con của `app`, kiểm soát việc bật và đường dẫn của view động, có hai tùy chọn, như sau:

  ```json
  "load_dynamic_views": true,
  "dynamic_views_path": ["./views"],
  ```

  Trong đó:

  - `load_dynamic_views`: Giá trị boolean, giá trị mặc định là `false`. Khi nó là `true`, framework sẽ tìm kiếm các tệp view trong đường dẫn view và biên dịch động chúng thành các tệp `.so`, sau đó tải chúng vào ứng dụng. Khi bất kỳ tệp view nào thay đổi, nó cũng sẽ gây ra quá trình biên dịch và tải lại tự động;
  - `dynamic_views_path`: Một mảng các chuỗi, mỗi chuỗi đại diện cho đường dẫn tìm kiếm của view động. Nếu giá trị đường dẫn không bắt đầu bằng `/`, `./` hoặc `../`, và giá trị không phải là `.` hoặc `..`, thì đường dẫn này là đường dẫn tương đối của mục `document_root` trước đó, nếu không thì đó là một đường dẫn tuyệt đối hoặc một đường dẫn tương đối đến thư mục hiện tại.

  Xem [View](VI-06-View)

- ### Trường Tiêu đề Máy chủ

  Tùy chọn con của `app` cấu hình trường tiêu đề `Server` của tất cả các phản hồi được gửi bởi framework. Giá trị mặc định là một chuỗi rỗng. Khi tùy chọn này trống, framework sẽ tự động tạo một trường tiêu đề ở dạng `Server: drogon/chuỗi_phiên_bản`. Ví dụ:

  ```json
  "server_header_field": ""
  ```

- ### Yêu cầu Keep-Alive

  Tùy chọn `keepalive_requests` đặt số lượng yêu cầu tối đa có thể được phục vụ thông qua một kết nối keep-alive. Sau khi số lượng yêu cầu tối đa được thực hiện, kết nối sẽ bị đóng. Giá trị mặc định là 0 có nghĩa là không giới hạn. Ví dụ:

  ```json
  "keepalive_requests": 0
  ```

- ### Yêu cầu Pipelining

  `pipelining_requests` đặt số lượng yêu cầu chưa được xử lý tối đa có thể được lưu vào bộ nhớ đệm trong bộ đệm pipeline. Sau khi số lượng yêu cầu tối đa được thực hiện, kết nối sẽ bị đóng. Giá trị mặc định là 0 có nghĩa là không giới hạn. Để biết chi tiết về pipelining, vui lòng xem `rfc2616-8.1.1.2`. Ví dụ:

  ```json
  "pipelining_requests": 0
  ```


# Tiếp theo: [Lập trình hướng khía cạnh (AOP)](VI-12-AOP-Aspect-Oriented-Programming)




