[English](ENG-18-Testing-Framework) | [简体中文](CHN-18-测试框架)

DrogonTest是一个内置在Drogon中的最小测试框架，可实现简单的异步测试和同步测试。 它用于Drogon自己的单元测试和集成测试。 但也可用于测试使用Drogon构建的应用程序。 DrogonTest的语法受到[GTest](https://github.com/google/googletest)和[Catch2](https://github.com/catchorg/Catch2)的启发。

您不必为应用程序使用DrgonTest。您可以使用喜欢的任何东西。但它是一个选择。

### 基本测试

让我们从一个简单的例子开始。有一个函数，可以计算至某数为止之自然数之和，并想测试它的正确性。

```c++
// Tell DrogonTest to generate `test::run()`. Only defined this in the main file
#define DROGON_TEST_MAIN
#include <drogon/drogon_test.h>

int sum_all(int n)
{
    int result = 1;
    for(int i=2;i<n;i++) result += i;
    return result;
}

DROGON_TEST(Sum)
{
    CHECK(sum_all(1) == 1);
    CHECK(sum_all(2) == 3);
    CHECK(sum_all(3) == 6);
}

int main(int argc, char** argv)
{
    return drogon::test::run(argc, argv);
}
```

编译并运行...好吧，它通过了，但有一个明显的错误。 `sum_all(0)` 应该是0。我们可以将它添加到我们的测试中

```c++
DROGON_TEST(Sum)
{
    CHECK(sum_all(0) == 0);
    CHECK(sum_all(1) == 1);
    CHECK(sum_all(2) == 3);
    CHECK(sum_all(3) == 6);
}
```

现在测试失败了：

```
In test case Sum
↳ /path/to/your/test/main.cc:47  FAILED:
  CHECK(sum_all(0) == 0)
With expansion
  1 == 0
```

注意到框架在表达式的两端打印了实际值。 让我们可以立即看到发生了什么。 解决方法很简单：

```c++
int sum_all(int n)
{
    int result = 0;
    for(int i=1;i<n;i++) result += i;
    return result;
}
```

### 测试的类型

DrogonTest带有各种类型的测试和操作。 基本的 `CHECK()` 只检查表达式的计算结果是否为真。 如果没有，它会打印到控制台。 `CHECK_THROWS()` 检查表达式是否抛出异常。诸如此类。另一方面`REQUIRE()`检查表达式是否为真。 如果没有则从函数返回。

| 失败后/表达式 | 为真       | 抛出异常          | 没有 抛出异常      | 抛出特定异常         |
| ------------- | ---------- | ----------------- | ------------------ | -------------------- |
| 不做任何事    | CHECK      | CHECK_THROWS      | CHECK_NOTHROW      | CHECK_THROWS_AS      |
| 返回          | REQUIRE    | REQUIRE_THROWS    | REQUIRE_NOTHROW    | REQUIRE_THROWS_AS    |
| 自协程返回    | CO_REQUIRE | CO_REQUIRE_THROWS | CO_REQUIRE_NOTHROW | CO_REQUIRE_THROWS_AS |
| 杀死进程      | MANDATE    | MANDATE_THROWS    | MANDATE_NOTHROW    | MANDATE_THROWS_AS    |

让我们尝试一个稍微实际的例子。 假设我们正在测试文件的内容是否符合预期。 若程序无法打开文件则没有必要进一步检查。 因此，我们可以使用REQUIRE来缩短和减少重复代码。

```c++
DROGON_TEST(TestContent)
{
    std::ifstream in("data.txt");
    REQUIRE(in.is_open());
    // Instead of
    // CHECK(in.is_open() == true);
    // if(in.is_open() == false)
    //    return;

    ...
}
```

同样，`CO_REQUIRE` 就像REQUIRE。 但是用于协程。 当操作修改且不可恢复的全局状态时，可以使用`MANDATE`。 唯一合乎逻辑的做法是停止测试。

### 异步测试

Drogon是一个异步网站框架。 它仅遵循因此DrogonTest支持测试异步。 DrogonTest通过`TEST_CTX`变量跟踪测试上下文。 只需按**值**捕获变量。 例如，测试远程API是否成功并返回JSON：

```c++
DROGON_TEST(RemoteAPITest)
{
    auto client = HttpClient::newHttpClient("http://localhost:8848");
    auto req = HttpRequest::newHttpRequest();
    req->setPath("/");
    client->sendRequest(req, [TEST_CTX](ReqResult res, const HttpResponsePtr& resp) {
        // There's nothing we can do if the request didn't reach the server
        // or the server generated garbage.
        REQUIRE(res == ReqResult::Ok);
        REQUIRE(resp != nullptr);

        CHECK(resp->getStatusCode() == k200OK);
        CHECK(resp->contentType() == CT_APPLICATION_JSON);
    });
}
```

由于测试框架支持C++14/17兼容性，协程必须包装在`AsyncTask`中或通过`sync_wait`调用。

```c++
DROGON_TEST(RemoteAPITestCoro)
{
    auto api_test = [TEST_CTX]() {
        auto client = HttpClient::newHttpClient("http://localhost:8848");
        auto req = HttpRequest::newHttpRequest();
        req->setPath("/");

        auto resp = co_await client->sendRequestCoro(req);
        CO_REQUIRE(resp != nullptr);
        CHECK(resp->getStatusCode() == k200OK);
        CHECK(resp->contentType() == CT_APPLICATION_JSON);
    };

    sync_wait(api_test());
}
```

### 启动 Drogon 的事件循环

一些测试需要Drogon的事件循环运行。 例如，除非特别指定，否则HTTP客户端运行在Drogon的全局事件循环上。 以下样板处理了许多边缘情况，并确保事件循环在任何测试开始之前运行：

```c++
int main()
{
    std::promise<void> p1;
    std::future<void> f1 = p1.get_future();

    // Start the main loop on another thread
    std::thread thr([&]() {
        // Queues the promise to be fulfilled after starting the loop
        app().getLoop()->queueInLoop([&p1]() { p1.set_value(); });
        app().run();
    });

    // The future is only satisfied after the event loop started
    f1.get();
    int status = test::run(argc, argv);

    // Ask the event loop to shutdown and wait
    app().getLoop()->queueInLoop([]() { app().quit(); });
    thr.join();
    return status;
}
```

### CMake 集成

与大多数测试框架一样，DrgonTest可以将自身集成到CMake中。 `ParseAndAddDrogonTests` 函数将它在源文件中看到的测试添加到CMake的CTest框架中。

```cmake
include(ParseAndAddDrogonTest) # Also loads ParseAndAddDrogonTests
add_executable(mytest main.cpp)
target_link_libraries(mytest PRIVATE Drogon::Drogon)
ParseAndAddDrogonTests(mytest)
```

现在可以通过构建系统（在本例中为 Makefile）运行测试。

```bash
❯ make test
Running tests...
Test project path/to/your/test/build/
      Start  1: Sum
 1/1  Test  #1: Sum ....................................   Passed    0.00 sec
```
