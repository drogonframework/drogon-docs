## Cơ sở dữ liệu - Tổng quan

### Tổng quan

**Drogon** có công cụ đọc/ghi cơ sở dữ liệu tích hợp sẵn. Hoạt động kết nối cơ sở dữ liệu dựa trên công nghệ I/O không chặn. Do đó, ứng dụng hoạt động ở chế độ bất đồng bộ, không chặn hiệu quả từ lớp dưới cùng đến lớp trên cùng, đảm bảo hiệu suất đồng thời cao của Drogon. Hiện tại, Drogon hỗ trợ cơ sở dữ liệu PostgreSQL và MySQL. Nếu bạn muốn sử dụng cơ sở dữ liệu, trước tiên phải cài đặt môi trường phát triển của cơ sở dữ liệu tương ứng. Drogon sẽ tự động phát hiện các tệp tiêu đề và tệp thư viện của các thư viện này và biên dịch các phần tương ứng. Để chuẩn bị môi trường phát triển cơ sở dữ liệu, hãy xem [Môi trường phát triển](VI-02-Installation#Database-Environment).

**Drogon** hỗ trợ cơ sở dữ liệu SQLite3 để hỗ trợ các ứng dụng nhẹ. Giao diện bất đồng bộ được triển khai thông qua nhóm luồng, giống như giao diện của các cơ sở dữ liệu đã nói ở trên.

### DbClient

Lớp cơ bản của cơ sở dữ liệu Drogon là `DbClient` (đây là một lớp abstract, kiểu cụ thể phụ thuộc vào giao diện tạo ra nó). Không giống như giao diện cơ sở dữ liệu chung, một đối tượng `DbClient` không đại diện cho một kết nối cơ sở dữ liệu duy nhất. Nó có thể chứa một hoặc nhiều kết nối cơ sở dữ liệu, vì vậy bạn có thể coi nó như một **đối tượng connection pool**.

`DbClient` cung cấp cả giao diện đồng bộ và bất đồng bộ. Giao diện bất đồng bộ cũng hỗ trợ cả chế độ chặn và không chặn. Tất nhiên, để hợp tác với framework bất đồng bộ Drogon, bạn nên sử dụng giao diện bất đồng bộ với chế độ không chặn.

Thông thường, khi một giao diện bất đồng bộ được gọi, `DbClient` sẽ chọn ngẫu nhiên một trong các kết nối nhàn rỗi mà nó quản lý để thực hiện các thao tác truy vấn liên quan. Khi kết quả trả về, `DbClient` sẽ xử lý dữ liệu và trả về cho người gọi thông qua đối tượng hàm gọi lại; Nếu không có kết nối nhàn rỗi, nội dung thực thi sẽ được lưu vào bộ đệm. Khi một kết nối đã thực thi yêu cầu SQL của riêng nó, lệnh đang chờ xử lý sẽ được lấy từ bộ đệm để thực thi.

Để biết chi tiết về `DbClient`, hãy xem [DbClient](VI-08-1-Database-DbClient).

### Giao dịch

Đối tượng giao dịch có thể được tạo bởi `DbClient` để hỗ trợ các thao tác giao dịch. Ngoài giao diện `rollback()` bổ sung, đối tượng giao dịch về cơ bản giống với `DbClient`. Lớp giao dịch là `Transaction`. Để biết chi tiết về lớp `Transaction`, hãy xem [Transaction](VI-08-2-Database-Transaction).

### ORM

Drogon cũng cung cấp hỗ trợ cho **ORM**. Người dùng có thể sử dụng lệnh `drogon_ctl` để đọc các bảng trong cơ sở dữ liệu và tạo mã nguồn mô hình tương ứng. Sau đó, thực hiện các thao tác cơ sở dữ liệu của các mô hình này thông qua template lớp `Mapper<MODEL>`. Mapper cung cấp các giao diện đơn giản và thuận tiện cho các thao tác cơ sở dữ liệu tiêu chuẩn, cho phép người dùng thực hiện việc thêm, xóa và thay đổi bảng mà không cần viết câu lệnh SQL. Đối với **ORM**, vui lòng tham khảo [ORM](VI-08-3-Database-ORM)


# Tiếp theo: [DbClient](VI-08-1-Database-DbClient)




