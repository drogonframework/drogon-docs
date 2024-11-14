## Cơ sở dữ liệu - FastDbClient

Như tên gọi của nó, `FastDbClient` sẽ cung cấp hiệu suất cao hơn so với `DbClient` thông thường. Không giống như `DbClient` có vòng lặp sự kiện riêng, nó chia sẻ vòng lặp sự kiện với các luồng I/O mạng và luồng chính của ứng dụng web, điều này làm cho việc triển khai nội bộ của `FastDbClient` khả dụng ở chế độ không khóa (lock-free) và hiệu quả hơn.

Các thử nghiệm cho thấy `FastDbClient` có cải thiện hiệu suất từ 10% đến 20% so với `DbClient` trong điều kiện tải cực cao.

### Tạo và Lấy

`FastDbClient` phải được tạo tự động bởi framework với tệp cấu hình hoặc bằng cách gọi giao diện `app().createDbClient()`:

Tùy chọn con `is_fast` của tùy chọn `db_client` trong tệp cấu hình cho biết client có phải là `FastDbClient` hay không. Hoặc người dùng có thể tạo `FastDbClient` bằng cách gọi phương thức `app().createDbClient()` với tham số cuối cùng được đặt thành `true`.

Framework tạo một `FastDbClient` riêng biệt cho mỗi vòng lặp sự kiện của I/O và vòng lặp sự kiện chính, và mỗi `FastDbClient` quản lý một số kết nối cơ sở dữ liệu nội bộ. Số lượng vòng lặp sự kiện của I/O được điều khiển bởi tùy chọn `"threads_num"` của framework, thường được đặt thành số lõi CPU của máy chủ. Số lượng kết nối cơ sở dữ liệu cho mỗi vòng lặp sự kiện là giá trị của tùy chọn `"connection_number"` của client cơ sở dữ liệu. Vui lòng tham khảo [Tệp cấu hình](VI-10-Configuration-File#db_clients). Do đó, tổng số kết nối cơ sở dữ liệu do `FastDbClient` nắm giữ là `(threads_num + 1) * connection_number`.

Giao diện để lấy một `FastDbClient` tương tự như `DbClient` thông thường, như sau:

```c++
orm::DbClientPtr getFastDbClient(const std::string &name = "default");
/// Sử dụng drogon::app().getFastDbClient("tên_client") để lấy đối tượng FastDbClient.
```

Cần chỉ ra rằng do tính chất đặc biệt của `FastDbClient`, người dùng phải gọi giao diện trên trong luồng vòng lặp sự kiện của I/O hoặc luồng chính để lấy con trỏ thông minh chính xác. Trong các luồng khác, chỉ có thể lấy con trỏ null và không thể sử dụng được.

### Sử dụng

Việc sử dụng `FastDbClient` gần như giống hệt với `DbClient` thông thường, ngoại trừ các hạn chế sau:

- Cả việc lấy và sử dụng nó phải nằm trong luồng vòng lặp sự kiện của I/O của framework hoặc luồng chính. Nếu nó được sử dụng trong các luồng khác, sẽ có lỗi không thể đoán trước (vì điều kiện không khóa bị phá hủy). May mắn thay, hầu hết việc lập trình ứng dụng đều nằm trong luồng I/O, chẳng hạn như trong các hàm xử lý của các controller khác nhau, trong hàm filter của các filter. Dễ dàng biết rằng các hàm gọi lại khác nhau của giao diện `FastDbClient` cũng nằm trong luồng I/O hiện tại và có thể được sử dụng lồng nhau một cách an toàn.
- Không bao giờ sử dụng giao diện chặn của `FastDbClient`, vì giao diện này sẽ chặn luồng hiện tại, và luồng hiện tại cũng là luồng xử lý I/O cơ sở dữ liệu của đối tượng này, điều này sẽ gây ra chặn vĩnh viễn, và người dùng không có cơ hội để nhận được kết quả.
- Giao diện tạo giao dịch đồng bộ có khả năng bị chặn (khi tất cả các kết nối đều bận), vì vậy giao diện tạo giao dịch đồng bộ của `FastDbClient` trả về con trỏ null trực tiếp. Nếu bạn muốn sử dụng giao dịch trên `FastDbClient`, vui lòng sử dụng giao diện tạo giao dịch bất đồng bộ.
- Sau khi sử dụng `FastDbClient` để tạo một đối tượng ORM Mapper, bạn cũng nên chỉ sử dụng các giao diện bất đồng bộ, không chặn của đối tượng mapper.


# Tiếp theo: [Chế độ hàng loạt tự động](VI-08-5-Database-auto_batch)



