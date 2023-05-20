[English](ENG-02-Installation) | [简体中文](CHN-02-安装)

本节以Ubuntu 18.04, CentOS 7.5, MacOS 12.2为例，简介安装过程，其它系统，大同小异；

## 系统要求

*  Linux内核应不低于2.6.9，64位版本；
*  gcc版本不低于5.4.0, 建议使用11及以上版本；
*  构建工具是cmake,cmake版本应不低于3.5；
*  git版本管理工具；

## 依赖库

*  内置
   *  trantor，non-blocking I/O C++网络库，也是作者开发，已作为git仓库submodule，无需提前安装；
*  必须
   *  jsoncpp，json的c++库，版本**不低于1.7**；
   *  libuuid，生成uuid的c库；
   *  zlib，用于支持压缩传输；
*  可选
   *  boost，版本**不低于1.61**，只在C++编译器不支持c++17或STL库不完整支持`std::filesystem`时才需要安装；
   *  OpenSSL，安装后drogon将支持HTTPS，否则drogon只支持HTTP；
   *  c-ares, 安装后drogon对DNS的支持会具有更好的性能；
   *  libbrotli，安装后drogon的HTTP响应会支持brotli压缩；
   *  postgreSQL, mariadb, sqlite3的客户端开发库，安装后drogon会提供对响应的库的访问能力；
   *  hiredis, 安装后drogon将支持redis的访问；
   *  gtest, 安装后drogon的单元测试代码可以被编译；
   *  yaml-cpp, 安装后drogon将支持yaml格式的配置文件;

## 系统准备范例

#### Ubuntu 18.04

* 环境

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

* OpenSSL (可选，提供HTTPS支持)

  ```shell
  sudo apt install openssl
  sudo apt install libssl-dev
  ```


#### CentOS 7.5

* 环境

  ```shell
  yum install git
  yum install gcc
  yum install gcc-c++
  ```

  ```shell
  # 默认安装的 cmake 版本太低，使用源码安装
  git clone https://github.com/Kitware/CMake
  cd CMake/
  ./bootstrap && make && make install
  ```

  ```shell
  # 升级 gcc
  yum install centos-release-scl
  yum install devtoolset-11
  scl enable devtoolset-11 bash
  ```

  > **注意: `scl enable devtoolset-11 bash`命令仅是临时性的使新的gcc生效，直到会话结束。如果想永久使用新版gcc,可以使用命令`echo "scl enable devtoolset-11 bash" >> ~/.bash_profile`, 系统重新启动后将自动使用新版gcc。**

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

* OpenSSL (可选，提供HTTPS支持)

  ```shell
  yum install openssl-devel
  ```


#### MacOS 12.2

* 环境

  MacOS 內建都有,更新即可

  ```shell
  # 升級 gcc
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

* OpenSSL (可选，提供HTTPS支持)

  ```shell
  brew install openssl
  ```

#### Windows

* 环境:

  安装Visual Studio 2019专业版,安装选项中至少包括：

  * MSVC C++生成工具
  * Windows 10 SDK
  * 用于Windows的C++ CMake工具
  * Google Test测试适配器

`connan`包管理器可以提供Drogon项目的所有依赖,  如果有python环境，可以通过pip安装`connan`包管理器。

```
pip install conan
```
>当然也可以通过从[官网](https://conan.io/)下载`connan`的安装文件进行安装。

创建`conanfile.txt`文件中并添加如下内容:


* jsoncpp

  ```txt
  [requires]
  jsoncpp/1.9.4
  ```

* uuid

  不需要安装，Windows 10 SDK已经包含了uuid库。

* zlib

  ```txt
  [requires]
  zlib/1.2.11
  ```

* OpenSSL (可选，提供HTTPS支持)

  ```txt
  [requires]
  openssl/1.1.1t
  ```

## 数据库环境 (可选)

> **注意：下面的这些库都不是必须的, 用户可以根据实际需求选择安装一个或者多个数据库。**

> **注意：如果将来的开发需要用到数据库，请先安装好数据库环境，再安装drogon, 否则，会出现找不到数据库的问题。**

* #### PostgreSQL

  PostgreSQL的原生C库libpq是需要安装的，安装方法如下：

  * `ubuntu 16`: `sudo apt-get install postgresql-server-dev-all`
  * `ubuntu 18`: `sudo apt-get install postgresql-all`
  * `centOS 7`: `yum install postgresql-devel`
  * `MacOS`: `brew install postgresql`
  * `Windows conanfile`: `libpq/13.2`

* #### MySQL

  MySQL的原生库不支持异步读写，而通过同步接口+线程池的方式对上层提供异步接口并不是一个好的策略，幸好，MySQL还有一个原开发者社区维护的版本MariaDB，该版本和MySQL的对应版本兼容，并且它的开发库支持异步读写，因此，Drogon的MySQL支持采用MariaDB开发库，你的系统，Mysql和MariaDB最好不要混用，可以统一安装成MariaDB。

  安装方法如下：

  * `ubuntu`: `sudo apt install libmariadbclient-dev`
  * `centOS 7`: `yum install mariadb-devel`
  * `MacOS`: `brew install mariadb`  
  * `Windows conanfile`: `libmariadb/3.1.13`

* #### Sqlite3

  * `ubuntu`: `sudo apt-get install libsqlite3-dev`
  * `centOS`: `yum install sqlite-devel`
  * `MacOS`: `brew install sqlite3`
  * `Windows conanfile`: `sqlite3/3.36.0`

* #### Redis

  * `ubuntu`: `sudo apt-get install libhiredis-dev`
  * `centOS`: `yum install hiredis-devel`
  * `MacOS`: `brew install hiredis`
  * `Windows conanfile`: `hiredis/1.0.0`

> **注意: 上述有些命令只安装了开发库，如果还要安装server端，请自行google。**

## 安装drogon

假设上述系统环境和库依赖都已经准备好，安装过程是非常简单的；

* #### Linux源码安装

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

  > 默认是编译debug版本，如果想编译release版本，cmake命令要带如下参数：

  ```shell
  cmake -DCMAKE_BUILD_TYPE=Release ..
  ```

  安装结束后，将有如下文件被安装在系统中(CMAKE_INSTALL_PREFIX可以改变安装位置)：

  * drogon的头文件被安装到/usr/local/include/drogon中；
  * drogon的库文件libdrogon.a被安装到/usr/local/lib中；
  * drogon的命令行工具drogon_ctl被安装到/usr/local/bin中；
  * trantor的头文件被安装到/usr/local/include/trantor中；
  * trantor的库文件libtrantor.a被安装到/usr/local/lib中；

* #### Windows 源码安装

  1. 下载Drogon源码

      ```shell
      cd $WORK_PATH
      git clone https://github.com/drogonframework/drogon
      cd drogon
      git submodule update --init
      ```

  2. 安装依赖库

      ```dos
      mkdir build
      cd build
      conan profile detect --force
      conan install .. -s compiler="msvc" -s compiler.version=193 -s compiler.cppstd=17 -s build_type=Debug  --output-folder . --build=missing
      ```

     > 编辑`conanfile.txt`文件可以添加依赖，修改依赖库的版本。


  3. 编译并安装

      ```dos
      cmake ..  -DCMAKE_BUILD_TYPE=Debug -DCMAKE_TOOLCHAIN_FILE="conan_toolchain.cmake" -DCMAKE_POLICY_DEFAULT_CMP0091=NEW -DCMAKE_INSTALL_PREFIX="D:"
      cmake --build . --parallel --target install
      ```

  > **注意: conan和cmake的build type必须保持一致。**

  安装结束后，将有如下文件被安装在系统中(`CMAKE_INSTALL_PREFIX` 可以改变安装位置)：

  * drogon的头文件被安装到`D:/include/drogon`中；
  * drogon的库文件drogon.dll被安装到`D:/bin`中；
  * drogon的命令行工具drogon_ctl.exe 被安装到`D:/bin`中；
  * trantor的头文件被安装到`D:/include/trantor`中；
  * trantor的库文件trantor.dll被安装到`D:/bin`中；

  添加`bin`和`cmake`路径到`path`环境变量中：
  ```
  D:\bin
  ```
  ```
  D:\lib\cmake\Drogon
  ```
  ```
  D:\lib\cmake\Trantor
  ```

* #### Windows vcpkg安装
  [观看安装教程](https://www.youtube.com/watch?v=0ojHvu0Is6A)

  **安装 vckpg:**
  1. `git`安装`vcpkg`。

     ```
     git clone https://github.com/microsoft/vcpkg
     cd vcpkg
     ./bootstrap-vcpkg.bat
     ```

     > 说明: 要升级vcpkg, 只需要输入`git pull`

  2. 将 `vpckg`路径添加到环境变量 **_path_**.
  3. 终端中输入`vcpkg`或者`vcpkg.exe`检查vcpkg已经正确安装。

  **正式安装 Drogon:**

  1. 输入指令安装drogon框架:

     * 32-Bit: `vcpkg install drogon`
     * 64-Bit: `vcpkg install drogon:x64-windows`
     * extra : `vcpkg install jsoncpp:x64-windows zlib::x64-windows openssl::x64-windows sqlite3:x64-windows libpq:x64-windows libpqxx:x64-windows drogon[core,ctl,sqlite3,postgres,orm]:x64-windows`

     注意:

     * 如果有依赖包没有安装而出现错误, 只需安装这个包,例如:

       zlib : `vcpkg install zlib` 或者 `vcpkg install zlib:x64-windows` for 64-Bit

     * 检查安装结果:

       `vcpkg list`

     * 需要运行`vcpkg install drogon[ctl]`(32 bit)或者`vcpkg install drogon[ctl]:x64-windows`(64 bit)以包含 drogon_ctl。更多的安装特性选项请运行`vcpkg search drogon`查看。

  2. 添加__*drogon_ctl*__命令和依赖到环境变量__*path*__:

     ```
     C:\Dev\vcpkg\installed\x64-windows\tools\drogon
     ```

     ```
     C:\Dev\vcpkg\installed\x64-windows\bin
     ```

     ```
     C:\Dev\vcpkg\installed\x64-windows\lib
     ```

     ```
     C:\Dev\vcpkg\installed\x64-windows\include
     ```

  3. 重启 __*powershell*__, 输入:`drogon_ctl`或者`drogon_ctl.exe`, 如果出现:
     ```
     usage: drogon_ctl [-v | --version] [-h | --help] <command> [<args>]
     commands list:
     create                  create some source files(Use 'drogon_ctl help create' for more information)
     help                    display this message
     press                   Do stress testing(Use 'drogon_ctl help press' for more information)
     version                 display version of this tool
     ```
     说明已经安装好了。

> 说明:
> 你需要熟悉用这些生成CPP库的工具:
> `gcc`或者`g++`(**_[msys2](https://www.msys2.org/), [mingw-w64](https://www.mingw-w64.org/), [tdm-gcc](https://jmeubank.github.io/tdm-gcc/download/)_**)或Microsoft Visual Studio compiler。

> 请考虑使用**_make.exe/nmake.exe/ninja.exe_**来进行构建，因为它们的配置和行为和 Linux上的make一致， 如果使用Linux/Windows混合开发，再发布到Linux上，在进行系统切换时，可以减少错误。

* #### 使用docker镜像

  我们也在[docker hub](https://hub.docker.com/r/drogonframework/drogon)上提供了构建好的docker镜像. 在这个docker里Drogon和它所有的依赖都已经安装完毕，用户可以在上面直接开发Drogon应用程序。

* #### 使用Nix包

  Nix包管理器在版本21.11后提供了Drogon的Nix包。

  > **如果你尚未安装 Nix:** 你可以按照[NixOS website](https://nixos.org/download.html)的说明进行操作.

  你可以在你项目的根目录下添加下面的`shell.nix`使用Drogon包:

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

  通过运行`nix-shell`进入shell。它将安装Drogon，并使你拥有安装了所有依赖的环境。

  Drogon的Nix包有一些选项，你可以按照需要进行配置:

  | 选项            | 默认值 |
  | --------------- | ------ |
  | sqliteSupport   | true   |
  | postgresSupport | false  |
  | redisSupport    | false  |
  | mysqlSupport    | false  |

  这里是如何更改选项值的一个例子:

  ```
    buildInputs = with pkgs; [
      (drogon.override {
        sqliteSupport = false;
      })
    ];
  ```

* #### 使用CPM.cmake

  你可以使用[CPM.cmake](https://github.com/cpm-cmake/CPM.cmake)来包含drogon的源代码：

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

* #### 直接包含drogon源码

  当然，你也可以在你的项目中包含drogon源码，比如将drogon放置在你的项目目录的 third_party下，那么，你只需要在你项目的cmake文件里添加如下两行：

  ```cmake
  add_subdirectory(third_party/drogon)
  target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
  ```

# 03 [快速开始](CHN-03-快速开始)
