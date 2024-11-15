## Cơ sở dữ liệu - DbClient

### Khởi tạo đối tượng DbClient

Có hai cách để khởi tạo một đối tượng `DbClient`. Một là thông qua phương thức tĩnh của lớp `DbClient`. Bạn có thể xem định nghĩa trong tệp tiêu đề `DbClient.h`, như sau:

```c++
#if USE_POSTGRESQL
    static std::shared_ptr<DbClient> newPgClient(const std::string &connInfo, const size_t connNum);
#endif
#if USE_MYSQL
    static std::shared_ptr<DbClient> newMysqlClient(const std::string &connInfo, const size_t connNum);
#endif
```

Sử dụng giao diện trên để lấy con trỏ thông minh của đối tượng triển khai `DbClient`. Tham số `connInfo` là một chuỗi kết nối. Đặt một loạt các tham số kết nối ở dạng `key=value`. Để biết chi tiết, vui lòng tham khảo các nhận xét trong tệp tiêu đề. Tham số `connNum` là số lượng kết nối cơ sở dữ liệu của `DbClient`, điều này có tác động quan trọng đến concurrency. Vui lòng đặt nó theo tình huống thực tế.

Đối tượng thu được bằng phương pháp trên, người dùng phải tìm cách duy trì nó, chẳng hạn như đặt nó vào một số container toàn cục. **Tạo một đối tượng tạm thời và sau đó giải phóng nó sau khi sử dụng là một giải pháp rất không nên** vì những lý do sau:

- Điều này sẽ lãng phí thời gian tạo kết nối và ngắt kết nối, làm tăng độ trễ của hệ thống;
- Giao diện cũng là một giao diện không chặn. Điều đó có nghĩa là, khi người dùng nhận được đối tượng `DbClient`, kết nối do nó quản lý vẫn chưa được thiết lập. Framework không (cố ý) cung cấp giao diện gọi lại cho việc thiết lập kết nối thành công. Bạn vẫn phải sleep trước khi bắt đầu truy vấn?? Điều này trái với ý định ban đầu của framework bất đồng bộ.

Do đó, các đối tượng `DbClient` nên được xây dựng khi bắt đầu chương trình và được giữ và sử dụng trong suốt thời gian tồn tại. Rõ ràng, công việc này có thể được thực hiện hoàn toàn bởi framework. Vì vậy, framework Drogon cung cấp phương thức build thứ hai, được build bằng tệp cấu hình hoặc phương thức `createDbClient()`. Đối với phương pháp cấu hình của tệp cấu hình, hãy xem [db_clients](VI-10-Configuration-file#db_clients).

Khi cần, con trỏ thông minh `DbClient` được lấy thông qua giao diện của framework. Giao diện như sau:

```c++
orm::DbClientPtr getDbClient(const std::string &name = "default");
```

Tham số `name` là giá trị của tùy chọn cấu hình `name` trong tệp cấu hình để phân biệt nhiều đối tượng `DbClient` khác nhau của cùng một ứng dụng. Các kết nối được quản lý bởi `DbClient` luôn được kết nối lại, vì vậy người dùng không cần quan tâm đến trạng thái kết nối. Chúng gần như luôn được kết nối. **Lưu ý**: Phương thức này không thể được gọi trước khi chạy `app().run()`, nếu không người dùng sẽ nhận được một `shared_ptr` rỗng.

### Giao diện thực thi

`DbClient` cung cấp một số giao diện khác nhau cho người dùng, như được liệt kê bên dưới:

```c++
/// Phương thức bất đồng bộ
template <
        typename FUNCTION1,
        typename FUNCTION2,
        typename... Arguments>
void execSqlAsync(const std::string &sql,
                  FUNCTION1 &&rCallback,
                  FUNCTION2 &&exceptCallback,
                  Arguments &&... args) noexcept;

/// Phương thức bất đồng bộ bằng 'future'
template <typename... Arguments>
std::future<const Result> execSqlAsyncFuture(const std::string &sql,
                                             Arguments &&... args) noexcept;

/// Phương thức đồng bộ
template <typename... Arguments>
const Result execSqlSync(const std::string &sql,
                         Arguments &&... args) noexcept(false);

/// Phương thức kiểu stream
internal::SqlBinder operator<<(const std::string &sql);
```

Vì số lượng và kiểu tham số ràng buộc không thể được xác định trước, nên các phương thức này là các template hàm.

Các thuộc tính của các phương thức này được hiển thị trong bảng sau:

| Phương thức                                        | Đồng bộ/Bất đồng bộ | Chặn/Không chặn                         | Ngoại lệ                                                         |
| :--------------------------------------------- | :----------------------- | :------------------------------------------- | :---------------------------------------------------------------- |
| `void execSqlAsync`                             | Bất đồng bộ            | Không chặn                                 | Sẽ không ném ra ngoại lệ                                       |
| `std::future<const Result> execSqlAsyncFuture` | Bất đồng bộ            | Chặn khi gọi phương thức `get` của future | Có thể ném ra ngoại lệ khi gọi phương thức `get` của future  |
| `const Result execSqlSync`                      | Đồng bộ               | Chặn                                     | Có thể ném ra ngoại lệ                                            |
| `internal::SqlBinder operator<<`                | Bất đồng bộ            | Mặc định không chặn                       | Sẽ không ném ra ngoại lệ                                       |

Bạn có thể nhầm lẫn về sự kết hợp của bất đồng bộ và chặn. Nói chung, phương thức đồng bộ hóa liên quan đến IO mạng là chặn, và phương thức bất đồng bộ là không chặn. Tuy nhiên, phương thức bất đồng bộ cũng có thể hoạt động ở chế độ chặn, nghĩa là phương thức này sẽ chặn cho đến khi hàm gọi lại kết thúc thực thi. Khi phương thức bất đồng bộ của `DbClient` hoạt động ở chế độ chặn, hàm gọi lại sẽ được thực thi trong luồng của người gọi, và sau đó phương thức sẽ trả về.

Nếu ứng dụng của bạn liên quan đến các tình huống concurrency cao, vui lòng sử dụng các phương thức bất đồng bộ, không chặn. Nếu ở trong trường hợp concurrency thấp (chẳng hạn như trang quản lý thiết bị mạng), bạn có thể chọn phương thức đồng bộ để thuận tiện và trực quan.

- #### `execSqlAsync`

  ```c++
  template <typename FUNCTION1,
          typename FUNCTION2,
          typename... Arguments>
  void execSqlAsync(const std::string &sql,
                  FUNCTION1 &&rCallback,
                  FUNCTION2 &&exceptCallback,
                  Arguments &&... args) noexcept;
  ```

  Đây là giao diện bất đồng bộ được sử dụng phổ biến nhất, hoạt động ở chế độ không chặn;

  Tham số `sql` là một chuỗi câu lệnh SQL. Nếu có trình giữ chỗ cho các tham số ràng buộc, hãy sử dụng các quy tắc trình giữ chỗ của cơ sở dữ liệu tương ứng. Ví dụ: trình giữ chỗ PostgreSQL là `$1`, `$2` ..., trong khi trình giữ chỗ MySQL là `?` mà không có bất kỳ số nào.

  Tham số không xác định `args` đại diện cho tham số ràng buộc, có thể là 0 hoặc nhiều hơn. Số lượng tham số giống với số lượng trình giữ chỗ trong câu lệnh SQL. Các kiểu có thể là sau đây:

  - Kiểu số nguyên: có thể là số nguyên có độ dài từ khác nhau, và phải khớp với kiểu trường cơ sở dữ liệu;
  - Kiểu dấu phẩy động: có thể là `float` hoặc `double`, phải khớp với kiểu trường cơ sở dữ liệu;
  - Kiểu chuỗi: có thể là `std::string` hoặc `const char[]`, tương ứng với kiểu chuỗi của cơ sở dữ liệu hoặc các kiểu khác có thể được biểu thị bằng chuỗi;
  - Kiểu ngày: kiểu `trantor::Date`, tương ứng với kiểu ngày, datetime, timestamp của cơ sở dữ liệu.
  - Kiểu nhị phân: kiểu `std::vector<char>`, tương ứng với kiểu byte của PostgreSQL hoặc kiểu blob của MySQL;

  Các tham số này có thể là lvalue hoặc rvalue, có thể là biến hoặc hằng số nguyên văn, và người dùng có thể tự do sử dụng chúng.

  Các tham số `rCallback` và `exceptCallback` lần lượt đại diện cho hàm gọi lại kết quả và hàm gọi lại ngoại lệ, có định nghĩa cố định, như sau:

  - Hàm gọi lại kết quả: kiểu gọi là `void(const Result &)`, các đối tượng có thể gọi khác nhau phù hợp với kiểu gọi này, `std::function`, lambda, v.v. có thể được truyền dưới dạng tham số;
  - Hàm gọi lại ngoại lệ: kiểu gọi là `void(const DrogonDbException &)`, có thể truyền các đối tượng có thể gọi khác nhau phù hợp với kiểu gọi này;

  Sau khi thực hiện SQL thành công, kết quả thực thi được gói gọn bởi lớp `Result` và được chuyển cho người dùng thông qua hàm gọi lại kết quả; nếu có bất kỳ ngoại lệ nào trong quá trình thực thi SQL, hàm gọi lại ngoại lệ sẽ được thực thi, và người dùng có thể lấy thông tin ngoại lệ từ đối tượng `DrogonDbException`.

  Chúng ta hãy cho một ví dụ:

  ```c++
  auto clientPtr = drogon::app().getDbClient();
  clientPtr->execSqlAsync("select * from users where org_name=$1",
                              [](const drogon::orm::Result &result) {
                                  std::cout << result.size() << " rows selected!" << std::endl;
                                  int i = 0;
                                  for (auto row : result)
                                  {
                                      std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
                                  }
                              },
                              [](const DrogonDbException &e) {
                                  std::cerr << "error:" << e.base().what() << std::endl;
                              },
                              "default");
  ```

  Từ ví dụ, chúng ta có thể thấy rằng đối tượng `Result` là một container tương thích tiêu chuẩn `std`, hỗ trợ iterator, bạn có thể lấy đối tượng của mỗi hàng thông qua vòng lặp phạm vi. Các giao diện khác nhau của các đối tượng `Result`, `Row` và `Field`, vui lòng tham khảo mã nguồn.

  Lớp `DrogonDbException` là lớp cơ sở cho tất cả các ngoại lệ cơ sở dữ liệu. Vui lòng tham khảo các nhận xét trong mã nguồn.

- #### `execSqlAsyncFuture`

  ```c++
  template <typename... Arguments>
  std::future<const Result> execSqlAsyncFuture(const std::string &sql,
                                              Arguments &&... args) noexcept;
  ```

  Giao diện future bất đồng bộ bỏ qua hai tham số gọi lại của giao diện trước đó. Gọi giao diện này sẽ ngay lập tức trả về một đối tượng future. Người dùng phải gọi phương thức `get()` của đối tượng future để nhận kết quả được trả về. Ngoại lệ được lấy thông qua cơ chế `try/catch`. Nếu phương thức `get()` không nằm trong `try/catch`, và không có `try/catch` trong toàn bộ call stack, chương trình sẽ thoát khi xảy ra ngoại lệ thực thi SQL.

  Ví dụ:

  ```c++
  auto f = clientPtr->execSqlAsyncFuture("select * from users where org_name=$1",
                                      "default");
  try
  {
      auto result = f.get(); // Chặn cho đến khi chúng tôi nhận được kết quả hoặc bắt được ngoại lệ;
      std::cout << result.size() << " rows selected!" << std::endl;
      int i = 0;
      for (auto row : result)
      {
          std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
      }
  }
  catch (const DrogonDbException &e)
  {
      std::cerr << "error:" << e.base().what() << std::endl;
  }
  ```

- #### `execSqlSync`

  ```c++
  template <typename... Arguments>
  const Result execSqlSync(const std::string &sql,
                          Arguments &&... args) noexcept(false);
  ```

  Giao diện đồng bộ là đơn giản và trực quan nhất, các tham số đầu vào là chuỗi SQL và các tham số ràng buộc, trả về một đối tượng `Result`, cuộc gọi sẽ chặn luồng hiện tại và ném ra ngoại lệ khi xảy ra lỗi, vì vậy cũng hãy chú ý bắt ngoại lệ bằng `try/catch`.

  Ví dụ:

  ```c++
  try
  {
      auto result = clientPtr->execSqlSync("update users set user_name=$1 where user_id=$2",
                                          "test",
                                          1); // Chặn cho đến khi chúng tôi nhận được kết quả hoặc bắt được ngoại lệ;
      std::cout << result.affectedRows() << " rows updated!" << std::endl;
  }
  catch (const DrogonDbException &e)
  {
      std::cerr << "error:" << e.base().what() << std::endl;
  }
  ```

- #### `operator<<`

  ```c++
  internal::SqlBinder operator<<(const std::string &sql);
  ```

  Giao diện stream là đặc biệt. Nó nhập câu lệnh SQL và các tham số lần lượt thông qua toán tử `<<`, và chỉ định hàm gọi lại kết quả và hàm gọi lại ngoại lệ thông qua toán tử `>>`. Ví dụ, ví dụ trước về việc chọn, sử dụng giao diện stream như sau:

  ```c++
  *clientPtr << "select * from users where org_name=$1"
              << "default" >> [](const drogon::orm::Result &result) {
                  std::cout << result.size() << " rows selected!" << std::endl;
                  int i = 0;
                  for (auto row : result)
                  {
                      std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
                  }
              } >> [](const DrogonDbException &e) {
                  std::cerr << "error:" << e.base().what() << std::endl;
              };
  ```

  Cách sử dụng này hoàn toàn tương đương với giao diện bất đồng bộ, không chặn đầu tiên, và giao diện nào được sử dụng phụ thuộc vào thói quen sử dụng của người dùng. Nếu bạn muốn nó hoạt động ở chế độ chặn, bạn có thể sử dụng `<<` để nhập một tham số `Mode::Blocking`, không được mô tả ở đây.

  Ngoài ra, giao diện stream có một cách sử dụng đặc biệt. Sử dụng một hàm gọi lại kết quả đặc biệt, framework có thể chuyển kết quả cho người dùng theo từng hàng. Kiểu gọi của hàm gọi lại này như sau:

  ```c++
  void(bool, Arguments...);
  ```

  Khi tham số bool đầu tiên là `true`, có nghĩa là kết quả là một hàng rỗng, tức là tất cả các kết quả đã được trả về, đây là hàm gọi lại cuối cùng;
  Đằng sau là một loạt các tham số, tương ứng với giá trị của mỗi cột của một hàng bản ghi, framework sẽ thực hiện chuyển đổi kiểu, tất nhiên, người dùng cũng nên chú ý đến kiểu khớp. Các kiểu này có thể là tham chiếu lvalue kiểu `const`, hoặc tham chiếu rvalue, và tất nhiên là các kiểu giá trị.

  Hãy viết lại ví dụ trước với hàm gọi lại này:

  ```c++
  int i = 0;
  *clientPtr << "select user_name, user_id from users where org_name=$1"
              << "default" >> [&i](bool isNull, const std::string &name, int64_t id) {
                  if (!isNull)
                      std::cout << i++ << ": user name is " << name << ", user id is " << id << std::endl;
                  else
                      std::cout << i << " rows selected!" << std::endl;
              } >> [](const DrogonDbException &e) {
                  std::cerr << "error:" << e.base().what() << std::endl;
              };
  ```

  Có thể thấy rằng các giá trị của trường `user_name` và `user_id` trong câu lệnh select lần lượt được gán cho các biến `name` và `id` trong hàm gọi lại, và người dùng không cần phải tự xử lý các chuyển đổi này, điều này rõ ràng mang lại một sự thuận tiện nhất định, và người dùng có thể sử dụng nó một cách linh hoạt.

> **Lưu ý: Điều quan trọng cần nhấn mạnh là trong lập trình bất đồng bộ, người dùng phải chú ý đến biến `i` trong ví dụ trên. Người dùng phải đảm bảo rằng biến `i` hợp lệ khi xảy ra gọi lại vì nó bị bắt bởi tham chiếu. Hàm gọi lại sẽ được gọi trong một luồng khác, và ngữ cảnh hiện tại có thể đã thất bại khi xảy ra gọi lại. Lập trình viên thường sử dụng con trỏ thông minh để giữ các biến được tạo tạm thời và sau đó chụp chúng thông qua các hàm gọi lại để đảm bảo tính hợp lệ của các biến.**

### Tóm tắt

Mỗi đối tượng `DbClient` có một hoặc nhiều luồng `EventLoop` riêng để kiểm soát IO kết nối cơ sở dữ liệu, chấp nhận yêu cầu thông qua giao diện đồng bộ hoặc bất đồng bộ, và trả về kết quả thông qua hàm gọi lại.

Giao diện chặn của `DbClient` chỉ chặn luồng người gọi, miễn là luồng người gọi không phải là luồng `EventLoop`, nó sẽ không ảnh hưởng đến hoạt động bình thường của luồng `EventLoop`. Khi hàm gọi lại được gọi, chương trình bên trong hàm gọi lại được chạy trên luồng `EventLoop`. Do đó, không thực hiện bất kỳ thao tác chặn nào trong hàm gọi lại, nếu không nó sẽ ảnh hưởng đến hiệu suất concurrency của việc đọc và ghi cơ sở dữ liệu. Bất kỳ ai quen thuộc với lập trình I/O không chặn đều nên hiểu ràng buộc này.


# Tiếp theo: [Giao dịch](VI-08-2-Database-Transaction)


