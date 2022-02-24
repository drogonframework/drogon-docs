本节以Linux为例，简介安装过程，其它系统，大同小异；

## 系统要求

*  Linux内核应不低于2.6.9，64位版本；
*  gcc版本不低于5.4.0；
*  构建工具是cmake,cmake版本应不低于3.5；
*  git版本管理工具；

## 依赖库

*  trantor，non-blocking I/O C++网络库，也是作者开发，已作为git仓库submodule，无需提前安装；
*  jsoncpp，json的c++库，版本**不低于1.7**；
*  libuuid，生成uuid的c库；
*  zlib，用于支持压缩传输；
*  OpenSSL，并非必须，如果安装了OpenSSL库，drogon将支持HTTPS，否则drogon只支持HTTP；
*  c-ares, 并非必须，如果安装了ares库，drogon对DNS的支持会具有更好的性能；
*  libbrotli，并非必须，如果安装了brotli库，drogon的HTTP响应会支持brotli压缩；
*  boost，版本**不低于1.61**，只在C++编译器不支持c++17或STL库不完整支持`std::filesystem`时才需要安装；
*  postgreSQL, mariadb, sqlite3的客户端开发库，并非必须，安装后drogon会提供对响应的库的访问能力；
*  gtest, 并非必须，如果安装了gtest库，drogon的单元测试代码可以被编译；


## 系统准备范例

#### Ubuntu 18.04

##### 环境

```shell
sudo apt install git
sudo apt install gcc
sudo apt install g++
sudo apt install cmake
```

##### jsoncpp

```shell
sudo apt install libjsoncpp-dev
```

##### uuid

```shell
sudo apt install uuid-dev
```

##### OpenSSL

```shell
sudo apt install openssl
sudo apt install libssl-dev
```

##### zlib

```shell
sudo apt install zlib1g-dev
```


#### CentOS 7.5

##### 环境

```shell
yum install git
yum install gcc
yum install gcc-c++
```

默认安装的cmake版本太低，使用源码安装

```shell
git clone https://github.com/Kitware/CMake
cd CMake/
./bootstrap && make && make install
```

升级gcc

```shell
yum install centos-release-scl
yum install devtoolset-8
scl enable devtoolset-8 bash
```

**注意: `scl enable devtoolset-8 bash`命令仅是临时性的使新的gcc生效，直到会话结束。如果想永久使用新版gcc,可以使用命令`echo "scl enable devtoolset-8 bash" >> ~/.bash_profile`, 系统重新启动后将自动使用新版gcc。**


##### jsoncpp

```shell
git clone https://github.com/open-source-parsers/jsoncpp
cd jsoncpp/
mkdir build
cd build
cmake ..
make && make install
```

##### uuid

```shell
yum install libuuid-devel
```

##### OpenSSL

```shell
yum install openssl-devel
```

##### zlib

```shell
yum install zlib-devel
```

## 数据库环境

**注意：下面的这些库都不是必须的, 用户可以根据实际需求选择安装一个或者多个数据库。**

**注意：如果将来的开发需要用到数据库，请先安装好数据库环境，再安装drogon, 否则，会出现找不到数据库的问题。**

#### PostgreSQL

PostgreSQL的原生C库libpq是需要安装的，安装方法如下：

* `ubuntu 16`: ```sudo apt-get install postgresql-server-dev-all```
* `ubuntu 18`: ```sudo apt-get install postgresql-all```
* `centOS 7`: ```yum install postgresql-devel```
* `MacOS`: ```brew install postgresql```

#### MySQL

MySQL的原生库不支持异步读写，而通过同步接口+线程池的方式对上层提供异步接口并不是一个好的策略，幸好，MySQL还有一个原开发者社区维护的版本MariaDB，该版本和MySQL的对应版本兼容，并且它的开发库支持异步读写，因此，Drogon的MySQL支持采用MariaDB开发库，你的系统，Mysql和MariaDB最好不要混用，可以统一安装成MariaDB。

安装方法如下：

* `ubuntu`: ```sudo apt install libmariadbclient-dev```
* `centOS 7`: ```yum install mariadb-devel```
* `MacOS`: ```brew install mariadb```

#### Sqlite3

* `ubuntu`: ```sudo apt-get install libsqlite3-dev```
* `centOS`: ```yum install sqlite-devel```
* `MacOS`: ```brew install sqlite3```


**注意: 上述有些命令只安装了开发库，如果还要安装server端，请自行google。**

## 安装drogon

假设上述系统环境和库依赖都已经准备好，安装过程是非常简单的；

```shell
cd $WORK_PATH
git clone https://github.com/an-tao/drogon
cd drogon
git submodule update --init
mkdir build
cd build
cmake ..
make && sudo make install
```

默认是编译debug版本，如果想编译release版本，cmake命令要带如下参数：

```shell
cmake -DCMAKE_BUILD_TYPE=Release .. 
```

安装结束后，将有如下文件被安装在系统中(CMAKE_INSTALL_PREFIX可以改变安装位置)：

* drogon的头文件被安装到/usr/local/include/drogon中；
* drogon的库文件libdrogon.a被安装到/usr/local/lib中；
* drogon的命令行工具drogon_ctl被安装到/usr/local/bin中；
* trantor的头文件被安装到/usr/local/include/trantor中；
* trantor的库文件libtrantor.a被安装到/usr/local/lib中；

#### 使用vcpkg安装

在windows下最简便的安装方式是使用vcpkg

```
vcpkg.exe install drogon
```

或者

```
vcpkg.exe install drogon:x64-windows
```

#### 使用docker镜像

我们也在[docker hub](https://hub.docker.com/r/drogonframework/drogon)上提供了构建好的docker镜像. 在这个docker里Drogon和它所有的依赖都已经安装完毕，用户可以在上面直接开发Drogon应用程序。

#### 直接使用drogon源码

当然，你也可以在你的项目中包含drogon源码，比如将drogon放置在你的项目目录的third_party下，那么，你只需要在你项目的cmake文件里添加如下两行：

```cmake
add_subdirectory(third_party/drogon)
target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
```

# 03 [快速开始](CHN-03-快速开始)
