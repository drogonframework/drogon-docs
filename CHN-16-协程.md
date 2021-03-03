## 协程

从1.4版本开始，Drogon支持[C++ coroutines][1]（协程）。 它提供了扁平化异步调用控制流的方法, 比如，避免著名的`callback hell`. 通过协程, 异步编程将像同步编程一样简单（同时保持了异步程序的高性能）。

### 术语

本文无意于解释什么是协程或它是如何工作的，而是向大家介绍如何在Drogon中使用协程。有很多术语，普通的例程也使用，但是在协程里，意义稍有不同，为了避免引起不必要的混淆，我们列举了一些常用术语。

**协程（Coroutine）** 是能暂停执行以在之后恢复的函数.<br/> 
**Return** 对普通函数来说意味着结束执行并返回一个值。 而协程需要返回一个包含`promise_type`类型的对象，用来恢复这个协程的执行。<br/>
**(co_)yield**意思是协程暂停执行并返回一个值。<br/>
**co_return**意思是协程结束并返回一个值（如果有值的话）。<br/>
**(co_)await**意思是当前的协程正在等待一个结果，如果结果没有立即准备好，比如需要发起网络请求，则当前协程被暂停执行，当前线程将执行其它任务。当结果准备好时，当前协程将被恢复执行（不一定在当前线程恢复）<br/>

### 使能协程

协程特性在Drogon中是header-only的，这意味着即使构建drogon库的编译器不支持协程，用户也可以
使用协程。如何使能协程和编译器有关，对版本>=8.0的GCC来说，可以通过`-std=c++20 -fcoroutines`编译参数使能协程。对MSVC来说(MSVC 19.25测试通过)需要设置`/std:c++latest`并且不能设置`/await`。例如可以通过如下cmake命令使能drogon的协程(GCC)：

```shell
cmake .. -DCMAKE_CXX_FLAGS="-fcoroutines"
```

注意截至clang12.0, Drogon的协程实现还不能在clang上工作。 而GCC11在c++20标准开启时是默认支持协程的，也就是说，如果编译器是GCC11，则编译Drogon应用程序不需要做任何特别设置。

### 使用协程

协程的性能和内在逻辑和异步接口相当，不过它的接口确是同步形式的，所以基本上，Drogon中的每个协程函数（或者可以在协程中被co_await的函数）接口均相当于对应的同步接口改成了`Coro`后缀。 比如`db->execSqlSync()`对应于 `db->execSqlCoro()`，而`client->sendRequestSync()`对应于`client->sendRequestCoro()`，等等。所有上述函数均返回一个_awaitable_对象，`co_await`它将马上或者将来恢复协程时得到一个结果，等待结果的过程中，当前线程将被框架用于执行其它处理IO等操作，这就是协程的美妙之处，它的代码看起来是同步的，但实际上它是异步的。

比如，我们想返回数据库中用户的个数：

```c++
app.registerHandler("/num_users",
    [](HttpRequestPtr req, std::function<void(const HttpResponsePtr&)> callback) -> Task<>
    // 返回值必须是某种resumable类型（框架已封装好）。
{
    auto sql = app().getDbClient();
    auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
    size_t num_users = result[0][0].as<size_t>();

    auto resp = HttpResponse::newHttpResponse();
    resp->setBody(std::to_string(num_users));
    callback(resp);

    co_return; // 该语句不是必须的，因为它位于协程的结束处。因为返回值是Task<void>类型，这里不需要返回任何值
}
```

几个重要的需要注意的地方：
 1. 任何使用了co_await的handler方法，它自身就成为一个协程，它的返回值就不能是void类型了，必须更换成框架封装好的Task<T>模板。
 2. 普通函数中的`return`在协程中必须换成`co_return`。
 3. 协程的参数要用值传递。不能是引用。

An _awaitable_ is an object following the coroutine standard. Don't worry too much about the detail. Just know that if you want the coroutine to yield something typed `T`. Then the return type will be `Task<T>`.

Passing most parameters by value is a direct consequence of coroutines being asynchronous. It's impossible to track when a reference goes out of scope as the object may destruct while the coroutine is waiting. Or the reference may live on another thread. Thus, may destruct while the coroutine is executing.

It makes sense to not have the callback but use the straightforward `co_return`. Which is supported, but may cause up to 8% of throughput under certain conditions. Please consider the performance drop and weather if it's too great for the use case. Again, the same example:

```c++
app.registerHandler("/num_users",
    [](HttpRequestPtr req) -> Task<HttpResponsePtr>)
    //               Now returning a response ^^^
{
    auto sql = app().getDbClient();
    auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
    size_t num_users = result[0][0].as<size_t>();

    auto resp = HttpResponse::newHttpResponse();
    resp->setBody(std::to_string(num_users));
    co_return resp;
}
```

Calling coroutines from websocket controllers aren't supported yet. Feel free to open an issue if you need the feature.

[1]: https://en.cppreference.com/w/cpp/language/coroutines

