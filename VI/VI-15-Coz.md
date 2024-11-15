## Hồ sơ nhân quả với coz

Với coz, bạn có thể định hình hai điều:

* Thông lượng (throughput)
* Độ trễ (latency)

Nếu bạn muốn định hình thông lượng của ứng dụng, bạn nên bật tùy chọn CMake `COZ_PROFILING` và bao gồm thông tin debug trong tệp thực thi của bạn với chế độ release `Debug` hoặc `RelWithDebInfo` trong CMake. Làm như vậy sẽ include các điểm tiến trình coz khi phục vụ yêu cầu. Định hình độ trễ hiện không được hỗ trợ trong toàn bộ phạm vi ứng dụng, nhưng vẫn có thể được thực hiện trong mã người dùng.

Khi bạn hoàn tất việc biên dịch ứng dụng của mình với các điểm tiến trình được include. Bạn cần chạy tệp thực thi với trình định hình (profiler) coz, ví dụ: `coz run --- [đường dẫn đến tệp thực thi của bạn]`.

Cuối cùng, ứng dụng cần được kiểm tra tải (stress), để có kết quả tốt nhất, bạn cần kiểm tra tải tất cả các đường dẫn mã và chạy profile trong một khoảng thời gian thích hợp, 15 phút trở lên.

Profile cuối cùng sẽ là tệp `profile.coz` được tạo trong thư mục làm việc hiện tại. Để xem kết quả, hãy mở profile trong [trình xem](https://plasma-umass.org/coz/) chính thức, hoặc bạn có thể chạy bản sao cục bộ từ [kho git](https://github.com/plasma-umass/coz) chính thức.

Coz cũng hỗ trợ phạm vi các tệp nguồn được include cho profile với `--source-scope <mẫu>` hoặc `-s <mẫu>` trong số những thứ khác, điều đó sẽ chứng minh là hữu ích.

Để biết thêm thông tin, hãy xem:

* `coz run --help`
* [Kho Git](https://github.com/plasma-umass/coz)
* [Sách trắng Coz](https://arxiv.org/pdf/1608.03676v1.pdf)

# 15 [Nén Brotli](VI-15-Brotli)
