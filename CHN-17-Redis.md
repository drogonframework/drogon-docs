Drogon 支持 Redis，Redis是一种非常快速的内存数据存储。 可以用作数据库缓存或消息代理。 与 Drogon 中其他组件一樣，Redis的操作是异步的。 这确保了 Drogon 即使在重负载下也能以非常高的并发性运行。 

Redis 支持依赖于hiredis 库。 如果在构建 Drogon 时hiredis 不可用，则Redis支持将不可用。 

### 创建客户端

Redis 客户端可以通过以下方式以方式创建：

```c++
app().createRedisClient("127.0.0.1", 6379);
...
// After app.run()
RedisClientPtr redisClient = app().getRedisClient();
``` 

另外，与Database客户端一样，Redis客户端也支持config文件配置，也支持配置成Fast模式，具体配置如下：

```json
    "redis_clients": [
        {
            //name: 客户端名字, 默认值是'default'
            //"name":"",
            //host: 服务端IP, 默认值是127.0.0.1
            "host": "127.0.0.1",
            //port: 服务端端口号, 默认值是6379
            "port": 6379,
            //passwd: 密码，默认为空
            "passwd": "",
            //db index: 默认值是0
            "db": 0,
            //is_fast: 默认值是false, 是否是fast模式，如果为true，会以更高效的方式运行，但是只能在IO线程或主线程中使用, 并且不能使用同步接口。
            "is_fast": false,
            //number_of_connections: 连接数, 默认值是1, 如果is_fast为true, 该数字表示每个IO线程或主线程内的连接数, 否则表示该客户端所有连接数
            "number_of_connections": 1,
            //timeout: 超时值，默认值是-1.0, 单位是秒，表示一条命令的超时时间，超过这个时间未得到结果将返回超时错误，0或者负值表示没有超时限制
            "timeout": -1.0
        }
    ]
```

### 使用Redis

execCommandAsync 以异步方式执行 Redis 命令。 它至少需要3个参数，第一个和第二个是在Redis命令成功或失败时调用的回调。 第三是命令本身。 该命令可以是C风格的格式字符串。 其余部分是格式字符串的参数。 例如，要设置 `name` 為 `drogon`： 

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {}
    [](const std::exception &err) {
        LOG_ERROR << "something failed!!! " << e.what();
    },
    "set name drogon");
```

或者将 `myid` 设置为 `587d-4709-86e4` 

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {}
    [](const std::exception &err) {
        LOG_ERROR << "something failed!!! " << e.what();
    },
    "set myid %s", "587d-4709-86e4");
```

同样的 execCommandAsync 也可以从 Redis 取得数据。 

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {
        if (r.type() == RedisResultType::kNull)
            LOG_INFO << "Cannot find variable associated with the key 'name'";
        else
            LOG_INFO << "Name is " << r.asString();
    }
    [](const std::exception &err) {
        LOG_ERROR << "something failed!!! " << e.what();
    },
    "get name");
```

### Redis 事务

Redis 事务允许在一个步骤中执行多个命令。 事务中的所有命令都按顺序执行，其他客户端的命令不会在事务**中间**执行。 注意redis的事务不是原子操作，也就是说收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。redis事务没有回滚操作。

newTransactionAsync 方法创建一个新事务。 然后就可以像普通的 RedisClient 一样使用事务。 最后，RedisTransaction::execute 方法执行事务。 

```c++
redisClient->newTransactionAsync([](const RedisTransactionPtr &transPtr) {
    transPtr->execCommandAsync(
        [](const drogon::nosql::RedisResult &r) { /* this command works */ }
        [](const std::exception &err) { /* this command failed */ },
    "set name drogon");

    transPtr->execute(
        [](const drogon::nosql::RedisResult &r) { /* transaction worked */ },
        [](const std::exception &err) { /* transaction failed */ });
});
```

### 协程

Redis 客户端也支持协程. 需要GCC 11或者更新的编译器，并且使用`cmake -DCMAKE_CXX_FLAGS="-std=c++20"` 来使能它。見[协程](CHN-16-协程)取得细节

```c++
try
{
    auto transaction = co_await redisClient->newTransactionCoro();
    co_await transaction->execCommandCoro("set zzz 123");
    co_await transaction->execCommandCoro("set mening 42");
    co_await transaction->executeCoro();
}
catch(const std::exception& e)
{
    LOG_ERROR << "Redis failed: " << e.what();
}
```
