## Lệnh drogon_ctl

Sau khi framework **Drogon** được biên dịch và cài đặt, bạn nên tạo dự án đầu tiên của mình bằng chương trình dòng lệnh `drogon_ctl` được cài đặt cùng với framework. Để thuận tiện, có lệnh rút gọn `dg_ctl`. Người dùng có thể lựa chọn theo sở thích của mình.

Chức năng chính của chương trình là giúp người dùng dễ dàng tạo các tệp dự án Drogon khác nhau. Sử dụng lệnh `dg_ctl help` để xem các chức năng mà nó hỗ trợ, như sau:

```console
$ dg_ctl help
usage: drogon_ctl <command> [<args>]
commands list:
create                  tạo một số tệp nguồn (Sử dụng 'drogon_ctl help create' để biết thêm thông tin)
help                    hiển thị thông báo này
version                 hiển thị phiên bản của công cụ này
press                   Thực hiện kiểm tra tải (Sử dụng 'drogon_ctl help press' để biết thêm thông tin)
```

### Lệnh con version

Lệnh con `version` được sử dụng để in phiên bản Drogon hiện đang được cài đặt trên hệ thống, như sau:

```console
$ dg_ctl version
     _
  __| |_ __ ___   __ _  ___  _ __
 / _` | '__/ _ \ / _` |/ _ \| '_ \
| (_| | | | (_) | (_| | (_) | | | |
 \__,_|_|  \___/ \__, |\___/|_| |_|
                 |___/

drogon ctl tools
version:0.9.30.771
git commit:d4710d3da7ca9e73b881cbae3149c3a570da8de4
compile config:-O3 -DNDEBUG -Wall -std=c++17 -I/root/drogon/trantor -I/root/drogon/lib/inc -I/root/drogon/orm_lib/inc -I/usr/local/include -I/usr/include/uuid -I/usr/include -I/usr/include/mysql
```

### Lệnh con create

Lệnh con `create` được sử dụng để tạo các đối tượng khác nhau. Nó hiện là chức năng chính của `drogon_ctl`. Sử dụng lệnh `dg_ctl help create` để in trợ giúp chi tiết cho lệnh này, như sau:

```console
$ dg_ctl help create
Sử dụng lệnh create để tạo một số tệp nguồn của ứng dụng web drogon

Usage:drogon_ctl create <view|controller|filter|project|model> [-options] <object name>

drogon_ctl create view <csp file name> [-o <output path>] [-n <namespace>]|[--path-to-namespace] //tạo các tệp nguồn HttpView từ tệp csp

drogon_ctl create controller [-s] <[namespace::]class_name> //tạo các tệp nguồn HttpSimpleController

drogon_ctl create controller -h <[namespace::]class_name> //tạo các tệp nguồn HttpController

drogon_ctl create controller -w <[namespace::]class_name> //tạo các tệp nguồn WebSocketController

drogon_ctl create filter <[namespace::]class_name> //tạo một filter có tên class_name

drogon_ctl create project <project_name> //tạo một project có tên project_name

drogon_ctl create model <model_path> //tạo các lớp model trong model_path
```

- #### Tạo View

  Lệnh `dg_ctl create view` được sử dụng để tạo các tệp nguồn từ các tệp CSP, hãy xem phần [View](VI-07-View). Nói chung, lệnh này không cần phải được sử dụng trực tiếp. Thực hành tốt hơn là cấu hình tệp CMake để thực thi lệnh này tự động. Ví dụ lệnh như sau, giả sử tệp CSP là `UsersList.csp`.

  ```shell
  dg_ctl create view UsersList.csp
  ```

- #### Tạo Controller

  Lệnh `dg_ctl create controller` được sử dụng để giúp người dùng tạo các tệp nguồn của Controller. Ba Controller hiện được Drogon hỗ trợ có thể được tạo bởi lệnh này.

  - Lệnh để tạo `HttpSimpleController` như sau:

  ```shell
  dg_ctl create controller SimpleControllerTest
  dg_ctl create controller webapp::v1::SimpleControllerTest
  ```

  Tham số cuối cùng là tên lớp của Controller, có thể được thêm tiền tố bởi một namespace.

  - Lệnh để tạo `HttpController` như sau:

  ```shell
  dg_ctl create controller -h ControllerTest
  dg_ctl create controller -h api::v1::ControllerTest
  ```

  - Lệnh để tạo `WebSocketController` như sau:

  ```shell
  dg_ctl create controller -w WsControllerTest
  dg_ctl create controller -w api::v1::WsControllerTest
  ```

- #### Tạo Filter

  Lệnh `dg_ctl create filter` được sử dụng để giúp người dùng tạo các tệp nguồn cho filter, hãy xem phần [Middleware và Filter](VI-06-Middleware-and-Filter).

  ```shell
  dg_ctl create filter LoginFilter
  dg_ctl create filter webapp::v1::LoginFilter
  ```

- #### Tạo Project

  Cách tốt nhất để người dùng tạo một dự án ứng dụng Drogon mới là thông qua lệnh `drogon_ctl`, như sau:

  ```shell
  dg_ctl create project TênDựÁn
  ```

  Sau khi lệnh được thực thi, một thư mục dự án hoàn chỉnh sẽ được tạo trong thư mục hiện tại. Tên thư mục là `TênDựÁn`, và người dùng có thể trực tiếp biên dịch dự án trong thư mục `build` (`cmake .. && make`). Tất nhiên, nó không có bất kỳ logic nghiệp vụ nào.

  Cấu trúc thư mục của dự án như sau:

  ```console
  ├── build                         Thư mục build
  ├── CMakeLists.txt                Tệp CMake cấu hình project
  ├── cmake_modules                 Tập lệnh CMake để tra cứu thư viện của bên thứ ba
  │   ├── FindJsoncpp.cmake
  │   ├── FindMySQL.cmake
  │   ├── FindSQLite3.cmake
  │   └── FindUUID.cmake
  ├── config.json                   Tệp cấu hình của ứng dụng Drogon, vui lòng tham khảo phần giới thiệu của tệp cấu hình.
  ├── controllers                   Thư mục nơi lưu trữ các tệp nguồn controller
  ├── filters                       Thư mục nơi lưu trữ các tệp filter
  ├── main.cc                       Chương trình chính
  ├── models                        Thư mục của tệp model cơ sở dữ liệu, tạo tệp nguồn model xem 11.2.5
  │   └── model.json
  ├── tests                         Thư mục dành cho các bài kiểm tra đơn vị/tích hợp
  │   └── test_main.cc              Điểm vào cho các bài kiểm tra
  └── views                         Thư mục nơi lưu trữ các tệp CSP của view, tệp nguồn không cần người dùng tạo thủ công, và các tệp CSP được tự động tiền xử lý để lấy các tệp nguồn view khi project được biên dịch.
  ```

- #### Tạo model

  Sử dụng lệnh `dg_ctl create model` để tạo các tệp nguồn model cơ sở dữ liệu. Tham số cuối cùng là thư mục nơi lưu trữ các model. Thư mục này phải chứa một tệp cấu hình model có tên `model.json` để cho `dg_ctl` biết cách kết nối với cơ sở dữ liệu và bảng nào cần được ánh xạ.

  Ví dụ: nếu bạn muốn tạo model trong thư mục project được đề cập ở trên, hãy thực thi lệnh sau trong thư mục project:

  ```shell
  dg_ctl create model models
  ```

  Lệnh này sẽ nhắc người dùng rằng tệp sẽ bị ghi đè trực tiếp. Sau khi người dùng nhập `y`, nó sẽ tạo ra tất cả các tệp model.

  Các tệp nguồn khác cần tham chiếu các lớp model nên include các tệp header model, chẳng hạn như:

  ```c++
  #include "models/User.h"
  ```

  Lưu ý rằng tên thư mục `models` được include để phân biệt giữa nhiều nguồn dữ liệu trong cùng một project. Xem [ORM](VI-08-3-Database-ORM).

### Kiểm tra tải

Người ta có thể sử dụng lệnh `dg_ctl press` để thực hiện kiểm tra tải (stress testing), có một số tùy chọn cho lệnh này.

- `-n num` Đặt số lượng yêu cầu (mặc định: 1)
- `-t num` Đặt số lượng luồng (mặc định: 1), Đặt số lượng thành số lượng CPU để đạt hiệu suất tối đa
- `-c num` Đặt số lượng kết nối đồng thời (mặc định: 1)
- `-q` Không có chỉ báo tiến trình (mặc định: không)

Ví dụ: người dùng có thể kiểm tra máy chủ HTTP như sau:

```shell
dg_ctl press -n 1000000 -t 4 -c 1000 -q http://localhost:8080/
dg_ctl press -n 1000000 -t 4 -c 1000 https://www.domain.com/path/to/be/tested
```


# Tiếp theo: [Giới thiệu về Controller](VI-05-0-Controller-Introduction)


