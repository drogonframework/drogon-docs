[English](/ENG/ENG-12-drogon_ctl-Command) | [简体中文](/CHN/CHN-11-drogon_ctl命令)

**Drogon**框架编译安装后，一个命令行程序 `drogon_ctl` 也同时被安装于系统，为了方便，还安装了一个完全一样的副本 `dg_ctl`。用户可以按喜好自行选用。

该程序目前的主要功能是为了方便用户创建各种drogon工程文件，使用`dg_ctl help`命令可以看到它支持的功能，如下：

```console
$ dg_ctl help
usage: drogon_ctl <command> [<args>]
commands list:
create                  create some source files(Use 'drogon_ctl help create' for more information)
help                    display this message
version                 display version of this tool
press                   Do stress testing(Use 'drogon_ctl help press' for more information)
```

### version子命令

version子命令用于打印目前安装于系统的drogon版本，如下：

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

### create子命令

create子命令用于创建各种对象，目前是drogon_ctl的主要功能，使用`dg_ctl help create`命令可以打印该命令的详细帮助，如下：

```console
$ dg_ctl help create
Use create command to create some source files of drogon webapp

Usage:drogon_ctl create <view|controller|filter|project|model> [-options] <object name>

drogon_ctl create view <csp file name> //create HttpView source files from csp file

drogon_ctl create controller [-s] <[namespace::]class_name> //create HttpSimpleController source files

drogon_ctl create controller -h <[namespace::]class_name> //create HttpController source files

drogon_ctl create controller -w <[namespace::]class_name> //create WebSocketController source files

drogon_ctl create filter <[namespace::]class_name> //create a filter named class_name

drogon_ctl create project <project_name> //create a project named project_name

drogon_ctl create model <model_path> //create model classes in model_path
```

* #### 视图创建

  `dg_ctl create view`命令用于从csp文件生成源文件，参见[视图](/CHN/CHN-06-视图)一节。一般情况下，该命令不需要直接使用，又cmake文件配置成自动执行是更好的方法。命令例子如下，假设csp文件是`UsersList.csp`.

  ```shell
  dg_ctl create view UsersList.csp
  ```

* #### 控制器创建

  `dg_ctl create controller`命令用于帮助用户创建控制器的源文件，drogon目前支持的三种控制器都可以由该命令创建，只是参数稍有差别。

  * 创建HttpSimpleController的命令如下：

  ```shell
  dg_ctl create controller SimpleControllerTest
  dg_ctl create controller webapp::v1::SimpleControllerTest
  ```

  最后一个参数是控制器的类名，可以在前面附加命名空间。

  * 创建HttpController的命令如下：

  ```shell
  dg_ctl create controller -h ControllerTest
  dg_ctl create controller -h api::v1::ControllerTest
  ```

  * 创建WebSocketController的命令如下：

  ```shell
  dg_ctl create controller -w WsControllerTest
  dg_ctl create controller -w api::v1::WsControllerTest
  ```

  参考wiki的控制器各个章节。

* #### 过滤器创建

  `dg_ctl create filter`命令用于帮助用户创建过滤器的源文件，参见[中间件和过滤器](/CHN/CHN-05-中间件和过滤器)一节。

  ```shell
  dg_ctl create filter LoginFilter
  dg_ctl create filter webapp::v1::LoginFilter
  ```

* #### 创建工程

  用户创建一个新的Drogon应用工程的方法是通过drogon_ctl命令实现，如下：

  ```shell
  dg_ctl create project ProjectName
  ```

  该命令执行完后，在当前目录会创建完整的工程目录，目录名是`ProjectName`,用户可以直接进build目录编译这个工程，当然他没有任何业务逻辑。

  工程的目录结构如下：

  ```console
  ├── build                         构建文件夹
  ├── CMakeLists.txt                工程的cmake配置文件
  ├── cmake_modules                 第三方库查找的cmake脚本
  │   ├── FindJsoncpp.cmake
  │   ├── FindMySQL.cmake
  │   ├── FindSQLite3.cmake
  │   └── FindUUID.cmake
  ├── config.json                   drogon应用的配置文件，请参考配置文件介绍章节
  ├── controllers                   存放控制器源文件的目录
  ├── filters                       存放过滤器文件的目录
  ├── main.cc                       主程序
  ├── models                        数据库模型文件的目录，模型源文件创建见11.2.5
  │   └── model.json
  ├── tests                         存放测试程序的目录
  │   └── test_main.cc              测试主程序
  └── views                         存放视图csp文件的目录，源文件无需用户手动创建，工程编译时会自动预处理csp文件得到视图的源文件
  ```

* #### 创建模型

  使用`dg_ctl create model`命令创建数据库模型源文件，最后一个参数是模型存放的目录，该目录里必须包含一个名为`model.json`的模型配置文件，用于告诉dg_ctl如何连接数据库以及映射哪些表。

  比如，如果要在上面提到的工程目录里创建模型，请在工程目录下执行如下命令：

  ```shell
  dg_ctl create model models
  ```

  该命令会提示用户目录下已有的文件将被直接覆盖，用户输入`y`确认后就会生成所有模型文件。其他文件要引用模型类需要include模型的头文件，比如：

  ```c++
  #include "models/User.h"
  ```

  注意要包含models目录名，这是为了区分同一个工程中多个数据源的情况。参见[ORM](/CHN/CHN-08-3-数据库-ORM)。

### 压力测试

用户可以使用`dg_ctl press`命令进行压力测试，该命令有几个选项

* `-n num` 设置请求的总数(默认是 1)
* `-t num` 设置线程总数(默认是 1), 设置成 CPU 数目可以达到最大性能
* `-c num` 设置并发连接数(默认是 1)
* `-q` 表示没有中间过程的信息输出(默认输出信息)

例子：

```shell
dg_ctl press -n1000000 -t4 -c1000 -q http://localhost:8080/
dg_ctl press -n 1000000 -t 4 -c 1000 https://www.domain.com/path/to/be/tested
```

# 12 [AOP 面向切面编程](/CHN/CHN-12-AOP面向切面编程)
