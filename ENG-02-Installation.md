This section takes Linux as an example to introduce the installation process. Other systems are similar;

## System Requirements

* The Linux kernel should be not lower than 2.6.9, 64-bit version;
* The gcc version is not less than 5.4.0;
* Use cmake as the build tool, and the cmake version should be not less than 3.5;
* Use git as the version management tool;

## Library Dependencies

* trantor, a non-blocking I/O C++ network library, also developed by the author of Drogon, has been used as a git repository submodule, no need to install in advance;
* jsoncpp, JSON's c++ library, the version should be **no less than 1.7**;
* libuuid, generating c library of uuid;
* zlib, used to support compressed transmission;
* OpenSSL, not mandatory, if the OpenSSL library is installed, drogon will support HTTPS as well, otherwise drogon only supports HTTP.
* c-ares, not mandatory, if the c-ares library is installed，drogon will be more efficient with DNS;
* libbrotli, not mandatory, if the libbrotli library is installed, drogon will support brotli compression when sending HTTP responses;
* boost, the version should be **no less than 1.61**, is required only if the C++ compiler does not support C++ 17 and if the STL doesn't fully support `std::filesystem`.
* the client development libraries of postgreSQL, mariadb and sqlite3, not mandatory, if one or more of them is installed, drogon will support access to the according database.
* gtest, not mandatory, if the gtest library is installed, the unit tests can be compiled.

## System Preparation Examples

#### Ubuntu 18.04

##### Environment

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

##### Environment

```shell
yum install git
yum install gcc
yum install gcc-c++
```

The default installed cmake version is too low, use source installation

```shell
git clone https://github.com/Kitware/CMake
cd CMake/
./bootstrap && make && make install
```

Upgrade gcc

```shell
yum install centos-release-scl
yum install devtoolset-8
scl enable devtoolset-8 bash
```

**Note: Command `scl enable devtoolset-8 bash` only activate the new gcc temporarily until the session is end. If you want to always use the new gcc, you could run command `echo "scl enable devtoolset-8 bash" >> ~/.bash_profile`, system will automatically activate the new gcc after restarting.**

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

#### MacOS 12.2
##### Environment
All the essentials are inherent in MacOS, you only need to upgrade it.
##### upgrade gcc
```shell
brew upgrade
```
##### jsoncpp

```shell
brew install jsoncpp
```
##### uuid

```shell
brew install ossp-uuid
```

##### OpenSSL

```shell
brew install openssl
```

##### zlib

```shell
brew install zlib
```

#### Windows

##### Environment
Install Visual Studio 2019 professional 2019, at least included these options:
* MSVC C++ building tools
* Windows 10 SDK
* C++ CMake tools for windows
* Test adaptor for Google Test

##### Package Manager
If python is installed on system, you could install conan package manager via pip, of course you can download the installation file from connan official website to install it also.
```
pip install conan
```
conan package manager could provide all dependencies that Drogon projector needs。

## Database Environment

**Note: These libraries below are not mandatory. You could choose to install one or more database according to your actual needs.**

**Note: If you want to develop your webapp with database, please install the database develop environment first, then install drogon, otherwise you will encounter a `NO DATABASE FOUND` issue.**



#### PostgreSQL

PostgreSQL's native C library libpq needs to be installed. The installation is as follows:

* `ubuntu 16`: ```sudo apt-get install postgresql-server-dev-all```
* `ubuntu 18`: ```sudo apt-get install postgresql-all```
* `centOS 7`: ```yum install postgresql-devel```
* `MacOS`: ```brew install postgresql```

#### MySQL

MySQL's native library does not support asynchronous read and write. Fortunately, MySQL also has a version of MariaDB maintained by the original developer community. This version is compatible with MySQL, and its development library supports asynchronous read and write. Therefore, Drogon uses the MariaDB development library to provide the right MySQL support, as a best practice，your operating system shouldn't install both Mysql and MariaDB at the same time.

MariaDB installation is as follows：

* `ubuntu`: ```sudo apt install libmariadbclient-dev```
* `centOS 7`: ```yum install mariadb-devel```
* `MacOS`: ```brew install mariadb```

#### Sqlite3

* `ubuntu`: ```sudo apt-get install libsqlite3-dev```
* `centOS`: ```yum install sqlite-devel```
* `MacOS`: ```brew install sqlite3```

### Redis
* `ubuntu`: ```sudo apt-get install libhiredis-dev```

**Note: Some of the above commands only install the development library. If you want to install a server also, please use Google search yourself.**

## Drogon Installation

Assuming that the above environment and library dependencies are all ready, the installation process is very simple;

#### Install by source in Linux

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


The default is to compile the debug version. If you want to compile the release version, the cmake command should take the following parameters:

```shell
cmake -DCMAKE_BUILD_TYPE=Release ..
```

After the installation is complete, the following files will be installed in the system（One can change the installation location with the CMAKE_INSTALL_PREFIX option）:

* The header file of drogon is installed into /usr/local/include/drogon;
* The drogon library file libdrogon.a is installed into /usr/local/lib;
* Drogon's command line tool drogon_ctl is installed into /usr/local/bin;
* The trantor header file is installed into /usr/local/include/trantor;
* The trantor library file libtrantor.a is installed into /usr/local/lib;

#### Install by source in Windows

After installed `conan` package manager, run command in PowerShell for Visual studio as bellow:
```
cd $WORK_PATH
git clone https://github.com/drogonframework/drogon
cd drogon
git submodule update --init
mkdir build
cd build
conan install .. -s compiler="Visual Studio" -s compiler.version=16 -s build_type=Debug -g cmake_paths
cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=D:/ -DCMAKE_TOOLCHAIN_FILE=./conan_paths.cmake
cmake --build . --parallel --target install
```
**Note: Must keep build type same in conan and cmake.**

After the installation is complete, the following files will be installed in the system（One can change the installation location with the CMAKE_INSTALL_PREFIX option）:

* The header file of drogon is installed into D:/include/drogon;
* The drogon library file drogon.dll is installed into D:/bin;
* Drogon's command line tool drogon_ctl.exe is installed into D:/bin;
* The trantor header file is installed into D:/include/trantor;
* The trantor library file trantor.dll is installed into D:/lib;

#### Use vcpkg

The easiest way to install drogon on windows is to use vcpkg

```
vcpkg.exe install drogon
```

Or

```
vcpkg.exe install drogon:x64-windows
```

__if you haven't install vcpkg:__

0. [Lazzy to read](https://www.youtube.com/watch?v=0ojHvu0Is6A)

1. Assuming that you don't have `cmake.exe`, `make.exe`/`nmake.exe`/`ninja.exe` and `vcpkg.exe`

2. Make it sure you're already install `git` for windows too

3. First, go to where you want to install `vcpkg`.
    - In this case, we're gonna use `C:/dev` directory.
    - If you don't have that directory yet, open your __*powershell*__ as administrator:
        - type and enter:
            - `cd c:/`
            - `mkdir dev;cd dev;` mean you will create __dev__ and go to __dev__ directory
            - `git clone https://github.com/microsoft/vcpkg` or `git clone git@github.com:microsoft/vcpkg.git`
            - `cd vcpkg`
            - `./bootstrap-vcpkg.bat` this will install `vcpkg.exe`
            note: to update your vcpkg, you just need to type `git pull`
             to make it sure that vcpkg directory always able to access:
                - add `C:/dev/vpckg` to your windows __*environment variables*__.
             - restart/re-open your __*powershell*__

4. Now check if vcpkg already installed properly, just type `vcpkg` or `vcpkg.exe`

5. To install drogon framework. Type:
    - 32-Bit: `vcpkg install drogon`
    - 64-Bit: `vcpkg install drogon:x64-windows`
    - extra : `vcpkg install jsoncpp:x64-windows zlib::x64-windows openssl::x64-windows sqlite3:x64-windows libpq:x64-windows libpqxx:x64-windows drogon[core,ctl,sqlite3,postgres,orm]:x64-windows`
    - wait till all dependencies are installed.
    - note:
        - if there's any package is/are uninstalled and you got error, just install that package. e.g.:
            - zlib : `vcpkg install zlib` or `vcpkg install zlib:x64-windows` for 64-Bit
        - to check what already installed:
            - `vcpkg list`
        - use `vcpkg search` for what available.

6. To add __*drogon_ctl*__ command and dependencies, you need to add some variables. By following this guide, you just need to add:
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
    to your windows __*environment variables*__.

7. reload/re-open your __*powershell*__, then type:
    - `drogon_ctl` or `drogon_ctl.exe`
    - press __enter__
    - if:
    ```
    usage: drogon_ctl [-v | --version] [-h | --help] <command> [<args>]
    commands list:
    create                  create some source files(Use 'drogon_ctl help create' for more information)
    help                    display this message
    press                   Do stress testing(Use 'drogon_ctl help press' for more information)
    version                 display version of this tool
    ```
    showed up, you are good to go.

8. Note:
    - you need to be familiar with building cpp libraries by using:
        - independent `gcc` or `g++` (__*[msys2](https://www.msys2.org/), [mingw-w64](https://www.mingw-w64.org/), [tdm-gcc](https://jmeubank.github.io/tdm-gcc/download/)*__)
        <br>or
        - Microsoft Visual Studio compiler

    - consider use __*make.exe/nmake.exe/ninja.exe*__ as cmake generator since configuration and build behaviour is same as *make* linux, if some devs using Linux/Windows and you are planning to deploy on Linux environment, it's less prone error when switching operating-system.

<br>

#### Use Docker Image

We also provide a pre-build docker image on the [docker hub](https://hub.docker.com/r/drogonframework/drogon). All dependencies of Drogon and Drogon itself are already installed in the docker environment, where users can build Drogon-based applications directly.

#### Use Nix Package

There is a Nix package for Drogon which was released in version 21.11.

You can use the package by adding the following `shell.nix` to your project root:

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

Enter the shell by running `nix-shell`. This will install Drogon and enter you into an environment with all its dependencies.

The Nix package has a few options which you can configure according to your needs:

| option | default value |
| --- | --- |
| sqliteSupport | true |
| postgresSupport | false |
| redisSupport | false |
| mysqlSupport | false |

Here is an example of how you can change their values:

```
  buildInputs = with pkgs; [
    (drogon.override {
      sqliteSupport = false;
    })
  ];
```

__if you haven't installed Nix:__ You can follow the instructions on the [NixOS website](https://nixos.org/download.html).

#### Include drogon source code locally

Of course, you can also include the drogon source in your project. Suppose you put the drogon under the third_party of your project directory (don't forget to update submodule in the drogon source directory). Then, you only need to add the following two lines to your project's cmake file:

```cmake
add_subdirectory(third_party/drogon)
target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
```

#### Use [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake)

You can use [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake) to include the drogon source code:

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

# 03 [Quick Start](ENG-03-Quick-Start)
