[English](ENG-08-1-Database-DbClient) | [简体中文](CHN-08-1-数据库-DbClient)

### 构建DbClient

构造DbClient对象有两种途径，一个是通过DbClient类的静态方法，在DbClient.h头文件可以看到定义，如下：

```c++
#if USE_POSTGRESQL
    static std::shared_ptr<DbClient> newPgClient(const std::string &connInfo, const size_t connNum);
#endif
#if USE_MYSQL
    static std::shared_ptr<DbClient> newMysqlClient(const std::string &connInfo, const size_t connNum);
#endif
```

得到DbClient实现对象的智能指针，参数connInfo是个连接字符串，采用key=value的形式设置一系列连接参数，具体说明见头文件的注释。参数connNum是DbClient的连接数，即该对象管理的数据库连接个数，对并发有关键影响，请根据实际情况设置。

通过这种方法得到的对象，用户要想办法**持久化**，比如放在某些全局容器内，**创建临时对象，使用完再释放是非常不推荐的方案**，理由如下：

* 白白的浪费创建连接和断开连接的时间，增加了系统时延；
* 该接口也是非阻塞接口，也就是说，用户拿到DbClient对象时，它管理的连接还没建立起来，框架没有（故意的）提供连接建立成功的回调接口，难道还要sleep一下再开始查询么？这和异步框架的初衷相违背。

所以，应该在程序开始之初就构建这些对象，并在整个生存周期持有并使用它。显然，这个工作完全可以由框架来做，因此，框架提供了第二种构建方式，就是通过配置文件构建或使用createDbClient接口创建，配置方法见[配置文件](9-配置文件#db_clients数据库客户端)。

需要使用时，通过框架的接口获得DbClient的智能指针，接口如下（注意该接口必须在app.run()调用后才能得到正确的对象）：

```c++
orm::DbClientPtr getDbClient(const std::string &name = "default");
```

参数name就是配置文件中的name配置项的值，用以区分同一个应用的多个不同的DbClient对象。DbClient管理的连接总是断线重连的，所以用户不用关心连接状态，他们几乎总是正常连接的状态。

### 执行接口

DbClient对外提供了几种不同的接口，列举如下：

```c++
/// 异步接口
template <typename FUNCTION1,
          typename FUNCTION2,
          typename... Arguments>
void execSqlAsync(const std::string &sql,
                  FUNCTION1 &&rCallback,
                  FUNCTION2 &&exceptCallback,
                  Arguments &&... args) noexcept;

/// 异步future接口
template <typename... Arguments>
std::future<const Result> execSqlAsyncFuture(const std::string &sql,
                                             Arguments &&... args) noexcept;

/// 同步接口
template <typename... Arguments>
const Result execSqlSync(const std::string &sql,
                         Arguments &&... args) noexcept(false);

/// 流式接口
internal::SqlBinder operator<<(const std::string &sql);
```

因为涉及任意数量和类型的绑定参数，因此这些接口都是函数模板。

这些接口的性质如下表所示：

| 接口                                         | 同步/异步 | 阻塞/非阻塞                     | 异常                                |
| :------------------------------------------* | :-------* | :--------------------------* | :--------------------------------* |
| void execSqlAsync                            | 异步      | 非阻塞                        | 不抛异常                             |
| std::future<const Result> execSqlAsyncFuture | 异步      | 调用future的get方法时阻塞       | 调用future的get方法时可能抛异常       |
| const Result execSqlSync                     | 同步      | 阻塞                          | 可能抛异常                           |
| internal::SqlBinder operator<<               | 异步      | 默认非阻塞，也可以阻塞           | 不抛异常                            |

你可能对异步和阻塞的组合有点困惑，一般而言，同步接口涉及网络IO都是阻塞的，异步接口则是非阻塞的，不过，异步接口也可以工作于阻塞模式，意思是说，这个接口会阻塞一直等到回调函数执行完毕才会退出。DbClient的异步接口工作于阻塞模式时，回调函数会在同一个线程被执行，然后该接口才执行完毕。

如果你的应用涉及高并发场景，请选择异步非阻塞接口，如果是低并发场景，比如一个网络设备的管理页面，则可以出于直观方便的考虑，选择同步接口。

* #### execSqlAsync

  ```c++
  template <typename FUNCTION1,
          typename FUNCTION2,
          typename... Arguments>
  void execSqlAsync(const std::string &sql,
                  FUNCTION1 &&rCallback,
                  FUNCTION2 &&exceptCallback,
                  Arguments &&... args) noexcept;
  ```

  这是最常使用的异步接口，工作于非阻塞模式；

  参数`sql`是sql语句的字符串，如果有需要绑定参数的占位符，使用相应数据库的占位符规则，比如PostgreSQL的占位符是$1,$2..，而MySQL的占位符是？。

  不定参数`args`代表绑定的参数，可以是零个或多个，具体数据和sql语句的占位符个数一致，类型可以是以下几类：

  * 整数类型：可以是各种字长的整数，应和数据库字段类型相匹配；
  * 浮点类型：可以是`float`或者`double`，应和数据库字段类型相匹配；
  * 字符串类型：可以是`std::string`或者`const char[]`，对应数据库的字符串类型或者其他可以用字符串表示的类型；
  * 日期类型：`trantor::Date`类型，对应数据库的date，datetime，timestamp等字段类型。
  * 二进制类型：`std::vector<char>`类型，对应PostgreSQL的bytea类型或者Mysql的blob类型；

  这些参数可以是左值，也可以是右值，可以是变量，也可以是字面常量，用户可以自由掌握。

  参数rCallback和exceptCallback分别表示结果回调函数和异常回调函数，它们有固定的定义，如下：

  * 结果回调函数：调用类型为void (const Result &)，符合这个调用类型的各种可调用对象，std::function,lambda等等都可以作为参数传入；
  * 异常回调函数：调用类型为void (const DrogonDbException &)，可传入和这个调用类型一致的各种可调用对象；

  sql执行成功后，执行结果由Result类包装并通过结果回调函数传递给用户；如果sql执行有任何异常，异常回调函数被执行，用户可以从DrogonDbException对象获得异常信息。

  我们举个例子：

  ```c++
  auto clientPtr = drogon::app().getDbClient();
  clientPtr->execSqlAsync("select * from users where org_name=$1",
                              [](const drogon::orm::Result &result) {
                                  std::cout << r.size() << " rows selected!" << std::endl;
                                  int i = 0;
                                  for (auto row : result)
                                  {
                                      std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
                                  }
                              },
                              [](const DrogonDbException &e) {
                                  std::cerr << "error:" << e.base().what() << std::endl;
                              },
                              "default");
  ```

  从例子中我们可以看出，Result对象是个std标准兼容的容器，支持迭代器，它封装的结果集可以通过范围循环取到每一行的对象，Result，Row和Field对象的各种接口，请参考源码；

  DrogonDbException类是所有数据库异常的基类，具体的定义和它子类的说明，请参考源码中的注释。

* #### execSqlAsyncFuture

  ```c++
  template <typename... Arguments>
  std::future<const Result> execSqlAsyncFuture(const std::string &sql,
                                              Arguments &&... args) noexcept;
  ```

  异步future接口省略了前一个接口的中间两个参数（使用future对象代替回调函数），调用这个接口会立即返回一个future对象，用户必须调用future的get()方法，得到返回的结果，异常要通过try/catch机制得到，如果调用get()方法时没有try/catch，并且整个调用堆栈中也没有try/catch，则程序会在sql执行发生异常的时候退出。

  例如：

  ```c++
  auto f = clientPtr->execSqlAsyncFuture("select * from users where org_name=$1",
                                      "default");
  try
  {
      auto result = f.get(); // Block until we get the result or catch the exception;
      std::cout << result.size() << " rows selected!" << std::endl;
      int i = 0;
      for (auto row : result)
      {
          std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
      }
  }
  catch (const DrogonDbException &e)
  {
      std::cerr << "error:" << e.base().what() << std::endl;
  }
  ```

* #### execSqlSync

  ```c++
  template <typename... Arguments>
  const Result execSqlSync(const std::string &sql,
                          Arguments &&... args) noexcept(false);
  ```

  同步接口是最简单直观的，输入参数是sql字符串和绑定的参数，返回一个Result对象，调用会阻塞当前线程，并且在出现错误时抛异常，所以也要注意try/catch捕获异常。

  例如：

  ```c++
  try
  {
      auto result = clientPtr->execSqlSync("update users set user_name=$1 where user_id=$2",
                                          "test",
                                          1); // Block until we get the result or catch the exception;
      std::cout << result.affectedRows() << " rows updated!" << std::endl;
  }
  catch (const DrogonDbException &e)
  {
      std::cerr << "error:" << e.base().what() << std::endl;
  }
  ```

* #### operator<<

  ```c++
  internal::SqlBinder operator<<(const std::string &sql);
  ```

  流式接口比较特殊，它把sql语句和参数依次通过`<<`操作符输入，而通过`>>`操作符指定结果回调函数和异常回调函数，比如前面select的例子，使用流式接口是如下的样子：

  ```c++
  *clientPtr  << "select * from users where org_name=$1"
              << "default"
              >> [](const drogon::orm::Result &result)
                  {
                      std::cout << result.size() << " rows selected!" << std::endl;
                      int i = 0;
                      for (auto row : result)
                      {
                          std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
                      }
                  }
              >> [](const DrogonDbException &e)
                  {
                      std::cerr << "error:" << e.base().what() << std::endl;
                  };
  ```

  这种写法和第一种异步非阻塞接口是完全等效的，采用哪种接口取决于用户的使用习惯。如果想让它工作于阻塞模式，可以使用`<<`输入一个`Mode::Blocking`参数，这里不再赘述。

  另外，流式接口还有一个特殊的用法，使用一种特殊的结果回调，可以让框架逐行的把结果传递给用户，这种回调的调用类型如下：

  ```c++
  void (bool,Arguments...);
  ```

  第一个bool参数为true时，表示这次回调是一个空行，也就是，所有结果都已经返回了，这是最后一次回调；
  后面是一系列参数，对应一行记录的每一列的值，框架会做好类型转换，当然，用户也要注意类型的匹配。这些类型可以是const型的左值引用，也可以是右值引用，当然也可以是值类型。

  我们再把上一个例子用这种回调重写一下：

  ```c++
  int i = 0;
  *clientPtr  << "select user_name, user_id from users where org_name=$1"
              << "default"
              >> [&i](bool isNull, const std::string &name, int64_t id)
                      {
                      if (!isNull)
                          std::cout << i++ << ": user name is " << name << ", user id is " << id << std::endl;
                      else
                          std::cout << i << " rows selected!" << std::endl;
                      }
              >> [](const DrogonDbException &e)
                  {
                      std::cerr << "error:" << e.base().what() << std::endl;
                  };
  ```

  可以看到，select语句中的user_name和user_id字段的值，被分别赋给了回调函数中的name和id变量，用户无需自己处理这些转换，这显然提供了一定的便利性，用户可以在实践中灵活运用。

> **注意: 借着这个例子，要强调一点异步编程必须注意的地方，就是上面例子中的变量i，用户必须保证在回调发生时，变量i还是有效的，因为它是被引用捕获的，它的有效性并不是理所当然的，回调会在别的线程被调用，而回调发生时，当前的上下文环境很可能已经失效了。类似的场景常常使用智能指针持有临时创建的变量，再被回调捕获，从而保证变量的有效性。**

### 总结

每个DbClient对象有且仅有一个自己的EventLoop线程，这个线程负责控制数据库连接IO，通过异步或同步接口接受请求，再通过回调函数返回结果。

它虽然也提供阻塞的接口，这种接口只是阻塞调用者线程，只要调用者线程不是EventLoop线程，就不会影响EventLoop线程的正常运转。回调函数被调用时，回调内的程序是运行在EventLoop线程的，所以，不要在回调内部进行任何阻塞操作，否则会影响数据库的并发，熟悉non-blocking I/O编程的人都应该明白这个约束。

# 08.2 [事务](CHN-08-2-数据库-事务)
