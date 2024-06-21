[English](ENG-01-Overview) | [简体中文](CHN-01-概述)

**Drogon**是一个基于C++17/20的Http应用框架，使用Drogon可以方便的使用C++构建各种类型的Web应用服务端程序。

Drogon的主要应用平台是Linux，也支持Mac OS、FreeBSD和Windows。

它的主要特点如下：

* 网络层使用基于epoll(macOS/FreeBSD下是kqueue)的非阻塞IO框架，提供高并发、高性能的网络IO。详细请见[TFB Tests Results](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite)；
* 全异步编程模式；
* 支持Http1.0/1.1(server端和client端)；
* 基于template实现了简单的反射机制，使主程序框架、控制器(controller)和视图(view)完全解耦；
* 支持cookies和内建的session；
* 支持后端渲染，把控制器生成的数据交给视图生成Html页面，视图由CSP模板文件描述，通过CSP标签把C++代码嵌入到Html页面，由drogon的命令行工具在编译阶段自动生成C++代码并编译；
* 支持运行期的视图页面动态加载(动态编译和加载so文件)；
* 非常方便灵活的路径(path)到控制器处理函数(handler)的映射方案；
* 支持过滤器(filter)链，方便在控制器之前执行统一的逻辑(如登录验证、Http Method约束验证等)；
* 支持https(基于OpenSSL实现);
* 支持websocket(server端和client端);
* 支持Json格式请求和应答, 对Restful API应用开发非常友好;
* 支持文件下载和上传,支持sendfile系统调用；
* 支持gzip/brotli压缩传输；
* 支持pipelining；
* 提供一个轻量的命令行工具drogon_ctl，帮助简化各种类的创建和视图代码的生成过程；
* 基于非阻塞IO实现的异步数据库读写，目前支持PostgreSQL和MySQL(MariaDB)数据库；
* 基于线程池实现sqlite3数据库的异步读写，提供与上文数据库相同的接口；
* 支持ARM架构；
* 方便的轻量级ORM实现，支持常规的对象到数据库的双向映射操作；
* 支持插件，可通过配置文件在加载期动态拆装；
* 支持内建插入点的AOP

# 02 [安装drogon](CHN-02-安装)
