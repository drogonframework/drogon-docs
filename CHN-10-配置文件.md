你可以通过DrogonAppFramework实例的多个接口配置各种参数来控制Http服务端的某些行为。不过，使用配置文件是更好的方式，原因如下：

* 使用配置文件而不是源码，可以在运行期而不是编译期决定应用的行为，这无疑是更方便和灵活的方式；
* 可以保持main文件的简洁；

所有的配置接口都有对应的配置文件选项支持，基于上面这些额外的好处，建议应用开发者使用配置文件配置应用的各种参数。

配置文件的加载很简单，在DrogonAppFramework实例调用run接口之前，调用loadConfigFile接口即可，参数是配置文件的路径和文件名，比如：

```c++
int main()
{
    drogon::app().loadConfigFile("config.json");
    drogon::app().run();
}
```

这段程序，加载配置文件`config.json`，然后运行。具体的监听端口、日志输出、数据库配置等等行为都可以由配置文件配置，事实上，这段程序基本就可以是整个webapp应用的主函数的全部代码了。

### 文件说明

配置文件的例子在源码目录的顶层`config.example.json`，如果你使用drogon_ctl create project命令创建工程，那么，在工程目录里也可以找到内容一致的文件`config.json`。所以，你基本上不需要重写，而是对这个文件进行适当的修改，就可以完成对webapp的配置。

文件是json格式，支持注释，你可以用c++的注释符号`/**/`和`//`把不需要的配置项注释掉。

注释掉配置项后，框架会使用默认值初始化，对应项的默认值在配置文件中都有说明，下面分项详述。

### SSL

ssl项是为了配置https服务的加密配置文件，如下：
```json
  "ssl": {
    "cert": "../../trantor/trantor/tests/server.pem",
    "key": "../../trantor/trantor/tests/server.pem",
    "conf":[
      ["Options", "Compression"],
      ["min_protocol", "TLSv1.2"]
    ]
  }
```

其中，cert是证书文件路径，key是私钥文件路径，如果一个文件既包含证书也包含私钥，则两个路径可以配制成相同的。conf则是可选的SSL选项。他们会直接被传入[SSL_CONF_cmd](https://www.openssl.org/docs/manmaster/man3/SSL_CONF_cmd.html)裡，用来客製化SSL连线。这些选项必须是一个或是两个元素的字串数组。

文件的编码格式PEM。

### listeners监听器

顾名思义，listeners项是为了配置webapp的监听器，它是一个JSON数组类型，每一个JSON对象都表示一个监听，具体的配置如下：

```json
"listeners": [
    {
      "address": "0.0.0.0",
      "port": 80,
      "https": false
    },
    {
      "address": "0.0.0.0",
      "port": 443,
      "https": true,
      "cert": "",
      "key": ""
    }
  ]
```

其中：

* `address`: 字符串类型，表示监听的IP地址，如果没有该项，则采用默认值"0.0.0.0"
* `port`: 整数类型，表示监听的端口，必须是合法的端口号，没有默认值，必填项。
* `https`: 布尔类型，表示是否采用https，默认值是false，表示使用http。
* `cert`和`key`: 字符串类型，在https为true时有效，表示https的证书和私钥，默认值是空字符串，表示采用全局的`ssl`配置的证书和私钥文件；

### db_clients

用于配置数据库客户端，它是一个JSON数组类型，每一个JSON对象都表示一个单独的数据库客户端，具体的配置如下：

```json
  "db_clients":[
    {
      "name":"",
      "rdbms": "postgresql",
      "host": "127.0.0.1",
      "port": 5432,
      "dbname": "test",
      "user": "",
      "passwd": "",
      "is_fast": false,
      "connection_number":1,
      "filename": ""
    }
  ]
```

其中：

* `name`：字符串，客户端名字，默认是"default"，name是应用开发者从框架获取数据库客户端的标记，如果有多项客户端，name字段必须不同，否则，框架不会正常工作；
* `rdbms`：字符串，表示数据库服务端类型，目前支持"postgresql"，"mysql"和“sqlite3”，大小写不敏感。
* `host`：字符串，数据库服务端地址，localhost是默认值；
* `port`：整数，数据库服务端的端口号；
* `dbname`：字符串，数据库名字；
* `user`：字符串，用户名；
* `passwd`：字符串，密码；
* `is_fast`：bool，默认false，表明该客户端是否是[FastDbClient](CHN-08-4-数据库-FastDbClient)；
* `connection_number`：到数据库服务端的连接数，至少是1，默认值也是1，影响数据读写的并发量；如果`is_fast`为真，该数值表示每个事件循环的连接数，否则表示总的连接数；
* `filename`: sqlite3数据库的文件名；

### threads_num线程数

属于app选项的子项，整数，默认值是1，表示框架IO线程数，对网络并发有明确的影响，这个数字并不是越大越好，了解non-blocking I/O原理的应该知道，这个数值应该和你期望网络IO占用的CPU核心数一致。如果这个参数设为0，那么IO线程数将设置为全部CPU核心数。比如：

```json
"threads_num": 16,
```

表示，网络IO占用16个线程，在高负荷情况下，最多可以跑满16个CPU核心(线程核心)。

### session会话

会话相关的选项也是app的子项，控制是否采用session和session的超时时间。如：

```json
"enable_session": true,
"session_timeout": 1200,
```

其中：
  
* `enable_session`：布尔值，表示是否采用会话，默认值是false，如果客户端不支持cookie，请设为false，因为框架会为每个不带会话cookie的请求创建新的会话，如果客户端不支持cookie而又有大量请求，则服务端会生成大量的无用session对象，这是完全没必要的资源和性能损失；
* `session_timeout`：整数值，表示会话的超时时间，单位是秒，默认值是0，只有enable_session为true时才发挥作用。0表示永久有效。

### document_root根目录

app项的子项，字符串，表示Http根目录对应的文档路径，是静态文件下载的根路径，默认值是"./"，表示程序运行的当前路径。比如：

```json
"document_root": "./",
```

### upload_path上传文件路径

app项的子项，字符串，表示上传文件的默认路径，默认值是"uploads"，如果这个值不是`/`,`./`或`../`开始的，并且这个值也不是`.`或`..`，则这个路径是前面document_root项的相对路径，否则就是一个绝对路径或者当前目录的相对路径。如：

```json
"upload_path":"uploads",
```

#### file_types文件类型

app项的子项，字符串数组，默认值如下，表示框架支持的静态文件下载类型，如果请求的静态文件扩展名在这些类型之外的，框架将返回404错误。

```json
"file_types": [
      "gif",
      "png",
      "jpg",
      "js",
      "css",
      "html",
      "ico",
      "swf",
      "xap",
      "apk",
      "cur",
      "xml"
    ],
```

### connections连接数控制

app项的子项，有两个选项，如下：

```json
 "max_connections": 100000,
 "max_connections_per_ip": 0,
```

其中：

* `max_connections`：整数，默认值是100000，表示同时并发的最大连接数；当服务端维持的连接达到这个数量时，新的TCP连接请求将被直接拒绝。
* `max_connections_per_ip`：整数，默认值是0，表示单个IP的最大连接数，0表示没有限制。

### log日志选项

app项的子项，同时也是个JSON对象，控制日志输出的行为，如下：

```json
 "log": {
      "log_path": "./",
      "logfile_base_name": "",
      "log_size_limit": 100000000,
      "log_level": "TRACE"
    },
```

其中：

* `log_path`：字符串，默认值是空串，表示文件存放的路径，如果是空串，则所有日志输出到标准输出；
* `logfile_base_name`：字符串，表示日志文件的`basename`，默认值是空串，这时basename将为drogon；
* `log_size_limit`：整数，单位是字节，默认值是100000000(100M)，当日志文件的大小达到这个数值时，日志文件会切换。
* `log_level`：字符串，默认值是"DEBUG"，表示日志输出的最低级别，可选值从低到高为："TRACE","DEBUG","INFO","WARN"，其中的TRACE级别只有在DEBUG编译的情况下才有效。

**注意： Drogon的文件日志采用了非阻塞输出结构，大概可以达到每秒百万行的日志输出能力，可以放心使用。**

### 应用控制

也是app子项，有两个控制项，如下：

```json
    "run_as_daemon": false,
    "relaunch_on_error": false,
```

其中：

* `run_as_daemon`：布尔值，默认值是false，当为true时，应用程序将以daemon的形式变成1号进程的子进程运行于系统后台。
* `relaunch_on_error`：布尔值，默认值时false，当为true时，应用程序将fork一次，子进程执行真正的工作，父进程什么都不干，只负责在子进程崩溃或因其它原因退出时重启子进程，这是一种简单的服务保护机制。

### use_sendfile发送文件

app子选项，布尔值，表示在发送文件时是否采用linux系统调用sendfile，默认值时true，使用sendfile可以提高发送效率，减少大文件的内存占用。如下：

```json
"use_sendfile": true,
```

**注意：即使该项为true，sendfile系统调用也不一定会采用，因为小文件使用sendfile并不一定划算，框架会根据自己的优化策略决定是否采用。**

### use_gzip压缩传输

app子选项，布尔值，默认值是true，表示Http响应的Body是否采用压缩传输。当该项为true时，下面的情况采用压缩传输：

* 客户端支持gzip压缩；
* body是文本类型；
* body的长度大于一定值；

配置例子如下：

```json
"use_gzip": true,
```

### files_cache_time文件缓存时间

app子选项，整数值，单位秒，表示静态文件在框架内部的缓存时间，即，在这段时间内对该文件的请求，框架会直接从内存返回响应而不会读取文件系统，这对提高静态文件性能有一定好处，当然，代价是更新实时性有n秒的延时。默认值是5秒，0表示永远缓存（只会读文件系统一次，慎用），负数表示没有缓存。如下：

```json
 "static_files_cache_time": 5,
```

### simple_controllers_map

app子选项，JSON对象数组，每一项表示一个从Http路径到HttpSimpleController的映射，这种配置只是一个可选途径，并不是必须配置在这里，请参阅[HttpSimpleController](CHN-04-1-控制器-HttpSimpleController)。
具体的配置如下：

```json
"simple_controllers_map": [
      {
        "path": "/path/name",
        "controller": "controllerClassName",
        "http_methods": ["get","post"],
        "filters": ["FilterClassName"]
      }
    ],
```

其中：

* `path`：字符串，Http路径；
* `controller`：字符串，HttpSimpleController的名字；
* `http_methods`：字符串数组，支持的Http方法，这个列表之外的会被过滤掉，返回405错误；
* `filters`：字符串数组，路径上的filter列表，参见[过滤器](CHN-05-过滤器)；

### idle_connection_timeout空闲连接超时控制

app子选项，整数值，单位秒，默认值是60，当一个连接超过这个数值的时间没有任何读写的时候，该连接将会被强制断开。如下：

```json
"idle_connection_timeout":60
```

### dynamic_views动态视图加载

app的子选项，控制动态视图的使能和路径，有两个选项，如下：

```json
"load_dynamic_views":true,
"dynamic_views_path":["./views"],
```

其中：

* `dynamic_views_path`：布尔值，默认值是false，当为true时，框架会在视图路径中搜索视图文件，并动态编译成so文件，然后加载进应用，当任何视图文件发生变化时，也会引起自动编译和重新加载；
* `dynamic_views_path`：字符串数组，每一项表示动态视图的搜索路径，如果路径值不是`/`,`./`或`../`开始的，并且这个值也不是`.`或`..`，则这个路径是前面document_root项的相对路径，否则就是一个绝对路径或者当前目录的相对路径。

参见[视图](CHN-06-视图)。


### server_header_field头字段

app的子选项，配置由框架发送的所有response的Server头字段，默认值是空串，当该选项为空时，框架会自动生成形如`Server: drogon/version string`的头字段。如下：

```json
"server_header_field": ""
```


### keepalive_requests长连接请求数

`keepalive_requests` 选项设置客户端在一个keepalive长连接上可以发送的最大请求数。当达到这个请求数时，长连接将被关闭。默认值0代表没有限制。如下：
```json
"keepalive_requests": 0
```

### Pipelining请求数

`pipelining_requests`选项用于设置长连接上已接收但未必处理的最大的请求数。当这个数字达到时，长连接将被关闭。默认值0代表没有限制。关于pipelining的详细描述，请参阅标准文档`rfc2616-8.1.1.2`。配置如下所示：

```json
"pipelining_requests": 0
```

# 11 [drogon_ctl命令](CHN-11-drogon_ctl命令)