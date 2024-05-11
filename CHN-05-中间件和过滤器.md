[English](ENG-05-Middleware-and-Filter) | [简体中文](CHN-05-中间件和过滤器)

中间件(middleware)和过滤器(filter)可以帮助用户提高编程效率，在HttpController的[例子](CHN-04-2-控制器-HttpController)中，getInfo方法在返回用户信息之前应该先校验用户是否登录，我们把这个逻辑写在getInfo方法里当然是可以的，但是，很显然，校验用户登录属于通用逻辑，很多接口都将用到，应该把它单独提取出来，再配置到调用handler之前，这就是filter的作用。

drogon的中间件采用了洋葱圈模型, 框架做完URL路径匹配后，会依次调用注册到该路径上的中间件，在每个中间件中，用户可以选择拦截或放行请求，并添加前置、后置处理逻辑。
如果有一个中间件拦截了请求，该请求将不会继续深入洋葱圈内层，对应的handler也不会被调用，但是仍然会通过外层中间件的后置处理逻辑。

过滤器实际上是省略后置操作的中间件，经过进一步的包装后暴露给用户。中间件和过滤器可以在注册路径时混合使用。

### 内置中间件/过滤器

drogon内置了如下常用过滤器:

* `drogon::IntranetIpFilter`:只放行内网ip发来的http请求，否则返回404页面；
* `drogon::LocalHostFilter`:只放行本机127.0.0.1或者::1发来的http请求，否则返回404页面；

### 自定义中间件/过滤器

* #### 中间件的定义
  用户可以自定义中间件，需要继承`HttpMiddleware`类模板，模板类型就是子类类型，比如我们想为某些路由开启跨域支持，就可以定义如下:
  ```c++
  class MyMiddleware : public HttpMiddleware<MyMiddleware>
  {
  public:
      MyMiddleware(){};  // do not omit constructor

      void invoke(const HttpRequestPtr &req,
                  MiddlewareNextCallback &&nextCb,
                  MiddlewareCallback &&mcb) override
      {
          const std::string &origin = req->getHeader("origin");
          if (origin.find("www.some-evil-place.com") != std::string::npos)
          {
              // intercept directly
              mcb(HttpResponse::newNotFoundResponse(req));
              return;
          }
          // Do something before calling the next middleware
          nextCb([mcb = std::move(mcb)](const HttpResponsePtr &resp) {
              // Do something after the next middleware returns
              resp->addHeader("Access-Control-Allow-Origin", origin);
              resp->addHeader("Access-Control-Allow-Credentials","true");
              mcb(resp);
          });
      }
  };
  ```

  我们需要重载父类的`invoke`虚函数实现中间件逻辑；

  这个虚函数有三个参数，分别是：

  * **req**: http请求；
  * **nextCb**：进入内层的回调函数，调用该函数意味着继续深入洋葱圈内层，调用下一个中间件或最终handler。
    调用nextCb时接受另一个函数作为参数, 当从洋葱圈内层返回时，该函数会被调用，并传入内层返回的HttpResponsePtr。
  * **mcb**：返回上层的回调函数，调用该函数意味着返回洋葱圈上层。若跳过nextCb只调用mcb，意味着拦截该请求，直接返回上层。

* #### 过滤器的定义

  用户可以自定义过滤器，需要继承HttpFilter类模板，模板类型就是子类类型，比如我们想做一个LoginFilter，就可以定义如下:

  ```c++
  class LoginFilter:public drogon::HttpFilter<LoginFilter>
  {
  public:
      void doFilter(const HttpRequestPtr &req,
                    FilterCallback &&fcb,
                    FilterChainCallback &&fccb) override ;
  };
  ```

  你可以通过 `drogon_ctl` 命令创建过滤器, 见 [drogon_ctl](CHN-11-drogon_ctl命令#过滤器创建).

  我们需要重载父类的doFilter虚函数实现过滤器逻辑；

  这个虚函数有三个参数，分别是：

  * **req**: http请求；
  * **fcb**：过滤器回调函数，函数类型是void (HttpResponsePtr)，当过滤器判定请求不合法时，通过这个回调把特定的响应返回给浏览器；
  * **fccb**：过滤器链回调函数，函数类型是void()，当过滤器判定请求合法时，通过这个回调告诉drogon调用下一个过滤器或者最终的handler；

  具体的实现可以参考drogon内置过滤器的实现。

* #### 中间件/过滤器的注册

  中间件/过滤器总是伴随controller的注册进行，前面提到的注册handler的宏(PATH_ADD,METHOD_ADD等)都可以在最后添加一个或多个中间件/过滤器名字；比如，我们把前面getInfo方法的注册行改为如下形式：

  ```c++
  METHOD_ADD(User::getInfo,"/{1}/info?token={2}",Get,"LoginFilter","MyMiddleware");
  ```

  则在路径匹配成功后，必须满足如下条件，getInfo方法才会被调用：

  1. 请求必须是http get请求；
  2. 请求方必须已经登录；

  可以看到，中间件/过滤器的配置和注册是非常简单的，注册中间件的controller文件并不需要引用中间件的头文件，中间件和控制器也是充分解耦的。

  > **注意: 如果中间件/过滤器定义在命名空间里，注册时必须把命名空间写全**

# 06 [视图](CHN-06-视图)
