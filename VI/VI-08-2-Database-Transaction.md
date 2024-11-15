## Cơ sở dữ liệu - Giao dịch

> **Giao dịch** là một tính năng quan trọng của cơ sở dữ liệu quan hệ, và Drogon cung cấp hỗ trợ giao dịch với lớp `Transaction`.

Các đối tượng của lớp `Transaction` được tạo bởi `DbClient`, và nhiều thao tác liên quan đến giao dịch được thực hiện tự động:

- Khi bắt đầu tạo đối tượng `Transaction`, câu lệnh `BEGIN` được tự động thực thi để *bắt đầu* giao dịch;
- Khi đối tượng `Transaction` bị hủy, câu lệnh `COMMIT` được tự động thực thi để *kết thúc* giao dịch;
- Nếu có một ngoại lệ khiến giao dịch thất bại, câu lệnh `ROLLBACK` được tự động thực thi để *khôi phục* giao dịch;
- Nếu giao dịch đã được khôi phục, thì câu lệnh SQL sẽ trả về một ngoại lệ (ném ra ngoại lệ hoặc thực hiện gọi lại ngoại lệ);

### Tạo giao dịch

Phương thức tạo giao dịch được cung cấp bởi `DbClient` như sau:

```c++
std::shared_ptr<Transaction> newTransaction(const std::function<void(bool)> &commitCallback = std::function<void(bool)>())
```

Giao diện này rất đơn giản, nó trả về một con trỏ thông minh đến một đối tượng `Transaction`. Rõ ràng, khi con trỏ thông minh mất tất cả các chủ sở hữu và hủy đối tượng giao dịch, giao dịch sẽ kết thúc. Tham số `commitCallback` được sử dụng để trả về việc commit giao dịch có thành công hay không. Cần lưu ý rằng callback này chỉ được sử dụng để cho biết lệnh `COMMIT` có thành công hay không. Nếu giao dịch được tự động hoặc thủ công rollback trong quá trình thực thi, thì `callback` sẽ không được thực thi. Nói chung, lệnh `COMMIT` sẽ thành công, tham số kiểu bool của callback này là `true`. Chỉ một số trường hợp đặc biệt, chẳng hạn như ngắt kết nối trong quá trình commit, sẽ khiến `commitCallback` thông báo cho người dùng rằng việc commit không thành công, tại thời điểm này, trạng thái của giao dịch trên máy chủ không chắc chắn, người dùng cần xử lý tình huống này đặc biệt. Tất nhiên, xem xét rằng tình huống này hiếm khi xảy ra, với các dịch vụ không quan trọng, người dùng có thể chọn bỏ qua sự kiện này bằng cách bỏ qua tham số `commitCallback` khi tạo giao dịch (callback rỗng mặc định sẽ được chuyển đến phương thức `newTransaction`).

Giao dịch phải độc quyền kết nối cơ sở dữ liệu. Do đó, trong quá trình tạo giao dịch, `DbClient` cần chọn một kết nối nhàn rỗi từ connection pool của chính nó và giao nó cho quản lý đối tượng giao dịch. Điều này có một vấn đề. Nếu tất cả các kết nối trong `DbClient` đang thực thi SQL hoặc các giao dịch khác, giao diện sẽ bị chặn cho đến khi có kết nối nhàn rỗi.

Framework cũng cung cấp một giao diện bất đồng bộ để tạo giao dịch, như sau:

```c++
void newTransactionAsync(const std::function<void(const std::shared_ptr<Transaction> &)> &callback);
```

Giao diện này trả về đối tượng giao dịch thông qua hàm gọi lại, không chặn luồng hiện tại, và đảm bảo tính concurrency cao của ứng dụng. Người dùng có thể sử dụng nó hoặc phiên bản đồng bộ tùy theo tình huống thực tế.

### Giao diện giao dịch

Giao diện `Transaction` gần như giống hệt với `DbClient`, ngoại trừ hai điểm khác biệt sau:

- `Transaction` cung cấp một giao diện `rollback()` cho phép người dùng rollback giao dịch trong mọi trường hợp. Đôi khi, giao dịch đã được tự động rollback, và sau đó gọi giao diện `rollback()` không có tác động tiêu cực, vì vậy việc sử dụng rõ ràng giao diện `rollback()` là một chiến lược tốt để ít nhất đảm bảo rằng nó không được commit không chính xác.
- Người dùng không thể gọi giao diện `newTransaction()` của giao dịch, điều này rất dễ hiểu. Mặc dù cơ sở dữ liệu có khái niệm về giao dịch con (sub-transaction), nhưng framework hiện không hỗ trợ nó.

Trên thực tế, `Transaction` được thiết kế như một lớp con của `DbClient`, nhằm duy trì tính nhất quán của các giao diện này, đồng thời, nó cũng tạo điều kiện thuận lợi cho việc sử dụng [ORM](VI-08-3-Database-ORM).

Framework hiện không cung cấp giao diện để kiểm soát mức độ cô lập giao dịch (transaction isolation level), tức là mức độ cô lập là mức mặc định của dịch vụ cơ sở dữ liệu hiện tại.

### Vòng đời giao dịch

Con trỏ thông minh của đối tượng giao dịch được giữ bởi người dùng. Khi nó có SQL chưa được thực thi, framework sẽ giữ nó, vì vậy đừng lo lắng về việc đối tượng giao dịch bị hủy khi vẫn còn SQL chưa được thực thi. Ngoài ra, con trỏ thông minh đối tượng giao dịch thường bị bắt và sử dụng trong callback kết quả của một trong các giao diện của nó. Đây là cách sử dụng bình thường, đừng lo lắng rằng tham chiếu vòng (circular reference) sẽ khiến đối tượng giao dịch không bao giờ bị hủy, vì framework sẽ giúp người dùng tự động phá vỡ tham chiếu vòng.

### Một ví dụ

Đối với ví dụ đơn giản nhất, giả sử có một bảng task mà người dùng chọn một task chưa được xử lý và thay đổi nó thành trạng thái đang được xử lý. Để ngăn chặn race condition đồng thời, chúng ta sử dụng lớp `Transaction`, chương trình như sau:

```c++
{
    auto transPtr = clientPtr->newTransaction();
    transPtr->execSqlAsync("select * from tasks where status=$1 for update order by time",
                            "none",
                            [=](const Result &r) {
                                if (r.size() > 0)
                                {
                                    std::cout << "Got a task!" << std::endl;
                                    *transPtr << "update tasks set status=$1 where task_id=$2"
                                              << "handling"
                                              << r[0]["task_id"].as<int64_t>() >> [](const Result &r) {
                                                  std::cout << "Updated!";
                                                  // ... xử lý task ...
                                              } >> [](const DrogonDbException &e) {
                                                  std::cerr << "err:" << e.base().what() << std::endl;
                                              };
                                }
                                else
                                {
                                    std::cout << "No new tasks found!" << std::endl;
                                }
                            },
                            [](const DrogonDbException &e) {
                                std::cerr << "err:" << e.base().what() << std::endl;
                            });
}
```

Trong trường hợp này, `select for update` được sử dụng để tránh sửa đổi đồng thời. Câu lệnh update được hoàn thành trong callback kết quả của câu lệnh select. Dấu ngoặc nhọn ngoài cùng được sử dụng để giới hạn phạm vi của `transPtr` để nó có thể bị hủy kịp thời sau khi thực thi SQL để kết thúc giao dịch.


# Tiếp theo: [ORM](VI-08-3-Database-ORM)



