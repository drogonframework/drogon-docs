## View

### Giới thiệu về View

Mặc dù công nghệ render front-end đang phổ biến, dịch vụ ứng dụng back-end chỉ cần trả về dữ liệu tương ứng cho front-end. Tuy nhiên, một framework web tốt nên cung cấp công nghệ render back-end, để chương trình máy chủ có thể tạo động các trang HTML. View có thể giúp người dùng tạo các trang này. Như tên gọi của nó, nó chỉ chịu trách nhiệm thực hiện công việc liên quan đến việc trình bày, và logic nghiệp vụ phức tạp nên được giao cho controller.

Các ứng dụng web sớm nhất nhúng HTML vào mã chương trình để đạt được mục đích tạo động các trang HTML, nhưng điều này không hiệu quả, không trực quan, v.v. Vì vậy, có những ngôn ngữ như JSP, ngược lại, nhúng mã chương trình vào trang HTML. Drogon tất nhiên là giải pháp thứ hai. Tuy nhiên, rõ ràng là vì C++ được biên dịch và thực thi, chúng ta cần chuyển đổi trang được nhúng trong mã C++ thành chương trình nguồn C++ để biên dịch vào ứng dụng. Do đó, Drogon định nghĩa ngôn ngữ mô tả CSP (C++ Server Pages) chuyên biệt của riêng mình, sử dụng công cụ dòng lệnh `drogon_ctl` để chuyển đổi các tệp CSP thành các tệp nguồn C++ để biên dịch.

### CSP của Drogon

Giải pháp CSP của Drogon rất đơn giản, chúng tôi sử dụng các ký hiệu đánh dấu đặc biệt để nhúng mã C++ vào trang HTML. Trong đó:

- Nội dung giữa các thẻ `<%inc` và `%>` được coi là một phần của tệp header cần được include. Chỉ có câu lệnh `#include` có thể được viết ở đây, chẳng hạn như `<%inc #include "xx.h" %>`, nhưng nhiều tệp header phổ biến được Drogon tự động include. Về cơ bản, người dùng không sử dụng thẻ này;
- Mọi thứ giữa các thẻ `<%c++` và `%>` được coi là mã C++, chẳng hạn như `<%c++ std::string name="drogon"; %>`;
- Mã C++ thường được chuyển đến tệp nguồn đích nguyên vẹn, ngoại trừ hai thẻ đặc biệt sau:
  - `@@` đại diện cho biến dữ liệu được truyền bởi controller, từ đó bạn có thể lấy nội dung bạn muốn hiển thị;
  - `$$` đại diện cho một đối tượng stream đại diện cho nội dung của trang, và nội dung cần hiển thị có thể được hiển thị trên trang bởi toán tử `<<`;
- Nội dung được đặt giữa các thẻ `[[` và `]]` được coi là tên biến. View sẽ sử dụng tên làm khóa để tìm biến tương ứng từ dữ liệu được truyền từ controller và xuất nó ra trang. Khoảng trắng trước và sau tên biến sẽ bị bỏ qua. Các cặp `[[` và `]]` nên nằm trên cùng một dòng. Và vì lý do hiệu suất, chỉ hỗ trợ ba kiểu dữ liệu chuỗi (`const char *`, `std::string` và `const std::string`), các kiểu dữ liệu khác nên được xuất theo cách đã đề cập ở trên (bằng `$$`);
- Nội dung được đặt giữa các thẻ `{%` và `%}` được coi là tên của một biến hoặc biểu thức của chương trình C++ (không phải là từ khóa của dữ liệu được truyền bởi controller), và view sẽ xuất ra nội dung của biến hoặc giá trị của biểu thức đến trang. Dễ dàng biết rằng `{% val.xx %}` tương đương với `<%c++ $$ << val.xx; %>`, nhưng cách viết trước đơn giản và trực quan hơn. Tương tự, không viết hai thẻ trong các dòng riêng biệt;
- Nội dung được đặt giữa các thẻ `<%view` và `%>` được coi là tên của sub-view. Framework sẽ tìm sub-view tương ứng và điền nội dung của nó vào vị trí của thẻ; khoảng trắng trước và sau tên view sẽ bị bỏ qua. Không viết `<%view` và `%>` trong các dòng riêng biệt. Có thể sử dụng nhiều cấp độ lồng nhau, nhưng không lồng nhau vòng lặp;
- Nội dung giữa các thẻ `<%layout` và `%>` được coi là tên của layout. Framework sẽ tìm layout tương ứng và điền nội dung của view này vào một vị trí trong layout (trong layout, trình giữ chỗ `[[]]` đánh dấu vị trí này); khoảng trắng trước và sau tên layout sẽ bị bỏ qua, và `<%layout` và `%>` không nên được viết trong các dòng riêng biệt. Bạn có thể sử dụng nhiều cấp độ lồng nhau, nhưng không lồng nhau vòng lặp. Một tệp template chỉ có thể kế thừa từ một layout cơ sở, không hỗ trợ đa kế thừa từ các layout khác nhau.

### Cách sử dụng view

Phản hồi HTTP của ứng dụng Drogon được tạo bởi handler của controller, do đó, phản hồi được render bởi view cũng được tạo bởi handler, được tạo bằng cách gọi giao diện sau:

```c++
static HttpResponsePtr newHttpViewResponse(const std::string &viewName,
                                           const HttpViewData &data);
```

Giao diện này là một phương thức tĩnh của lớp `HttpResponse`, có hai tham số:

- **viewName**: Tên của view, tên của tệp CSP đến (**có thể bỏ qua phần mở rộng**);
- **data**: Handler của controller truyền dữ liệu đến view. Kiểu là `HttpViewData`. Đây là một map đặc biệt. Bạn có thể lưu và truy xuất bất kỳ kiểu đối tượng nào. Để biết chi tiết, vui lòng tham khảo mô tả [HttpViewData API](API-HttpViewData)

Như bạn có thể thấy, controller không cần include tệp header của view. Controller và view được tách rời tốt; kết nối duy nhất của chúng là biến dữ liệu.

### Một ví dụ đơn giản

Bây giờ chúng ta hãy tạo một view hiển thị các tham số của yêu cầu HTTP được gửi bởi trình duyệt trong trang HTML được trả về.

Lần này, chúng tôi trực tiếp định nghĩa handler bằng giao diện `HttpAppFramework`. Trong tệp chính, hãy thêm mã sau trước khi gọi phương thức `run()`:

```c++
drogon::HttpAppFramework::instance()
        .registerHandler("/list_para",
                        [=](const HttpRequestPtr &req,
                            std::function<void (const HttpResponsePtr &)> &&callback)
                        {
                            auto para = req->getParameters();
                            HttpViewData data;
                            data.insert("title", "ListParameters");
                            data.insert("parameters", para);
                            auto resp = HttpResponse::newHttpViewResponse("ListParameters.csp", data);
                            callback(resp);
                        });
```

Đoạn mã trên đăng ký một handler biểu thức lambda trên đường dẫn `/list_para`, chuyển các tham số được yêu cầu đến hiển thị view.
Sau đó, hãy chuyển đến thư mục `views` và tạo một tệp view `ListParameters.csp` với nội dung sau:

```html
<!DOCTYPE html>
<html>
<%c++
    auto para = @@.get<std::unordered_map<std::string, std::string, utils::internal::SafeStringHash>>("parameters");
%>
<head>
    <meta charset="UTF-8">
    <title>[[ title ]]</title>
</head>
<body>
    <%c++ if(para.size() > 0) { %>
    <h1>Parameters</h1>
    <table border="1">
      <tr>
        <th>name</th>
        <th>value</th>
      </tr>
      <%c++ for(auto iter : para){ %>
      <tr>
        <td>{% iter.first %}</td>
        <td><%c++ $$ << iter.second; %></td>
      </tr>
      <%c++ } %>
    </table>
    <%c++ } else { %>
    <h1>No parameter</h1>
    <%c++ } %>
</body>
</html>
```

Chúng ta có thể sử dụng công cụ dòng lệnh `drogon_ctl` để chuyển đổi `ListParameters.csp` thành các tệp nguồn C++ như sau:

```shell
drogon_ctl create view ListParameters.csp
```

Sau khi thao tác kết thúc, hai tệp nguồn, `ListParameters.h` và `ListParameters.cc`, sẽ xuất hiện trong thư mục hiện tại, có thể được sử dụng để biên dịch vào ứng dụng web;

Biên dịch lại toàn bộ dự án bằng CMake, chạy chương trình mục tiêu webapp, bạn có thể kiểm tra hiệu ứng trong trình duyệt, nhập `http://localhost/list_para?p1=a&p2=b&p3=c` vào thanh địa chỉ, bạn có thể thấy trang sau:

![view page](images/viewdemo.png)

Trang HTML được render bởi back-end được thêm vào một cách đơn giản.

### Xử lý tự động các tệp CSP

**Lưu ý: Nếu dự án của bạn được tạo bằng lệnh `drogon_ctl`, công việc được mô tả trong phần này sẽ được `drogon_ctl` thực hiện tự động.**

Rõ ràng, việc chạy thủ công lệnh `drogon_ctl` mỗi khi bạn sửa đổi tệp CSP là quá bất tiện. Chúng ta có thể đưa quá trình xử lý của `drogon_ctl` vào tệp `CMakeLists.txt`. Vẫn sử dụng ví dụ trước làm ví dụ. Giả sử chúng ta đặt tất cả các tệp CSP trong thư mục `views`, `CMakeLists.txt` có thể được thêm vào như sau:

```cmake
FILE(GLOB SCP_LIST ${CMAKE_CURRENT_SOURCE_DIR}/views/*.csp)
foreach(cspFile ${SCP_LIST})
  message(STATUS "cspFile:" ${cspFile})
  execute_process(COMMAND basename ARGS "-s .csp ${cspFile}" OUTPUT_VARIABLE classname)
  message(STATUS "view classname:" ${classname})
  add_custom_command(
    OUTPUT ${classname}.h ${classname}.cc
    COMMAND drogon_ctl ARGS create view ${cspFile}
    DEPENDS ${cspFile}
    VERBATIM)
  set(VIEWSRC ${VIEWSRC} ${classname}.cc)
endforeach()
```

Sau đó thêm một bộ sưu tập tệp nguồn mới `${VIEWSRC}` vào câu lệnh `add_executable` như sau:

```cmake
add_executable(webapp ${SRC_DIR} ${VIEWSRC})
```

### Biên dịch và tải động các view

Drogon cung cấp một cách để biên dịch và tải động các tệp CSP trong thời gian chạy ứng dụng, sử dụng giao diện sau:

```c++
void enableDynamicViewsLoading(const std::vector<std::string> &libPaths);
```

Giao diện là một phương thức thành viên của `HttpAppFramework`, và tham số là một mảng các chuỗi đại diện cho danh sách các thư mục chứa tệp CSP của view. Sau khi gọi giao diện này, Drogon sẽ tự động tìm kiếm các tệp CSP trong các thư mục này. Sau khi phát hiện các tệp CSP mới hoặc đã sửa đổi, các tệp nguồn sẽ được tạo tự động, biên dịch thành các tệp thư viện động và tải vào ứng dụng. Quá trình ứng dụng không cần phải khởi động lại. Người dùng có thể tự mình thử nghiệm và quan sát những thay đổi trên trang do sửa đổi tệp CSP.

Rõ ràng, chức năng này phụ thuộc vào môi trường phát triển. Nếu cả Drogon và webapp được biên dịch trên máy chủ này, thì sẽ không có vấn đề gì khi tải động trang CSP.

> **Lưu ý: View động không nên được biên dịch tĩnh vào ứng dụng. Điều này có nghĩa là nếu view được biên dịch tĩnh, thì nó không thể được cập nhật thông qua tải view động. Bạn có thể tạo một thư mục bên ngoài thư mục biên dịch và di chuyển view vào đó trong quá trình phát triển.**

> **Lưu ý: Tính năng này được sử dụng tốt nhất để điều chỉnh trang HTML trong giai đoạn phát triển. Trong môi trường production, bạn nên biên dịch tệp CSP trực tiếp vào tệp mục tiêu. Điều này chủ yếu là vì lý do bảo mật và ổn định.**

> **Lưu ý: Nếu xảy ra lỗi `symbol not found` khi tải view động, vui lòng sử dụng `cmake .. -DCMAKE_ENABLE_EXPORTS=on` để cấu hình dự án của bạn hoặc bỏ ghi chú dòng cuối cùng (`set_property(TARGET ${PROJECT_NAME} PROPERTY ENABLE_EXPORTS ON)`) trong tệp `CMakeLists.txt` của dự án, sau đó build lại dự án**

# Tiếp theo: [Session](VI-07-Session)

