## Kiểm thử Hiệu năng

Là một framework ứng dụng C++ HTTP, hiệu suất nên là một trong những trọng tâm cần chú ý. Phần này giới thiệu các thử nghiệm đơn giản và thành tích của Drogon.

### Môi trường thử nghiệm

* Hệ thống là Linux CentOS 7.4;
* Thiết bị là máy chủ Dell, CPU là hai CPU Intel(R) Xeon(R) E5-2670 @ 2.60GHz, 16 lõi và 32 luồng;
* Bộ nhớ 64GB;
* GCC phiên bản 7.3.0;

### Kế hoạch và kết quả kiểm tra

Chúng tôi chỉ muốn kiểm tra hiệu suất của framework Drogon, vì vậy chúng tôi muốn đơn giản hóa quá trình xử lý của controller càng nhiều càng tốt. Chúng tôi chỉ thực hiện `HttpSimpleController` và đăng ký nó trên đường dẫn `/benchmark`. Controller trả về `<p>Hello, world!</p>` cho bất kỳ yêu cầu nào. Đặt số lượng luồng Drogon thành 16. Hàm xử lý như sau và bạn có thể tìm thấy mã nguồn tại đường dẫn `drogon/examples/benchmark`:

```c++
void BenchmarkCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback)
{
    //viết logic ứng dụng của bạn ở đây
    auto resp = HttpResponse::newHttpResponse();
    resp->setBody("<p>Hello, world!</p>");
    resp->setExpiredTime(0);
    callback(resp);
}
```

Để so sánh, tôi đã chọn Nginx để so sánh kiểm tra, đã viết một `hello_world_module` và biên dịch nó với mã nguồn Nginx. Tham số `worker_processes` của Nginx được đặt thành 16.

Công cụ kiểm tra là `httpress`, một công cụ kiểm tra tải (stress testing) HTTP hiệu suất tốt.

Chúng tôi điều chỉnh các tham số của `httpress`, kiểm tra mỗi bộ tham số năm lần và ghi lại các giá trị lớn nhất và nhỏ nhất của số lượng yêu cầu được xử lý mỗi giây. Kết quả kiểm tra như sau:

| Dòng lệnh                                 | Mô tả                                                    | Drogon(kQPS) | Nginx(kQPS) |
| :------------------------------------------- | :------------------------------------------------------------- | :----------: | :---------: |
| `httpress -c 100 -n 1000000 -t 16 -k -q URL`   | 100 kết nối, 1 triệu yêu cầu, 16 luồng, Keep-Alive     |   561/552    |   330/329   |
| `httpress -c 100 -n 1000000 -t 12 -q URL`      | 100 kết nối, 1 triệu yêu cầu, 12 luồng, không Keep-Alive |   140/135    |    31/49    |
| `httpress -c 1000 -n 1000000 -t 16 -k -q URL`  | 1000 kết nối, 1 triệu yêu cầu, 16 luồng, Keep-Alive    |   573/565    |   333/327   |
| `httpress -c 1000 -n 1000000 -t 16 -q URL`     | 1000 kết nối, 1 triệu yêu cầu, 16 luồng, không Keep-Alive |   155/143    |    52/50    |
| `httpress -c 10000 -n 4000000 -t 16 -k -q URL` | 10000 kết nối, 4 triệu yêu cầu, 16 luồng, Keep-Alive   |   512/508    |   316/314   |
| `httpress -c 10000 -n 1000000 -t 16 -q URL`    | 10000 kết nối, 1 triệu yêu cầu, 16 luồng, không Keep-Alive|   143/141    |    43/40    |

Như bạn có thể thấy, sử dụng tùy chọn Keep-Alive ở phía client, Drogon có thể xử lý hơn 500,000 yêu cầu mỗi giây trong trường hợp một kết nối có thể gửi nhiều yêu cầu. Điểm số này khá tốt. Trong trường hợp mỗi yêu cầu khởi tạo một kết nối, thời gian CPU sẽ được dành cho việc thiết lập và ngắt kết nối TCP và thông lượng sẽ giảm xuống còn 140,000 yêu cầu mỗi giây, điều này là hợp lý.

Dễ dàng nhận thấy Drogon có lợi thế rõ ràng hơn Nginx trong bài kiểm tra trên. Nếu ai đó thực hiện một bài kiểm tra chính xác hơn, vui lòng sửa cho tôi.

Hình ảnh bên dưới là ảnh chụp màn hình của một bài kiểm tra:

![Kết quả kiểm tra](images/benchmark.png)

# 14 [Hồ sơ nhân quả với coz](VI-14-Coz)


