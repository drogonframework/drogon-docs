Phân tích tệp là trích xuất tệp (hoặc các tệp) từ yêu cầu POST dữ liệu nhiều phần thành một đối tượng `HttpFile` thông qua `MultiPartParser`, đây là một số thông tin về:

## Đối tượng `MultiPartParser`

  #### Tóm tắt:

    Đây là đối tượng mà bạn sẽ sử dụng để trích xuất và lưu trữ tạm thời các tệp yêu cầu.

- ### `parse(const std::shared_ptr<HttpRequest> &req)`

  #### Tóm tắt:
    Nhận đối tượng yêu cầu làm tham số, đọc và xác định các tệp (nếu có) và chuyển nó đến biến `MultiPartParser`.

  #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);
    }
    ```

- ### `getFiles()`

  #### Tóm tắt
  Phải được gọi sau `parse()`, trả về các tệp của yêu cầu ở định dạng `std::vector<HttpFile>`.

  #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);

        // Kiểm tra xem có tệp không
        if (fileParser.getFiles().empty()) {
            // Không tìm thấy tệp
        }

        size_t num_of_files = fileParser.getFiles().size();
    }
    ```

- ### `getParameters()`

  #### Tóm tắt
  Phải được gọi sau `parse()`, trả về danh sách các phần khác từ biểu mẫu MultiPartData.

  #### Trả về:
  `std::unordered_map<std::string, std::string>` (khóa, giá trị)

  #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);

        if (!fileParser.getFiles().empty()) {
            for (const auto &header : fileParser.getParameters()){
                header.first // Khóa biểu mẫu
                header.second // Giá trị từ khóa biểu mẫu
            }
        }
    }
    ```

- ### `getParameter<typename T>(const std::string &key)`

  #### Tóm tắt
  Phải được gọi sau `parse()`, phiên bản riêng lẻ của `getParameters()`.

  #### Đầu vào:
  Kiểu của đối tượng mong đợi (sẽ được chuyển đổi tự động), giá trị khóa của tham số.

  #### Trả về:
  Nội dung của tham số tương ứng với khóa ở định dạng được thông báo, nếu nó không tồn tại, sẽ trả về giá trị mặc định của Đối tượng T.

  #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);

        std::string email = fileParser.getParameter<std::string>("email_form");

        // Kiểu chuỗi mặc định là ""
        if (email.empty()) {
            // không tìm thấy email_form
        }
    }
    ```

## Đối tượng `HttpFile`

  #### Tóm tắt:
  Đây là đối tượng đại diện cho một tệp trong bộ nhớ, được sử dụng bởi `MultiPartParser`.

- ### `getFileName()`

  #### Tóm tắt
  Tên tự giải thích, nhận tên gốc của tệp đã nhận được.

  #### Trả về:
  `std::string`.

    #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);

        std::string filename = fileParser.getFiles()[0].getFileName();
    }
    ```

- ### `fileLength()`

  #### Tóm tắt
  Lấy kích thước tệp.

  #### Trả về:
  `size_t`

    #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);

        size_t filesize = fileParser.getFiles()[0].fileLength();
    }
    ```

- ### `getFileExtension()`

  #### Tóm tắt
  Lấy phần mở rộng tệp.

  #### Trả về:
  `std::string`

    #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);

        std::string file_extension = fileParser.getFiles()[0].getFileExtension();
    }
    ```

- ### `getMd5()`

  #### Tóm tắt
  Lấy băm MD5 của tệp để kiểm tra tính toàn vẹn.

  #### Trả về:
  `std::string`

- ### `save()`

  #### Tóm tắt
  Lưu tệp vào hệ thống tệp. Thư mục lưu tệp là `UploadPath` được cấu hình trong `config.json` (hoặc tương đương). Đường dẫn đầy đủ là 
  ```c++ 
  drogon::app().getUploadPath() + "/" + this->getFileName()
  ```
  Hoặc để đơn giản hóa, nó được lưu dưới dạng: `UploadPath/filename`


- ### `save(const std::string &path)`

  #### Tóm tắt
  Phiên bản nếu tham số không bị bỏ qua, sử dụng tham số `path` thay vì `UploadPath`.
  
  #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);

        // Đường dẫn tương đối
        fileParser.getFiles()[0].save("./"); // Ghi tệp vào cùng thư mục trên máy chủ, với tên gốc

        // Đường dẫn tuyệt đối
        fileParser.getFiles()[0].save("/home/user/downloads/"); // Ghi tệp vào thư mục được chỉ định, với tên gốc
    }
    ```

- ### `saveAs(const std::string &path)`

  #### Tóm tắt
  Ghi tệp vào tham số đường dẫn với tên mới (bỏ qua tên gốc).
  
  #### Ví dụ:
  ```c++
    #include "mycontroller.h"

    using namespace drogon;

    void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
        // Chỉ yêu cầu POST (Biểu mẫu tệp)

        MultiPartParser fileParser;
        fileParser.parse(req);

        // Đường dẫn tương đối
        fileParser.getFiles()[0].saveAs("./image.png"); // Cùng đường dẫn của máy chủ
        /* Chỉ là ví dụ, Đừng làm điều này, bạn sẽ ghi đè định dạng tệp mà không kiểm tra xem nó có thực sự là png hay không */

        // Đường dẫn tuyệt đối
        fileParser.getFiles()[0].saveAs("/home/user/downloads/anyname." + fileParser.getFiles()[0].getFileExtension());
    }
    ```

# Tiếp theo: [Plugin](VI-10-Plugins) 
