## Cài đặt Drogon

Phần này lấy Ubuntu 18.04, CentOS 7.5, MacOS 12.2 làm ví dụ để giới thiệu quy trình cài đặt. Các hệ thống khác tương tự;

## Yêu cầu hệ thống

* Nhân Linux không nên thấp hơn 2.6.9, phiên bản 64-bit;
* Phiên bản GCC không nhỏ hơn 5.4.0, đề xuất sử dụng phiên bản 11 trở lên;
* Sử dụng CMake làm công cụ build, và phiên bản CMake không nên thấp hơn 3.5;
* Sử dụng Git làm công cụ quản lý phiên bản;

## Các thư viện phụ thuộc

* Tích hợp sẵn
  * Trantor, một thư viện mạng C++ I/O không chặn, cũng được phát triển bởi tác giả của Drogon, đã được sử dụng như một submodule của kho lưu trữ Git, không cần cài đặt trước;
* Bắt buộc
  * Jsoncpp, thư viện C++ của JSON, phiên bản phải **không thấp hơn 1.7**;
  * libuuid, tạo thư viện C của UUID;
  * zlib, được sử dụng để hỗ trợ truyền nén;
* Tùy chọn
  * Boost, phiên bản phải **không thấp hơn 1.61**, chỉ được yêu cầu nếu trình biên dịch C++ không hỗ trợ C++17 và nếu STL không hỗ trợ đầy đủ `std::filesystem`.
  * OpenSSL, sau khi cài đặt, Drogon cũng sẽ hỗ trợ HTTPS, nếu không Drogon chỉ hỗ trợ HTTP.
  * c-ares, sau khi cài đặt, Drogon sẽ hiệu quả hơn với DNS;
  * libbrotli, sau khi cài đặt, Drogon sẽ hỗ trợ nén Brotli khi gửi phản hồi HTTP;

  * các thư viện phát triển client của PostgreSQL, MariaDB và SQLite3, nếu một hoặc nhiều thư viện trong số đó được cài đặt, Drogon sẽ hỗ trợ truy cập vào cơ sở dữ liệu tương ứng.
  * hiredis, sau khi cài đặt, Drogon sẽ hỗ trợ truy cập vào Redis.
  * gtest, sau khi cài đặt, các bài kiểm tra đơn vị có thể được biên dịch.
  * yaml-cpp, sau khi cài đặt, Drogon sẽ hỗ trợ tệp cấu hình với định dạng YAML.


## Ví dụ về chuẩn bị hệ thống

#### Ubuntu 18.04

* Môi trường

  ```shell
  sudo apt install git
  sudo apt install gcc
  sudo apt install g++
  sudo apt install cmake
  ```

* jsoncpp

  ```shell
  sudo apt install libjsoncpp-dev
  ```

* uuid

  ```shell
  sudo apt install uuid-dev
  ```

* zlib

  ```shell
  sudo apt install zlib1g-dev
  ```

* OpenSSL (Tùy chọn, nếu bạn muốn hỗ trợ HTTPS)

  ```shell
  sudo apt install openssl
  sudo apt install libssl-dev
  ```

#### CentOS 7.5

* Môi trường

  ```shell
  yum install git
  yum install gcc
  yum install gcc-c++
  ```

  ```shell
  # Phiên bản CMake được cài đặt mặc định quá thấp, hãy sử dụng cài đặt từ mã nguồn
  git clone https://github.com/Kitware/CMake
  cd CMake/
  ./bootstrap && make && make install
  ```

  ```shell
  # Nâng cấp GCC
  yum install centos-release-scl
  yum install devtoolset-11
  scl enable devtoolset-11 bash
  ```

  > **Lưu ý: Lệnh `scl enable devtoolset-11 bash` chỉ kích hoạt GCC mới tạm thời cho đến khi phiên kết thúc. Nếu bạn muốn luôn sử dụng GCC mới, bạn có thể chạy lệnh `echo "scl enable devtoolset-11 bash" >> ~/.bash_profile`, hệ thống sẽ tự động kích hoạt GCC mới sau khi khởi động lại.**

* jsoncpp

  ```shell
  git clone https://github.com/open-source-parsers/jsoncpp
  cd jsoncpp/
  mkdir build
  cd build
  cmake ..
  make && make install
  ```

* uuid

  ```shell
  yum install libuuid-devel
  ```

* zlib

  ```shell
  yum install zlib-devel
  ```

* OpenSSL (Tùy chọn, nếu bạn muốn hỗ trợ HTTPS)

  ```shell
  yum install openssl-devel
  ```

#### MacOS 12.2

* Môi trường

  Tất cả các yếu tố cần thiết đều có sẵn trong MacOS, bạn chỉ cần nâng cấp nó.

  ```shell
  # nâng cấp GCC
  brew upgrade
  ```

* jsoncpp

  ```shell
  brew install jsoncpp
  ```

* uuid

  ```shell
  brew install ossp-uuid
  ```

* zlib

  ```shell
  brew install zlib
  ```

* OpenSSL (Tùy chọn, nếu bạn muốn hỗ trợ HTTPS)

  ```shell
  brew install openssl
  ```

#### Windows

* Môi trường (Visual Studio 2019)
  Cài đặt Visual Studio 2019 Professional 2019, ít nhất bao gồm các tùy chọn sau:
  * Công cụ xây dựng MSVC C++
  * Windows 10 SDK
  * Công cụ C++ CMake dành cho Windows
  * Bộ điều hợp kiểm tra cho Google Test

Trình quản lý gói `Conan` có thể cung cấp tất cả các dependency mà dự án Drogon cần. Nếu Python được cài đặt trên hệ thống, bạn có thể cài đặt trình quản lý gói `Conan` thông qua pip. 

```
pip install conan
```
> Tất nhiên bạn cũng có thể tải xuống tệp cài đặt từ [trang web chính thức của Conan](https://conan.io/) để cài đặt.

Tạo `conanfile.txt` và thêm nội dung sau vào đó:

* jsoncpp

  ```txt
  [requires]
  jsoncpp/1.9.4
  ```

* uuid

  Không yêu cầu cài đặt, Windows 10 SDK đã bao gồm thư viện UUID.

* zlib

  ```txt
  [requires]
  zlib/1.2.11
  ```

* OpenSSL (Tùy chọn, nếu bạn muốn hỗ trợ HTTPS)

  ```txt
  [requires]
  openssl/1.1.1t
  ```

## Môi trường cơ sở dữ liệu (Tùy chọn)

> **Lưu ý: Các thư viện dưới đây không bắt buộc. Bạn có thể chọn cài đặt một hoặc nhiều cơ sở dữ liệu theo nhu cầu thực tế của mình.**

> **Lưu ý: Nếu bạn muốn phát triển ứng dụng web của mình với cơ sở dữ liệu, vui lòng cài đặt môi trường phát triển cơ sở dữ liệu trước, sau đó cài đặt Drogon, nếu không bạn sẽ gặp phải sự cố `NO DATABASE FOUND`.**

* #### PostgreSQL

  Cần cài đặt thư viện C libpq gốc của PostgreSQL. Việc cài đặt như sau:

  * `ubuntu 16`: `sudo apt-get install postgresql-server-dev-all`
  * `ubuntu 18`: `sudo apt-get install postgresql-all`
  * `centOS 7`: `yum install postgresql-devel`
  * `MacOS`: `brew install postgresql`
  * `Windows conanfile`: `libpq/13.4`


* #### MySQL

  Thư viện gốc của MySQL không hỗ trợ đọc và ghi bất đồng bộ. May mắn thay, MySQL cũng có một phiên bản MariaDB được duy trì bởi cộng đồng nhà phát triển ban đầu. Phiên bản này tương thích với MySQL và thư viện phát triển của nó hỗ trợ đọc và ghi bất đồng bộ. Do đó, Drogon sử dụng thư viện phát triển MariaDB để cung cấp hỗ trợ MySQL phù hợp, cách tốt nhất là hệ điều hành của bạn không nên cài đặt cả MySQL và MariaDB cùng một lúc.

  Cài đặt MariaDB như sau：

  * `ubuntu`: `sudo apt install libmariadbclient-dev`
  * `centOS 7`: `yum install mariadb-devel`
  * `MacOS`: `brew install mariadb`
  * `Windows conanfile`: `libmariadb/3.1.13`


* #### SQLite3

  * `ubuntu`: `sudo apt-get install libsqlite3-dev`
  * `centOS`: `yum install sqlite-devel`
  * `MacOS`: `brew install sqlite3`
  * `Windows conanfile`: `sqlite3/3.36.0`


* #### Redis
  * `ubuntu`: `sudo apt-get install libhiredis-dev`
  * `centOS`: `yum install hiredis-devel`
  * `MacOS`: `brew install hiredis`
  * `Windows conanfile`: `hiredis/1.0.0`


> **Lưu ý: Một số lệnh trên chỉ cài đặt thư viện phát triển. Nếu bạn cũng muốn cài đặt máy chủ, vui lòng tự mình sử dụng Google tìm kiếm.**

## Cài đặt Drogon

Giả sử rằng môi trường ở trên và các thư viện phụ thuộc đều đã sẵn sàng, quá trình cài đặt rất đơn giản;

* #### Cài đặt bằng mã nguồn trong Linux

  ```shell
  cd $WORK_PATH
  git clone https://github.com/drogonframework/drogon
  cd drogon
  git submodule update --init
  mkdir build
  cd build
  cmake ..
  make && sudo make install
  ```

  > Mặc định là biên dịch phiên bản debug. Nếu bạn muốn biên dịch phiên bản release, lệnh CMake nên nhận các tham số sau:

  ```shell
  cmake -DCMAKE_BUILD_TYPE=Release ..
  ```

  Sau khi cài đặt hoàn tất, các tệp sau sẽ được cài đặt trong hệ thống (Bạn có thể thay đổi vị trí cài đặt bằng tùy chọn `CMAKE_INSTALL_PREFIX`):

  * Tệp tiêu đề của Drogon được cài đặt vào `/usr/local/include/drogon`;
  * Tệp thư viện Drogon `libdrogon.a` được cài đặt vào `/usr/local/lib`;
  * Công cụ dòng lệnh `drogon_ctl` của Drogon được cài đặt vào `/usr/local/bin`;
  * Tệp tiêu đề Trantor được cài đặt vào `/usr/local/include/trantor`;
  * Tệp thư viện Trantor `libtrantor.a` được cài đặt vào `/usr/local/lib`;


* #### Cài đặt bằng mã nguồn trong Windows

  1. Tải xuống mã nguồn Drogon

      ```dos
      cd %WORK_PATH%
      git clone https://github.com/drogonframework/drogon
      cd drogon
      git submodule update --init
      ```

  2. Cài đặt dependency

     Cài đặt dependency thông qua `Conan`:

      ```dos
      mkdir build
      cd build
      conan profile detect --force
      conan install .. -s compiler="msvc" -s compiler.version=193  -s compiler.cppstd=17 -s build_type=Debug  --output-folder . --build=missing
      ```

      > Sửa đổi `conanfile.txt` để thay đổi phiên bản của dependency.

  3. Biên dịch và cài đặt
      ```dos
      cmake ..  -DCMAKE_BUILD_TYPE=Debug -DCMAKE_TOOLCHAIN_FILE="conan_toolchain.cmake" -DCMAKE_POLICY_DEFAULT_CMP0091=NEW -DCMAKE_INSTALL_PREFIX="D:"
      cmake --build . --parallel --target install
      ```

  > **Lưu ý: Phải giữ kiểu build giống nhau trong Conan và CMake.**

  Sau khi cài đặt hoàn tất, các tệp sau sẽ được cài đặt trong hệ thống (Bạn có thể thay đổi vị trí cài đặt bằng tùy chọn `CMAKE_INSTALL_PREFIX`):

  * Tệp tiêu đề của Drogon được cài đặt vào `D:/include/drogon`;
  * Tệp thư viện Drogon `drogon.dll` được cài đặt vào `D:/bin`;
  * Công cụ dòng lệnh `drogon_ctl.exe` của Drogon được cài đặt vào `D:/bin`;
  * Tệp tiêu đề Trantor được cài đặt vào `D:/include/trantor`;
  * Tệp thư viện Trantor `trantor.dll` được cài đặt vào `D:/lib`;

  Thêm thư mục `bin` và `cmake` vào `PATH`:
  ```
  D:\bin
  ```
  ```
  D:\lib\cmake\Drogon
  ```
  ```
  D:\lib\cmake\Trantor
  ```


* #### Cài đặt bằng vcpkg trong Windows
  
  [Lười đọc](https://www.youtube.com/watch?v=0ojHvu0Is6A)

  **Cài đặt vcpkg:**

  1. Cài đặt `vcpkg` bằng `Git`.
  
     ```
     git clone https://github.com/microsoft/vcpkg
     cd vcpkg
     .\bootstrap-vcpkg.bat
     ```

     > Lưu ý: Để cập nhật vcpkg của bạn, bạn chỉ cần gõ `git pull`

  2. Thêm `vcpkg` vào **_đường dẫn_** biến môi trường Windows của bạn.
  3. Bây giờ hãy kiểm tra xem vcpkg đã được cài đặt đúng cách chưa, chỉ cần gõ `vcpkg` hoặc `vcpkg.exe`

  **Bây giờ Cài đặt Drogon:**

  1. Để cài đặt framework Drogon. Gõ:

     * 32-Bit: `vcpkg install drogon`
     * 64-Bit: `vcpkg install drogon:x64-windows`
     * Thêm : `vcpkg install jsoncpp:x64-windows zlib:x64-windows openssl:x64-windows sqlite3:x64-windows libpq:x64-windows libpqxx:x64-windows drogon[core,ctl,sqlite3,postgres,orm]:x64-windows`

     Lưu ý:

     * Nếu có bất kỳ gói nào bị gỡ cài đặt và bạn gặp lỗi, chỉ cần cài đặt gói đó. Ví dụ:

       zlib : `vcpkg install zlib` hoặc `vcpkg install zlib:x64-windows` cho 64-Bit

     * Để kiểm tra những gì đã được cài đặt:

       `vcpkg list`

     * Sử dụng `vcpkg search` cho những gì có sẵn.

  2. Để thêm lệnh **_drogon_ctl_** và dependency, bạn cần thêm một số biến. Bằng cách làm theo hướng dẫn này, bạn chỉ cần thêm:

     ```
     C:\dev\vcpkg\installed\x64-windows\tools\drogon
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\bin
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\lib
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\include
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\share
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\debug\bin
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\debug\lib
     ```

     vào **_biến môi trường_** Windows của bạn. Sau đó, khởi động lại/mở lại **_PowerShell_** của bạn.

  3. Tải lại/mở lại **_PowerShell_** của bạn, sau đó nhập: `drogon_ctl` hoặc `drogon_ctl.exe` nếu:
     ```
     usage: drogon_ctl [-v | --version] [-h | --help] <command> [<args>]
     commands list:
     create                  create some source files(Use 'drogon_ctl help create' for more information)
     help                    display this message
     press                   Do stress testing(Use 'drogon_ctl help press' for more information)
     version                 display version of this tool
     ```
     hiển thị, bạn đã sẵn sàng để bắt đầu.

  > Lưu ý:
  > Bạn cần phải quen thuộc với việc xây dựng các thư viện C++ bằng cách sử dụng: `GCC` hoặc `G++` độc lập (**_[MSYS2](https://www.msys2.org/), [MinGW-w64](https://www.mingw-w64.org/), [TDM-GCC](https://jmeubank.github.io/tdm-gcc/download/)_**) hoặc trình biên dịch Microsoft Visual Studio

  > Xem xét sử dụng **_make.exe/nmake.exe/ninja.exe_** làm trình tạo CMake vì cấu hình và hành vi build giống như _make* Linux, nếu một số nhà phát triển sử dụng Linux/Windows và bạn đang có kế hoạch triển khai trên môi trường Linux, thì lỗi sẽ ít xảy ra hơn khi chuyển đổi hệ điều hành.


* #### Sử dụng Docker Image

  Chúng tôi cũng cung cấp một Docker image được build sẵn trên [Docker Hub](https://hub.docker.com/r/drogonframework/drogon). Tất cả các dependency của Drogon và Drogon đã được cài đặt trong môi trường Docker, nơi người dùng có thể build các ứng dụng dựa trên Drogon trực tiếp.


* #### Sử dụng Gói Nix

  Có một gói Nix cho Drogon đã được phát hành trong phiên bản 21.11.

  > **Nếu bạn chưa cài đặt Nix:** Bạn có thể làm theo hướng dẫn trên [trang web NixOS](https://nixos.org/download.html).


  Bạn có thể sử dụng gói bằng cách thêm `shell.nix` sau vào thư mục gốc dự án của bạn:

  ```
  { pkgs ? import <nixpkgs> {} }:
  pkgs.mkShell {
    nativeBuildInputs = with pkgs; [
      cmake
    ];

    buildInputs = with pkgs; [
      drogon
    ];
  }
  ```

  Nhập shell bằng cách chạy `nix-shell`. Thao tác này sẽ cài đặt Drogon và đưa bạn vào một môi trường với tất cả các dependency của nó.

  Gói Nix có một vài tùy chọn mà bạn có thể cấu hình theo nhu cầu của mình:

  | Tùy chọn          | Giá trị mặc định |
  | --------------- | ------------- |
  | sqliteSupport   | true          |
  | postgresSupport | false         |
  | redisSupport    | false         |
  | mysqlSupport    | false         |

  Đây là một ví dụ về cách bạn có thể thay đổi giá trị của chúng:

  ```
    buildInputs = with pkgs; [
      (drogon.override {
        sqliteSupport = false;
      })
    ];
  ```


* #### Sử dụng CPM.cmake

  Bạn có thể sử dụng [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake) để bao gồm mã nguồn Drogon:


  ```cmake
  include(cmake/CPM.cmake)

  CPMAddPackage(
      NAME drogon
      VERSION 1.7.5
      GITHUB_REPOSITORY drogonframework/drogon
      GIT_TAG v1.7.5
  )

  target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
  ```


* #### Bao gồm mã nguồn Drogon cục bộ

  Tất nhiên, bạn cũng có thể bao gồm mã nguồn Drogon trong dự án của mình. Giả sử bạn đặt Drogon dưới thư mục `third_party` của thư mục dự án của bạn (đừng quên cập nhật submodule trong thư mục mã nguồn Drogon). Sau đó, bạn chỉ cần thêm hai dòng sau vào tệp CMake của dự án:

  ```cmake
  add_subdirectory(third_party/drogon)
  target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
  ```

# Tiếp theo: [Bắt đầu nhanh](VI-03-Quick-Start) 

