## Coroutine

Drogon supports [C++ coroutines][1] starting from version 1.4. They provide a way to flatten the control flow of asyncrynous calls, i.e. escaping the callback hell. With it, asyncrynois programming becomes as easy as syncrinous ones.

### Terminology

This page isn't intended to explain what is a coroutine nor how it works. But to show how to use coroutines in drogon. The usual vocabulary tends get messy as subroutines (functions) uses the same terminology as coroutine does, yet they have slightly different meanings. C++ coroutines could act as they are functions doesn't help either. To reduce confusion, these are the termiology we'll use - they are by no means perfect, but good enough.

**Coroutine** is a function that can suspend execution then resume.<br/> 
**Return** means a function finishing execution and gives a return value to it caller. Or a coroutine generating an _resumable_ object. Which can be used to resume the coroutine.<br/>
**Yield**ing is when a coroutine generates a result for the caller.<br/>
**co-return** means a coroutine yields and then exit.<br/>
**(co-)await**ing means the thread is waiting a coroutine to yield. The framework is free to use the thread for other purpose while awaiting.<br/>

### Enabling coroutines

The coroutine feature in drogon is header only. So the application can use coroutines even if drogon is built without coroutine support. How to enable coroutines depends on the compiler used. GCC >= 8 enables it by setting `-std=c++20 -fcoroutines` while MSVC (tested on MSVC 19.25) does by `/std:c++latest` and `/await` must not be set.

Note that drogon's implementation of coroutines won't work on clang (as of clang 12.0). And GCC 11 enables coroutines by default when C++20 is enabled.

### Using coroutines

Each and every coroutine in drogon is suffixed with `Coro`. E,g. `db->execSqlSync()` becomes `db->execSqlCoro()`. `client->sendRequestSync()`  becomes `client->sendRequestCoro()`, so on and so forth. All coroutines return an _resumable_ object. Then `co_await` on the object results in a value. The framework is free to use the thread to process IO and tasks when it's awaiting results to arrive - that's the beauty of coroutines. The code looks syncrynous; but it's in fact asyncrynously.

For example, querying the number of users exists in the database:

```c++
app.registerHandler("/num_users",
    [](HttpRequestPtr req, std::function<void(const HttpResponsePtr&)> callback) -> Task<>
    //                                     Must mark the return type as an _resumable_ ^^^
{
    auto sql = app().getDbClient();
    auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
    size_t num_users = result[0][0].as<size_t>();

    auto resp = HttpResponse::newHttpResponse();
    resp->setBody(std::to_string(num_users));
    callback(resp);

    // No need to return anything! This is a coroutine that yields `void`. Which is
    // indicated by the return type of Task<void>
    co_return; // If want to (not required), use co_return
}
```

Notice a few important points:
 1. Any handler that calls a coroutine MUST return an _resumable_
    * Turning the handler itself into a coroutine
 2. `co_return` replaces `return` in a coroutine
 3. Most parameters are passed by value

An _resumable_ is an object following the coroutine standard. Don't worry too much about the detail. Just know that if you want the coroutine to yield something typed `T`. Then the return type will be `Task<T>`.

Passing most parameters by value is a direct consequence of coroutines being asynchronous. It's impossible to track when a reference goes out of scope as the object may destruct while the coroutine is waiting. Or the reference may live on another thread. Thus, may destruct while the coroutine is executing.

It makes sense to not have the callback but use the straightforward `co_return`. Which is supported, but may cause up to 8% of throughput under certain conditions. Please consider the performance drop and whether it's too great for the use case. Again, the same example:

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

