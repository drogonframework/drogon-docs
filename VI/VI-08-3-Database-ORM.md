## Cơ sở dữ liệu - ORM

### Mô hình

Sử dụng ORM của Drogon, trước tiên bạn cần tạo các lớp mô hình (model class). Chương trình dòng lệnh `drogon_ctl` của Drogon cung cấp khả năng tạo các lớp mô hình. Chương trình đọc thông tin bảng từ cơ sở dữ liệu do người dùng chỉ định và tự động tạo nhiều tệp nguồn cho các lớp mô hình dựa trên thông tin này. Khi người dùng sử dụng mô hình, vui lòng include tệp header tương ứng.

Rõ ràng, mỗi lớp Model tương ứng với một bảng cơ sở dữ liệu cụ thể, và một instance của lớp mô hình tương ứng một hàng bản ghi trong bảng.

Lệnh để tạo các lớp mô hình như sau:

```shell
drogon_ctl create model <đường_dẫn_mô_hình>
```

Tham số cuối cùng là đường dẫn để lưu trữ các lớp mô hình. Phải có một tệp cấu hình `model.json` trong đường dẫn để cấu hình các tham số kết nối của `drogon_ctl` đến cơ sở dữ liệu. Nó là một tệp ở định dạng JSON và hỗ trợ comment. Các ví dụ như sau:

```json
{
  "rdbms": "postgresql",
  "host": "127.0.0.1",
  "port": 5432,
  "dbname": "test",
  "user": "test",
  "passwd": "",
  "tables": [],
  "relationships": {
      "enabled": false,
      "items": []
  }
}
```

Các tham số được cấu hình giống như tệp cấu hình của ứng dụng. Vui lòng tham khảo [Tệp cấu hình](VI-10-Configuration-File#db_clients).

Tùy chọn cấu hình `tables` là duy nhất cho cấu hình mô hình. Nó là một mảng các chuỗi. Mỗi chuỗi đại diện cho tên của bảng sẽ được chuyển đổi thành lớp mô hình. Nếu tùy chọn này trống, tất cả các bảng sẽ được sử dụng để tạo các lớp mô hình.

Thư mục `models` và tệp `model.json` tương ứng đã được tạo trước trong thư mục dự án được tạo bằng lệnh `drogon_ctl create project`. Người dùng có thể chỉnh sửa tệp cấu hình và tạo các lớp mô hình bằng lệnh `drogon_ctl`.

### Giao diện lớp mô hình

Chủ yếu có hai loại giao diện mà người dùng trực tiếp sử dụng, giao diện getter và giao diện setter.

Có hai loại giao diện getter:

- Một giao diện có dạng như `getColumnName()` nhận con trỏ thông minh của trường. Giá trị trả về là một con trỏ thay vì một giá trị được sử dụng chủ yếu cho trường NULL. Người dùng có thể xác định xem trường có phải là trường NULL hay không bằng cách xác định xem con trỏ có trống hay không.
- Một giao diện có dạng như `getValueOfColumnName()`, đúng như tên gọi, là giá trị thu được. Vì lý do hiệu quả, giao diện trả về một tham chiếu không đổi. Nếu trường tương ứng là NULL, giao diện trả về giá trị mặc định do tham số hàm đưa ra.

Ngoài ra, kiểu khối nhị phân (blob, bytea) có một giao diện đặc biệt, ở dạng `getValueOfColumnNameAsString()`, tải dữ liệu nhị phân vào đối tượng `std::string` và trả về cho người dùng.

Giao diện setter được sử dụng để đặt giá trị của trường tương ứng, ở dạng `setColumnName()`, và kiểu tham số và kiểu trường tương ứng. Các trường được tạo tự động (chẳng hạn như khóa chính tự động tăng) không có giao diện setter.

Giao diện `toJson()` được sử dụng để chuyển đổi đối tượng mô hình thành đối tượng JSON. Kiểu khối nhị phân được mã hóa base64. Vui lòng tự mình thử nghiệm với nó.

Các thành viên tĩnh của lớp Model đại diện cho thông tin của bảng. Ví dụ, tên của mỗi trường có thể được lấy thông qua thành viên tĩnh `Cols`, rất tiện lợi để sử dụng trong trình soạn thảo hỗ trợ tự động gợi ý.

### Template lớp Mapper

Việc ánh xạ giữa đối tượng mô hình và bảng cơ sở dữ liệu được thực hiện bởi template lớp `Mapper`. Template lớp `Mapper` đóng gói các thao tác phổ biến như thêm, xóa và thay đổi, để người dùng có thể thực hiện các thao tác trên mà không cần viết câu lệnh SQL.

Việc xây dựng đối tượng `Mapper` rất đơn giản. Tham số template là kiểu của mô hình bạn muốn truy cập. Hàm tạo chỉ có một tham số, đó là con trỏ thông minh `DbClient` đã đề cập trước đó. Như đã đề cập trước đó, lớp `Transaction` là một lớp con của `DbClient`, vì vậy bạn cũng có thể tạo một đối tượng `Mapper` với một con trỏ thông minh đến một giao dịch, có nghĩa là ánh xạ `Mapper` cũng hỗ trợ giao dịch.

Giống như `DbClient`, `Mapper` cũng cung cấp các giao diện bất đồng bộ và đồng bộ. Giao diện đồng bộ bị chặn và có thể ném ra ngoại lệ. Đối tượng future được trả về bị chặn trong `get()` và có thể ném ra ngoại lệ. Giao diện bất đồng bộ bình thường không ném ra ngoại lệ, nhưng trả về kết quả thông qua hai callback (callback kết quả và callback ngoại lệ). Loại callback ngoại lệ giống như trong giao diện `DbClient`. Callback kết quả cũng được chia thành nhiều loại theo hàm giao diện. Danh sách như sau (T là tham số template, là kiểu của mô hình):

![](https://drogonframework.github.io/drogon-docs/images/mapper_method1_en.png)
![](https://drogonframework.github.io/drogon-docs/images/mapper_method2_en.png)
![](https://drogonframework.github.io/drogon-docs/images/mapper_method3_en.png)

> **Lưu ý: Khi sử dụng giao dịch, ngoại lệ không nhất thiết phải gây ra rollback. Giao dịch sẽ không được rollback trong các trường hợp sau: Khi giao diện `findByPrimaryKey` không tìm thấy một hàng đủ điều kiện, khi giao diện `findOne` tìm thấy ít hơn hoặc nhiều hơn một bản ghi, mapper sẽ ném ra ngoại lệ hoặc vào một callback ngoại lệ, kiểu ngoại lệ là `UnexpectedRows`. Nếu logic nghiệp vụ cần được rollback trong điều kiện này, vui lòng gọi rõ ràng giao diện `rollback()`.**

### Tiêu chí

Trong phần trước, nhiều giao diện yêu cầu các tham số đối tượng tiêu chí đầu vào. Đối tượng tiêu chí là một instance của lớp `Criteria`, cho biết một điều kiện nhất định, chẳng hạn như một trường lớn hơn, bằng, nhỏ hơn một giá trị nhất định, hoặc một điều kiện như `IS NULL`.

```c++
template <typename T>
Criteria(const std::string &colName, const CompareOperator &opera, T &&arg)
```

Hàm tạo của một đối tượng tiêu chí rất đơn giản. Nói chung, đối số đầu tiên là tên của trường, đối số thứ hai là giá trị enum đại diện cho loại so sánh, và đối số thứ ba là giá trị đang được so sánh. Nếu loại so sánh là `IsNull` hoặc `IsNotNull`, thì tham số thứ ba không bắt buộc.

Ví dụ:

```c++
Criteria("user_id", CompareOperator::EQ, 1);
```

Ví dụ trên cho thấy trường `user_id` bằng 1 là một điều kiện. Trong thực tế, chúng tôi thích viết như sau:

```c++
Criteria(Users::Cols::_user_id, CompareOperator::EQ, 1);
```

Điều này tương đương với cái trước, nhưng cách viết này có thể sử dụng tự động gợi ý của trình soạn thảo, hiệu quả hơn và ít bị lỗi hơn;

Lớp `Criteria` cũng hỗ trợ các điều kiện `WHERE` tùy chỉnh cùng với một hàm tạo tùy chỉnh.

```c++
template <typename... Arguments>
explicit Criteria(const CustomSql &sql, Arguments &&...args)
```

Đối số đầu tiên là một đối tượng `CustomSql` của câu lệnh SQL với trình giữ chỗ `$?`, trong khi lớp `CustomSql` chỉ là một wrapper của `std::string`. Đối số không xác định thứ hai là một parameter pack đại diện cho tham số ràng buộc, hoạt động giống như những tham số trong [`execSqlAsync`](VI-08-1-Database-DbClient.md#execSqlAsync).

Ví dụ:

```c++
Criteria(CustomSql("tags @> $?"), "cloud");
```

Lớp `CustomSql` cũng có một user-defined string literal liên quan, vì vậy chúng tôi khuyên bạn nên viết như sau thay thế:

```c++
Criteria("tags @> $?"_sql, "cloud");
```

Điều này tương đương với cái trước.

Các đối tượng `Criteria` hỗ trợ các toán tử `AND` và `OR`. Tổng của hai đối tượng tiêu chí tạo thành một đối tượng tiêu chí mới, giúp dễ dàng xây dựng các điều kiện lồng nhau. Ví dụ:

```c++
Mapper<Users> mp(dbClientPtr);
auto users = mp.findBy(
    (Criteria(Users::Cols::_user_name, CompareOperator::Like, "%Smith") &&
     Criteria(Users::Cols::_gender, CompareOperator::EQ, 0)) ||
    (Criteria(Users::Cols::_user_name, CompareOperator::Like, "%Johnson") &&
     Criteria(Users::Cols::_gender, CompareOperator::EQ, 1)));
```

Chương trình trên là để truy vấn tất cả những người đàn ông tên Smith hoặc những người phụ nữ tên Johnson từ bảng `users`.

### Giao diện chuỗi của Mapper

Một số ràng buộc SQL phổ biến, chẳng hạn như `LIMIT`, `OFFSET`, v.v., template lớp `Mapper` cũng cung cấp hỗ trợ, được cung cấp dưới dạng giao diện chuỗi, có nghĩa là người dùng có thể chuỗi nhiều ràng buộc để viết. Sau khi thực thi bất kỳ giao diện nào trong Mục 10.5.3, các ràng buộc này sẽ bị xóa, tức là chúng hợp lệ trong một thao tác:

```c++
Mapper<Users> mp(dbClientPtr);
auto users = mp.orderBy(Users::Cols::_join_time).limit(25).offset(0).findAll();
```

Chương trình này là để chọn danh sách người dùng từ bảng `users`, trả về trang đầu tiên gồm 25 hàng mỗi trang.

Về cơ bản, tên của giao diện chuỗi thể hiện chức năng của nó, vì vậy tôi sẽ không đi sâu vào chi tiết ở đây. Vui lòng tham khảo tệp tiêu đề `Mapper.h`.

### Chuyển đổi

Tùy chọn cấu hình `convert` là duy nhất cho cấu hình mô hình. Nó thêm một lớp chuyển đổi trước hoặc sau khi một giá trị được đọc từ hoặc ghi vào cơ sở dữ liệu. Đối tượng bao gồm một key boolean `enabled` để sử dụng chức năng này hay không. Mảng các đối tượng `items` bao gồm các key sau:

- `table`: Tên của bảng chứa cột
- `column`: Tên của cột
- `method`: Đối tượng
  - `after_db_read`: Chuỗi, tên của phương thức được gọi sau khi đọc từ cơ sở dữ liệu, chữ ký: `void([const] std::shared_ptr<type> [&])`
  - `before_db_write`: Chuỗi, tên của phương thức được gọi trước khi ghi vào cơ sở dữ liệu, chữ ký: `void([const] std::shared_ptr<type> [&])`
- `includes`: Mảng chuỗi, tên của các tệp include được bao quanh bởi `"` hoặc `<,>`

### Mối quan hệ

Mối quan hệ giữa các bảng cơ sở dữ liệu có thể được cấu hình thông qua tùy chọn `relationships` trong tệp cấu hình `model.json`. Chúng tôi sử dụng cấu hình thủ công thay vì tự động phát hiện khóa ngoại của bảng vì dự án thực tế có thể không sử dụng khóa ngoại.

Nếu tùy chọn `enable` là `true`, các lớp mô hình được tạo sẽ thêm các giao diện tương ứng theo cấu hình `relationships`.

Có ba loại mối quan hệ, `has one`, `has many` và `many to many`.

- #### `has one`

  `has one` đại diện cho mối quan hệ một-một. Một bản ghi trong bảng gốc có thể được liên kết với một bản ghi trong bảng đích, và ngược lại. Ví dụ: bảng `products` và bảng `skus` có mối quan hệ một-một, chúng ta có thể định nghĩa như sau:

  ```json
  {
    "type": "has one",
    "original_table_name": "products",
    "original_table_alias": "product",
    "original_key": "id",
    "target_table_name": "skus",
    "target_table_alias": "SKU",
    "target_key": "product_id",
    "enable_reverse": true
  }
  ```

  Trong đó:

  - `"type"`: Cho biết mối quan hệ này là một-một;
  - `"original_table_name"`: Tên của bảng gốc (phương thức tương ứng sẽ được thêm vào mô hình tương ứng với bảng này);
  - `"original_table_alias"`: Bí danh (tên trong phương thức, vì mối quan hệ một-một là số ít, nên đặt nó thành `product`), nếu tùy chọn này trống, tên bảng được sử dụng để tạo tên phương thức;
  - `"original_key"`: Khóa liên kết của bảng gốc;
  - `"target_table_name"`: Tên của bảng đích;
  - `"target_table_alias"`: Bí danh của bảng đích, nếu tùy chọn này trống, tên bảng được sử dụng để tạo tên phương thức;
  - `"target_key"`: Khóa liên kết của bảng đích;
  - `"enable_reverse"`: Cho biết có tự động tạo mối quan hệ ngược lại hay không, tức là thêm một phương thức để lấy các bản ghi của bảng gốc trong lớp mô hình tương ứng với bảng đích.

  Theo cài đặt này, trong lớp mô hình tương ứng với bảng `products`, phương thức sau sẽ được thêm vào:

  ```c++
  /// Giao diện mối quan hệ
  void getSKU(const DbClientPtr &clientPtr,
              const std::function<void(Skus)> &rcb,
              const ExceptionCallback &ecb) const;
  ```

  Đây là một giao diện bất đồng bộ trả về đối tượng `SKU` được liên kết với sản phẩm hiện tại trong callback.

  Đồng thời, vì tùy chọn `enable_reverse` được đặt thành `true`, phương thức sau sẽ được thêm vào lớp mô hình tương ứng với bảng `skus`:

  ```c++
  /// Giao diện mối quan hệ
  void getProduct(const DbClientPtr &clientPtr,
                  const std::function<void(Products)> &rcb,
                  const ExceptionCallback &ecb) const;
  ```

- #### `has many`

  `has many` đại diện cho mối quan hệ một-nhiều. Trong mối quan hệ như vậy, bảng đại diện cho `many` thường có một trường được liên kết với khóa chính của bảng khác. Ví dụ: `products` và `reviews` thường có mối quan hệ một-nhiều, chúng ta có thể định nghĩa như sau:

  ```json
  {
    "type": "has many",
    "original_table_name": "products",
    "original_table_alias": "product",
    "original_key": "id",
    "target_table_name": "reviews",
    "target_table_alias": "",
    "target_key": "product_id",
    "enable_reverse": true
  }
  ```

  Ý nghĩa của mỗi cấu hình ở trên giống như ví dụ trước, vì vậy tôi sẽ không lặp lại ở đây, vì có nhiều `review` cho một `product`, nên không cần tạo bí danh cho `reviews`. Theo cài đặt này, sau khi chạy `drogon_ctl create model`, giao diện sau sẽ được thêm vào mô hình tương ứng với bảng `products`:

  ```c++
  void getReviews(const DbClientPtr &clientPtr,
                  const std::function<void(std::vector<Reviews>)> &rcb,
                  const ExceptionCallback &ecb) const;
  ```

  Trong mô hình tương ứng với bảng `reviews`, giao diện sau sẽ được thêm vào:

  ```c++
  void getProduct(const DbClientPtr &clientPtr,
                  const std::function<void(Products)> &rcb,
                  const ExceptionCallback &ecb) const;
  ```

- #### `many to many`

  Như tên của nó, `many to many` đại diện cho mối quan hệ nhiều-nhiều. Thông thường, mối quan hệ nhiều-nhiều yêu cầu một bảng pivot. Mỗi bản ghi trong bảng pivot tương ứng với một bản ghi trong bảng gốc và một bản ghi khác trong bảng đích. Ví dụ: bảng `products` và bảng `carts` có mối quan hệ nhiều-nhiều, có thể được định nghĩa như sau:

  ```json
  {
    "type": "many to many",
    "original_table_name": "products",
    "original_table_alias": "",
    "original_key": "id",
    "pivot_table": {
      "table_name": "carts_products",
      "original_key": "product_id",
      "target_key": "cart_id"
    },
    "target_table_name": "carts",
    "target_table_alias": "",
    "target_key": "id",
    "enable_reverse": true
  }
  ```

  Đối với bảng pivot, có một cấu hình `pivot_table` bổ sung. Các tùy chọn bên trong dễ hiểu và được bỏ qua ở đây.

  Mô hình `products` được tạo theo cấu hình này sẽ thêm phương thức sau:

  ```c++
  void getCarts(const DbClientPtr &clientPtr,
                const std::function<void(std::vector<std::pair<Carts, CartsProducts>>)> &rcb,
                const ExceptionCallback &ecb) const;
  ```

  Lớp mô hình của bảng `carts` sẽ thêm phương thức sau:

  ```c++
  void getProducts(const DbClientPtr &clientPtr,
                  const std::function<void(std::vector<std::pair<Products, CartsProducts>>)> &rcb,
                  const ExceptionCallback &ecb) const;
  ```

### RESTful API Controller

`drogon_ctl` cũng có thể tạo RESTful API Controller cho mỗi mô hình (hoặc bảng) trong khi tạo mô hình, để người dùng có thể tạo API có thể thêm, xóa, sửa đổi và tìm kiếm bảng mà không cần viết code. Các API này hỗ trợ nhiều chức năng như truy vấn theo khóa chính, truy vấn theo điều kiện, sắp xếp theo các trường cụ thể, trả về các trường được chỉ định, và gán bí danh cho mỗi trường để ẩn cấu trúc bảng. Nó được điều khiển bởi tùy chọn `restful_api_controllers` trong `model.json`. Các tùy chọn này có comment tương ứng trong tệp JSON.

Cần lưu ý rằng controller của mỗi bảng được thiết kế để bao gồm một lớp base và một lớp con (subclass). Trong đó, lớp base và bảng có liên quan chặt chẽ với nhau, và lớp con được sử dụng để triển khai logic nghiệp vụ đặc biệt hoặc sửa đổi định dạng giao diện. Ưu điểm của thiết kế này là khi cấu trúc bảng thay đổi, người dùng chỉ có thể cập nhật lớp base mà không cần ghi đè lớp con (bằng cách đặt tùy chọn `generate_base_only` thành `true`).


# Tiếp theo: [FastDbClient](VI-08-4-Database-FastDbClient)



