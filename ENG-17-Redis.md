Drogon supports Redis, a very fast, in-memory data store. Which could be used as a database cache or a message broker. Like everything in Drogon, Redis connections are asynchronous.  Which ensures Drogon running with very high concurrency even under heavy load. 

Redis support depends on the `hiredis` library. Redis support won't be available if hiredis is not available when building Drogon.

### Creating a client

Redis clients can be created and retrieve pragmatically through `app()`.

```c++
app().createRedisClient("127.0.0.1", 6379);
...
// After app.run()
RedisClientPtr redisClient = app().getRedisClient();
```

### Using Redis

execCommandAsync executes Redis commands in an asynchronous mannar. It takes at least 3 parameters, the first and second are callback whom will be called when the Redis command succeed or failed. The thrid being the command it self. The command could be a C-style format string. And the rests are arguments for the format string.  For example, to set the key `name` to `drogon`:

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {}
    [](const std::exception &err) {
        LOG_ERROR << "something failed!!! " << e.what();
    },
    "set name drogon");
```

Or set `myid` to `587d-4709-86e4`

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {}
    [](const std::exception &err) {
        LOG_ERROR << "something failed!!! " << e.what();
    },
    "set myid %s", "587d-4709-86e4");
```

The same execCommandAsync can also retrieve data from Redis. 

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

### Transaction

Redis transaction allows multiple commands to be executed in a single step. All commands within a transaction are executed in order, no commands by other clients can be executed **in middle** of a transaction. And a transaction is atomic. Meaning either everything is executed, or something went wrong and everything is rolled back before executing the transaction.

The newTransactionAsync method creates a new transaction. Then the transaction could be used just like a normal RedisClient. Finally, the RedisTransaction::execute method executes said transaction.

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
