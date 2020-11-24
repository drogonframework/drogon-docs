`会话（Session）`是web应用的重要概念，用于在服务端保存客户端的状态，一般和浏览器的`cookie`配合，drogon提供了对会话的支持。drogon默认**关闭**会话选择，你也可以通过如下接口关闭或打开：

```c++
        void disableSession();
        void enableSession(const size_t timeout=0);  
```

都是通过`HttpAppFramework`单例调用，timeout参数代表了会话失效的时间，单位是秒，框架默认值是1200，即如果用户20分钟以上没有访问应用，则他对应的会话就失效了。timeout设置为0表示drogon将在整个生存期保留用户的会话；

打开会话特性前请确定你的客户端支持cookie，否则，drogon会为每次不含SessionID的请求创建新的会话，这会白白浪费内存和计算资源。

### 会话对象

drogon的会话对象类型是`drogon::Session`，它和`HttpViewData`非常类似，可以通过关键字存取任意类型的对象；支持并发读写；具体的使用请参考Session class的说明；

drogon框架会把会话对象放到`HttpRequest`对象里传递给用户，用户可以通过`HttpRequest`类的如下接口获取`Session`对象。

```c++
SessionPtr session() const;
```

获得的是Session对象的智能指针，通过它可以存取各种对象；

### 会话的例子

我们这次加一个需要会话支持的功能，比如，我们要限制用户的访问频度，某一次访问后，如果10秒以内再次访问，就返回错误，否则返回ok。我们需要在会话里记录上次访问的时间，然后和本次访问的时间做比较，就可以实现这个功能。

我们创建一个Filter来实现这个功能，假设类名是TimeFilter，实现如下：

```c++
#include "TimeFilter.h"
#include <trantor/utils/Date.h>
#include <trantor/utils/Logger.h>
#define VDate "visitDate"
void TimeFilter::doFilter(const HttpRequestPtr &req,
                          FilterCallback &&cb,
                          FilterChainCallback &&ccb)
{
    trantor::Date now=trantor::Date::date();
    LOG_TRACE<<"";
    if(req->session()->find(VDate))
    {
        auto lastDate=req->session()->get<trantor::Date>(VDate);
        LOG_TRACE<<"last:"<<lastDate.toFormattedString(false);
        req->session()->modify<trantor::Date>(VDate,
                                        [now](trantor::Date &vdate) {
                                            vdate = now;
                                        });
        LOG_TRACE<<"update visitDate";
        if(now>lastDate.after(10))
        {
            //10 sec later can visit again;
            ccb();
            return;
        }
        else
        {
            Json::Value json;
            json["result"]="error";
            json["message"]="Access interval should be at least 10 seconds";
            auto res=HttpResponse::newHttpJsonResponse(json);
            cb(res);
            return;
        }
    }
    LOG_TRACE<<"first access,insert visitDate";
    req->session()->insert(VDate,now);
    ccb();
}
```

我们再注册一个lambda表达式到`/slow`路径上，同时附加上TimeFilter，代码如下：

```c++
drogon::HttpAppFramework::instance()
    .registerHandler
     ("/slow",
      [=](const HttpRequestPtr &req,
          std::function<void (const HttpResponsePtr &)> &&callback)
          {
              Json::Value json;
              json["result"]="ok";
              auto resp=HttpResponse::newHttpJsonResponse(json);
              callback(resp);
          },
          {Get,"TimeFilter"}
      );
```

调用框架接口打开会话

```c++
drogon::HttpAppFramework::instance().enableSession(1200);
```

用cmake重新编译整个工程，运行目标程序webapp，就可以通过浏览器看到效果了。

# [数据库](CHN-08-0-数据库-概述)