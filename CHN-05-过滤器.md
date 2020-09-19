过滤器(filter)可以帮助用户提高编程效率，在HttpController的[例子](CHN-04-2-控制器-HttpController)中，getInfo方法在返回用户信息之前应该先校验用户是否登录，我们把这个逻辑写在getInfo方法里当然是可以的，但是，很显然，校验用户登录属于通用逻辑，很多接口都将用到，应该把它单独提取出来，再配置到调用handler之前，这就是filter的作用。

drogon框架做完URL路径匹配后，会先依次调用注册到该路径上的过滤器，只有当所有过滤器都允许"通过"时，对应的handler才会被调用;

### 内置过滤器

drogon内置了如下常用过滤器:

* `drogon::IntranetIpFilter`:只放行内网ip发来的http请求，否则返回404页面；
* `drogon::LocalHostFilter`:只放行本机127.0.0.1或者::1发来的http请求，否则返回404页面；

### 自定义过滤器

当然，用户可以自定义过滤器，需要继承HttpFilter类模板，模板类型就是子类类型，比如我们想做一个LoginFilter，就可以定义如下:

```c++
class LoginFilter:public drogon::HttpFilter<LoginFilter>
{
public:
    virtual void doFilter(const HttpRequestPtr &req,
                          FilterCallback &&fcb,
                          FilterChainCallback &&fccb) override ;
};
```

你可以通过 `drogon_ctl` 命令创建过滤器, 见 [drogon_ctl](CHN-11-drogon_ctl命令#过滤器创建).

我们需要重载父类的doFilter虚函数实现过滤器逻辑；

这个虚函数有三个参数，分别是：

* **req**: http请求；
* **fcb**：过滤器回调函数，函数类型是void (HttpResponsePtr)，当过滤器判定请求不合法时，通过这个回调把特定的响应返回给浏览器；
* **fccb**：过滤器链回调函数，函数类型是void ()，当过滤器判定请求合法时，通过这个回调告诉drogon调用下一个过滤器或者最终的handler；

具体的实现可以参考drogon内置过滤器的实现。

### 过滤器的注册

过滤器总是伴随controller的注册进行，前面提到的注册handler的宏(PATH_ADD,METHOD_ADD等)都可以在最后添加一个或多个过滤器名字；比如，我们把前面getInfo方法的注册行改为如下形式：

```c++
METHOD_ADD(User::getInfo,"/{1}/info?token={2}",Get,"LoginFilter");
```

则在路径匹配成功后，必须满足如下条件，getInfo方法才会被调用：

1. 请求必须是http get请求；
2. 请求方必须已经登录；

可以看到，过滤器的配置和注册是非常简单的，注册过滤器的controller文件并不需要引用过滤器的头文件，过滤器和控制器也是充分解耦的。

**注意: 如果过滤器定义在命名空间里，注册过滤器时必须把命名空间写全**

# 06 [视图](CHN-06-视图)