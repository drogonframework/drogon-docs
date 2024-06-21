[English](CHS-FAQ-1-Understanding-drogon-threading-model.md) | [简体中文](/CHN/CHN-FAQ-1-线程模型.md)

# 理解Drogon's的线程模型

drogon是一个快速的C++ Web应用程序框架，部分原因是没有抽象化底层线程模型并把它们包裹起来。 然而这也常引发一些用户的疑惑。 社群中经常会看到一些问题和讨论，为什么响应只在一些阻塞调用之后发送、为什么在同一个事件循环块上调用阻塞网络函数会导致死锁等等。本文的目的在解释导致它们的确切条件和如何避免它们。

## 事件循环和线程

Drogon在线程池上运行，其中每个线程都有自己的事件循环。事件循环是Drogon的核心。且每个drogon应用至少有2个事件循环。一个主循环和一个工作循环。一般来说， 主循环总是在主线程（启动`main`的线程）上运行。它负责启动所有工作循环。以hello world为例。 `app().run()` 在主线程上启动主循环。这进而产生3个工作线程/循环。

```cpp
#include <drogon/drogon.h>
using namespace drogon;

int main()
{
    app().registerHandler("/", [](const HttpRequest& req
        , std::function<void (const HttpResponsePtr &)> &&callback) {
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody("Hello wrold");
        callback(resp);
    });
    app().addListener("0.0.0.0", 80800);
    app().setNumThreads(3);
    app().run();
}
```

线程结构看起来像这样

```
            .-------------.
            | app().run() | <main thread>
            :-------------:
                  |
            .-----v------.
            | MAIN LOOP  |
            :------------:
                  | Spawns
      .-----------+--------------.
      |           |              |
.-----v----.   .--v-------.  .---v----.
| Worker 1 |   | Worker 2 |  | etc... |
:----------:   :----------:  :--------:
 <thread 1>     <thread 2>   <thread ...>
```

工作循环的数量取决于许多变量，包括为HTTP服务器指定了多少线程、创建了多少非快速DB和NoSQL连接等等，稍后再讨论快速连接与非快速连接，重要的是drogon而不仅仅有HTTP服务器线程。每个事件循环本质上都是一个任务队列，它主要处理如下几种事情：

1. 在事件循环的线程上读取任务队列里的任务并执行它，您可以从任何其他线程提交任务，任务的提交和执行是完全无锁的（感谢无锁数据结构）并且在所有情况下都不会导致数据竞争。事件循环会按顺序一个一个地处理任务。因此，任务具有明确定义的执行顺序。但是，在一个巨大的、长时间运行的任务之后排队的任务也会被延迟；
2. 当被该事件循环管理的网络资源上有任何注册的事件发生时，事件循环会调用对应的处理程序对该事件进行处理；
3. 当该事件循环管理的任何定时器到时时，事件循环会调用对应的定时器处理函数（通常由定时器的创建者提供）；
   当上述任何事件都没有发生时，事件循环/线程处于阻塞挂起的状态。

下面看一个例子：

```cpp
// queuing two tasks on the main loop
trantor::EventLoop* loop = app().getLoop();
loop->queueInLoop([]{
    std::cout << "task1: I'm gonna wait for 5s\n";
    std::this_thread::sleep_for(5s);
    std::cout << "task1: hello!\n";
});
loop->queueInLoop([]{
    std::cout << "task2: world!\n";
});
```

希望现在你能理解为什么运行上面的代码片段会导致`task1: I'm going wait for 5s`立即出现。暂停5秒钟，然后`task1: hello`和`task2: world!`再出现。

> **重点1：不要在事件循环中调用阻塞IO。这会导致其他任务必须等待该IO。**

## 网络IO

drogon中的几乎所有内容都与事件循环相关联。这包括TCP、HTTP客户端、数据库客户端和数据缓存。为避免竞争条件，所有IO都在关联的事件循环中完成。如果IO调用是从另一个线程进行的，则参数将被存储并作为任务提交给适当的事件循环。这有一些含意。例如，当从HTTP程序中进行数据库调用时。来自客户端的回调可能不一定（实际上，通常不会）与原始程序在同一线程上运行。

```cpp
app().registerHandler("/send_req", [](const HttpRequest& req
    , std::function<void (const HttpResponsePtr &)> &&callback) {
    // This handler will run on one of the HTTP server threads

    // Create a HTTP client that runs on the main loop
    auto client = HttpClient::newHttpClient("https://drogon.org", app().getLoop());
    auto request = HttpRequest::newHttpRequest();
    client->sendRequest(request, [](ReqResult result, const HttpResponse& resp) {
        // This callback runs on the main thread
    });
});
```

因此，如果您不知道您的代码实际在做什么，可能会阻塞事件循环。例如在主循环上创建大量HTTP客户端并发送所有传出请求， 或者在数据库回调中运行计算量大的函数。延后的其他线程请求的数据库查询。

```
 Worker 1       Main Loop        Worker2
.---------.    .----------.     .---------.
|         |    |          |     |         |
| req 1-. |    |----------|  .--+--req 2  |
|       :-+----+->        |  |  |         |
|         |    | send http|  |  |---------|
|---------| a.-|   req 1  |  |  |         |
|other req| s| |----------|  |  |         |
|---------| y| |      <---+--:  |         |
|         | n| |send http |     |         |
|         | c| |  req 2   |-.   |         |
|         |  | |----------| |a  |---------|
|         |  | |http resp1| |s  |other req|
|         |  :-|>compute  | |y  |---------|
|         |    |          | |n  |         |
|         | .--+-generate | |c  |         |
|         | |  | response | |   |         |
|         | |  |----------| |   |         |
|         | |  |http resp2|<:   |         |
|---------| |  | compute  |     |---------|
|response<|-:  |          |-----|>        |
|send back|    | generate |     |send resp|
|         |    | response |     | back    |
:---------:    :----------:     :---------:
```

同样的原理也适用于HTTP服务器。如果响应是从另外的线程生成的（例如：在DB回调中）。响应会在关联线程上排队等待发送而不是立即发送。

> **重点2：注意大量计算的函数。如果不小心，它们也会影响吞吐量。**

## 事件循环死锁

了解Drogon的设计方式后。不难看出如何使事件循环死锁。您只需提交一个远端IO请求并在同一个循环中等待它。事实上同步接口内部就是如此运最。它提交一个IO请求并等待回调（使用者要注意不能在当前循环中使用同步接口）。

```cpp
app().registerHandler("/dead_lock", [](const HttpRequest& req
    , std::function<void (const HttpResponsePtr &)> &&callback) {
    auto currentLoop = app().getIOLoops()[app().getCurrentThreadIndex()];
    auto client = HttpClient::newHttpClient("https://drogon.org", currentLoop);
    auto request = HttpRequest::newHttpRequest();
    auto resp = client->sendRequest(resp); // DEADLOCK! calling the sync interface
});
```

可以将其可视化为

```
     Some loop
   .------------.
   | new client |
   | new request|
   |send request|
 .->WAIT resp---+-.
 | |  ....      | |
?| |            | |
?| |------------| |
?| |            | |
?| |            | |
?| |            | |
?| |            | |
 | |            | |
 | |            | |
 | |            | |
 | |            | |
 | |            | |
 | |            | |
 | |------------| |
 | | read resp  | |
 :-+-         <-+-:
   | oops       |
   | deadlock   |
   :------------:
```

其他功能也是如此。数据库、NoSQL，你能想到的。幸运的是，非快速数据库客户端在它们自己的线程上运行； 每个客户端都有自己的线程。因此，从HTTP处理程序进行同步数据库查询是安全的。但是，**您不应在同一客户端的回调中运行同步数据库查询。否则同样的事情会发生**。

> **重点3：同步API对性能和安全都是不利的，请避开它们。如果必须，请确保在单独的线程上运行客户端。**

### 高速数据库客户端

Drogon是为性能而设计的，其次是易用性。快速数据库客户端共享HTTP服务器线程。这消除向另一个线程提交请求的需要来提高性能，并避免互斥锁和操作系统的上下文切换。然而由于共享线程。您不能对它们使用同步接口。这会使事件循环死锁。

## 使用协程

Drogon开发和使用中一个困境是，异步API更有效但使用起来很烦人。虽然同步API可能存在问题且速度缓慢，但是它们很容易编程。 Lambda声明可能冗长， 而且语法并不优雅，代码不会从上到下运行，而是充满了回调以及各种嵌套；与此相对，同步API比异步更干净，但性能差很多（它会使线程经常处于等待状态从而降低吞吐量）。 下面是异步编排和同步编排的对比：

- 异步：

  ```cpp
  // drogon's async DB API
  auto db = app().getDbClient();
  db->execSqlAsync("INSERT......", [db, callback](auto result){
      db->execSqlAsync("UPDATE .......", [callback](auto result){
          // Handle success
      },
      [callback](const DbException& e) {
          // handle failure
      })
  },
  [callback](const DbException& e){
      // handle failure
  })
  ```

- 同步：

  ```cpp
  // drogon's sync API. Exception can be handled automatically by the framework
  db->execSqlSync("INSERT.....");
  db->execSqlSync("UPDATE.....");
  ```

一定有办法一石二鸟吧？ C++20的协程就是我们的解决之道，本质上，协程是被编译器支持的回调包装器，使您的代码看起来像是同步的，但实际上一直都是异步的。下面是使用协程的相同的代码：

```cpp
co_await db->execSqlCoro("INSERT.....");
co_await db->execSqlCoro("UPDATE.....");
```

它与同步API的形式完全一样！但几乎在各个方面都更好。可以获得异步的所有好处，但继续使用类似同步的接口。他的原理超出本文的范围。 drogon维护者建议尽可能使用协程（GCC >= 11. MSVC >= 16.25）。然而，它并不是万能魔法，它不会解决阻塞事件循环和竞争条件，但使用协程更容易调试和理解异步代码。

> **重点4：尽可能使用协程**

## 总结

- 尽可能使用C++20协程和`快速数据库`连接
- 同步API可能减慢速度或导致事件循环死锁
- 如果您必须使用同步API。确保它们运行在跟当前程序不同的线程上
