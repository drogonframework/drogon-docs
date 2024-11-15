## Cơ sở dữ liệu - Chế độ Hàng loạt Tự động

Chế độ hàng loạt tự động chỉ có hiệu lực đối với thư viện client của PostgreSQL phiên bản 14+ và sẽ bị bỏ qua trong các trường hợp khác. Trước khi nói về xử lý hàng loạt tự động, trước tiên chúng ta hãy hiểu chế độ pipeline.

### Chế độ Pipeline

Kể từ PostgreSQL 14, thư viện client của nó cung cấp một giao diện chế độ pipeline. Trong chế độ pipeline, các yêu cầu SQL mới có thể được gửi trực tiếp đến máy chủ mà không cần đợi kết quả của yêu cầu trước đó trả về (điều này phù hợp với khái niệm pipeline HTTP). Để biết chi tiết, vui lòng tham khảo [Chế độ Pipeline](https://www.postgresql.org/docs/current/libpq-pipeline-mode.html). Chế độ này rất hữu ích cho hiệu suất, cho phép ít kết nối cơ sở dữ liệu hơn để hỗ trợ các yêu cầu đồng thời lớn hơn.
Drogon bắt đầu hỗ trợ chế độ này sau phiên bản 1.7.6, Drogon sẽ tự động kiểm tra xem libpq có hỗ trợ chế độ pipeline hay không, nếu có, tất cả các yêu cầu được gửi thông qua `DbClient` của Drogon đều ở chế độ pipeline.

### Chế độ Hàng loạt Tự động

Theo mặc định, đối với các client không giao dịch, Drogon tạo điểm đồng bộ hóa (synchronization point) cho mỗi yêu cầu SQL, tức là mỗi câu lệnh SQL riêng lẻ là một giao dịch ngầm, đảm bảo rằng các yêu cầu SQL độc lập với nhau, điều này làm cho chế độ pipeline và chế độ non-pipeline xuất hiện cho người dùng là hoàn toàn tương đương.

Tuy nhiên, việc tạo điểm đồng bộ hóa cho mỗi câu lệnh SQL cũng có một chi phí hiệu suất nhất định, vì vậy Drogon cung cấp một chế độ hàng loạt tự động trong đó thay vì tạo điểm đồng bộ hóa sau mỗi câu lệnh SQL, một điểm đồng bộ hóa được tạo sau một số câu lệnh SQL. Các quy tắc để tạo điểm đồng bộ hóa trong cùng một kết nối như sau:

- Một điểm đồng bộ hóa phải được tạo sau SQL cuối cùng trong vòng lặp `EventLoop`;
- Tạo điểm đồng bộ hóa sau một SQL ghi vào cơ sở dữ liệu;
- Tạo điểm đồng bộ hóa sau một SQL lớn;
- Tạo điểm đồng bộ hóa khi số lượng câu lệnh SQL liên tiếp sau điểm đồng bộ hóa cuối cùng đạt đến giới hạn trên;

Lưu ý rằng các câu lệnh SQL giữa hai điểm đồng bộ hóa trong cùng một liên kết thuộc về cùng một giao dịch ngầm. Drogon không cung cấp giao diện rõ ràng để mở và đóng các điểm đồng bộ hóa, vì vậy các SQL này có thể không liên quan đến nhau về mặt logic, nhưng do chúng nằm trong cùng một giao dịch nên chúng sẽ ảnh hưởng lẫn nhau, do đó, chế độ này không hoàn toàn an toàn, nó có các vấn đề sau:

- Một SQL không thành công sẽ khiến các câu lệnh SQL trước đó của nó bị rollback sau điểm đồng bộ hóa cuối cùng, nhưng người dùng sẽ không nhận được bất kỳ thông báo nào, vì không có giao dịch rõ ràng nào được sử dụng;
- Một SQL không thành công sẽ khiến tất cả các câu lệnh SQL tiếp theo trước điểm đồng bộ hóa tiếp theo trả về lỗi;
- Việc đánh giá ghi cơ sở dữ liệu dựa trên so khớp từ khóa đơn giản (INSERT, UPDATE, v.v.), không bao gồm tất cả các trường hợp, chẳng hạn như trường hợp gọi stored procedure thông qua các câu lệnh SELECT, vì vậy mặc dù Drogon cố gắng giảm thiểu các tác động tiêu cực của chế độ hàng loạt tự động, nhưng nó không hoàn toàn an toàn;

Do đó, chế độ hàng loạt tự động rất hữu ích để cải thiện hiệu suất, nhưng nó không an toàn. Người dùng sẽ quyết định sử dụng nó trong trường hợp nào. Ví dụ: để thực thi các câu lệnh SQL chỉ đọc thuần túy thông qua `DbClient` ở chế độ hàng loạt tự động.

> **Lưu ý** Ngay cả SQL chỉ đọc đôi khi cũng có thể gây ra lỗi giao dịch (chẳng hạn như SELECT timeout), vì vậy SQL tiếp theo của nó cũng sẽ bị ảnh hưởng bởi nó và thất bại (điều này có thể không được người dùng chấp nhận, vì chúng có thể không liên quan đến nhau trong logic ứng dụng), vì vậy, nói một cách chính xác, các trường hợp ứng dụng của chế độ hàng loạt tự động nên được giới hạn trong các truy vấn dữ liệu chỉ đọc và không quan trọng. Khuyến nghị người dùng tạo một `DbClient` hàng loạt tự động riêng biệt cho các câu lệnh SQL như vậy. Tất nhiên, dễ dàng nhận ra rằng các đối tượng giao dịch được tạo bởi `DbClient` ở chế độ hàng loạt tự động là an toàn để sử dụng.

### Bật Chế độ Hàng loạt Tự động

Khi sử dụng giao diện `newPgClient` để tạo một client, hãy đặt tham số thứ ba thành `true` để bật chế độ hàng loạt tự động;
Khi sử dụng tệp cấu hình để tạo một client, hãy đặt tùy chọn `auto_batch` thành `true` để bật chế độ hàng loạt tự động cho client;


# Tiếp theo: [Tham chiếu Yêu cầu](VI-09-0-References-request)



