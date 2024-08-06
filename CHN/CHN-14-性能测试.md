[English](/ENG/ENG-14-Benchmarks)

## 性能测试

作为C++的Http应用框架，性能应该是关注的重点之一，本节介绍Drogon的简单测试和成绩；

### 测试环境

* 系统是Linux CentOS 7.4；
* 设备是Dell服务器，CPU是两颗Intel(R) Xeon(R) CPU E5-2670 @ 2.60GHz，16核32线程；
* 内存64GB；
* gcc版本7.3.0；

### 测试方案和结果

我们只是为了测试drogon框架的性能，因而要尽量简化controller的处理，我们只做了一个HttpSimpleController，注册到`/benchmark`路径上。controller对任何请求都返回`<p>Hello, world!</p>`。设置drogon线程数为16。handler函数的代码如下, 你可以在`drogon/examples/benchmark`目录找到这些源码：

```c++
void BenchmarkCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback)
{
    //write your application logic here
    auto resp = HttpResponse::newHttpResponse();
    resp->setBody("<p>Hello, world!</p>");
    resp->setExpiredTime(0);
    callback(resp);
}
```

作为对比，我选取了nginx来做对比测试，使用nginx+module源码编译的方式，写了一个hello_world_module，测试时nginx的worker_processes参数设为16。

测试工具是一个性能不错的HTTP压力测试工具`httpress`。

我们调整httpress的参数，每组参数测试五次，记录每秒处理请求数的最大值和最小值。测试结果如下表：

| 命令行                                       | 说明                                              | Drogon(千 QPS) | nginx(千 QPS) |
| :------------------------------------------- | :------------------------------------------------ | :------------: | :-----------: |
| httpress -c 100 -n 1000000 -t 16 -k -q URL   | 100 连接，100 万请求，16 线程，Keep-Alive         |    561/552     |    330/329    |
| httpress -c 100 -n 1000000 -t 12 -q URL      | 100 连接，100 万请求，12 线程，一次请求一次连接   |    140/135     |     31/49     |
| httpress -c 1000 -n 1000000 -t 16 -k -q URL  | 1000 连接，100 万请求，16 线程，Keep-Alive        |    573/565     |    333/327    |
| httpress -c 1000 -n 1000000 -t 16 -q URL     | 1000 连接，100 万请求，16 线程，一次请求一次连接  |    155/143     |     52/50     |
| httpress -c 10000 -n 4000000 -t 16 -k -q URL | 10000 连接，400 万请求，16 线程，Keep-Alive       |    512/508     |    316/314    |
| httpress -c 10000 -n 1000000 -t 16 -q URL    | 10000 连接，100 万请求，16 线程，一次请求一次连接 |    143/141     |     43/40     |

可以看到，在客户端使用Keep-Alive选项，在一个连接可以发送多个请求的情况下，drogon每秒可以处理50多万次请求，这个成绩是相当不错的。每次请求都发起一次连接的情况下，CPU会消耗在TCP建立和断开等环节，吞吐量会下降至每秒14万次请求，这也是正常的。drogon对比nginx的成绩处于明显优势，也许是由于nginx配置不当未能发挥它的最大吞吐量，如果哪位高手做了更好的测试，欢迎指正。

下图是某一次测试的截图：

![测试截图](images/benchmark.png)

# 下一个: [Coz 分析](/CHN/CHN-15-Coz分析)
