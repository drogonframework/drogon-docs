[English](ENG-04-1-Controller-HttpSimpleController) | [简体中文](CHN-04-1-控制器-HttpSimpleController)

可以由`drogon_ctl`命令行工具快速生成基于`HttpSimpleController`的自定义类的源文件，命令格式如下：

```shell
drogon_ctl create controller <[namespace::]class_name>
```

我们创建一个名称为`TestCtrl`的控制器：

```shell
drogon_ctl create controller TestCtrl
```

可以看到，目录下新增加了两个文件，TestCtrl.h和TestCtrl.cc，下面阐述一下这两个文件。

TestCtrl.h如下：

```c++
#pragma once
#include <drogon/HttpSimpleController.h>
using namespace drogon;
class TestCtrl:public drogon::HttpSimpleController<TestCtrl>
{
public:
    virtual void asyncHandleHttpRequest(const HttpRequestPtr &req,
                                        std::function<void (const HttpResponsePtr &)> &&callback)override;
    PATH_LIST_BEGIN
    //list path definitions here;
    //PATH_ADD("/path","filter1","filter2",HttpMethod1,HttpMethod2...);
    PATH_LIST_END
};
```

TestCtrl.cc如下：

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void (const HttpResponsePtr &)> &&callback)
{
    //write your application logic here
}
```

每个HttpSimpleController类只能定义一个Http请求处理函数(handler)，而且通过虚函数重载定义。

从URL路径到处理函数的路由(或称映射)由宏完成，可以用`PATH_ADD`宏添加多重路径映射，所有`PATH_ADD`语句应夹在`PATH_LIST_BEGIN`和`PATH_LIST_END`宏语句之间。

第一个参数是映射的路径,路径后面的参数是对这个路径的约束，目前支持两种约束，一种是`HttpMethod`类型，表示该路径允许使用的Http方法，可以配置零个或多个，一种是`HttpFilter`类的名字，这种对象执行特定的过滤操作，也可以配置0个或多个，两种类型没有顺序要求，框架会处理好类型的匹配。关于Filter，请参阅[中间件和过滤器](CHN-05-中间件和过滤器)。

用户可以把同一个Simple Controller注册到多个路径上，也可以在同一个路径上注册多个Simple Controller通过 HTTP method 区分）。

你可以定义一个HttpResponse类的变量，然后使用callback()返回这个变量即可:

```c++
    //write your application logic here
    auto resp=HttpResponse::newHttpResponse();
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody("Your Page Contents");
    callback(resp);
```

> **上述路径到处理函数的映射是在编译期完成的，事实上，drogon框架也提供了运行期完成映射的接口，运行期映射可以让用户通过配置文件或其它用户接口完成映射或修改映射关系而无需重新编译这个程序(出于性能的考虑，禁止在运行app().run()之后再注册任何映射)。**

# 04.2 [HttpController](CHN-04-2-控制器-HttpController)
