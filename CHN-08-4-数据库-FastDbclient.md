顾名思义，FastDbClient会提供比普通的DbClient更高的性能。与DbClient拥有自己的EventLoop不同，它和Web应用的网络IO线程和主线程共用EventLoop，这使得FastDbClient的内部实现可以采用无锁的方式进行，因而会更高效。

经测试，极限高负载条件下，FastDbClient比DbClient有10%到20%的性能提升。

### 创建和获取

FastDbClient必须由框架使用框架的接口或通过配置文件创建，使用框架的createDbClient接口，当最后一个参数为true时，即可创建一个FastDbClient对象。

配置文件中的每个db_client配置项下有个is_fast子选项，该选项为true时，表明该对象是FastDbClient。

框架会针对每个IO事件循环和主事件循环创建单独的FastDbClient，每个FastDbClient内部管理数个数据库连接。IO事件循环数由框架的"threads_num"选择控制，一般设为主机的CPU核心数，每个事件循环管理的数据库连接数由数据库客户端的"connection_number"选项控制，所以一项FastDbClient的总连接数为`(threads_num+1) * connection_number`，参考[配置文件](CHN-10-配置文件#db_clients)。

FastDbClient的获取接口和普通DbClient的类似，如下：

```c++
orm::DbClientPtr getFastDbClient(const std::string &name = "default");
/// 使用drogon::app().getFastDbCLient("clientName")调用
```

需要指出的是，由于FastDbClient的特殊性，用户必须在IO事件循环线程或主线程内调用上述接口才能得到正确的智能指针，在其它线程只能获得空指针，无法使用。

### 使用

FastDbClient的使用与普通的DbClient几乎完全一致，除了下面这些限制（高性能的代价是使用上有约束，这是可以理解的）。

* 获取和使用都必须在框架的IO事件循环线程或主线程内，在其它线程获取FastDbClient只能得到空指针，在其它线程使用它，会有无法预料的错误（因为无锁的条件遭到破坏），好在用户在应用编程的多数地方都是在IO线程内，比如各种控制器的处理函数内，过滤器的过滤函数内。容易知道，FastDbClient接口的各种回调函数内也是在当前IO线程，可以放心嵌套使用。
* 永远不要使用FastDbClient的阻塞接口，因为这种接口会阻塞当前线程，而当前线程也是处理这个对象的数据库IO的线程，这会造成永久的阻塞，用户没有机会拿到结果。
* 同步的事务创建接口是有可能阻塞的（所有连接都忙的时候），所以FastDbClient的同步事务创建接口直接返回空指针，如果要在FastDbClient上使用事务，请使用异步的事务创建接口。
* 使用FastDbClient创建Orm的Mapper对象后，使用时也要注意只能使用异步非阻塞接口。

# 08.5 [自动批处理](CHN-08-3-数据库-自动批处理)