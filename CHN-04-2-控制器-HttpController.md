### 生成

可以由`drogon_ctl`命令行工具快速生成基于`HttpController`的自定义类的源文件，命令格式如下：

```shell
drogon_ctl create controller -h <[namespace::]class_name>
```

我们创建一个位于`demo v1`名称空间内且名称为`User`的控制器：

```shell
drogon_ctl create controller -h demo::v1::User
```

可以看到，目录下新增加了两个文件，demo_v1_User.h和demo_v1_User.cc:

demo_v1_User.h如下:
```c++
#pragma once

#include <drogon/HttpController.h>

using namespace drogon;

namespace demo
{
namespace v1
{
class User : public drogon::HttpController<User>
{
  public:
    METHOD_LIST_BEGIN
    // use METHOD_ADD to add your custom processing function here;
    // METHOD_ADD(User::get, "/{2}/{1}", Get); // path is /demo/v1/User/{arg2}/{arg1}
    // METHOD_ADD(User::your_method_name, "/{1}/{2}/list", Get); // path is /demo/v1/User/{arg1}/{arg2}/list
    // ADD_METHOD_TO(User::your_method_name, "/absolute/path/{1}/{2}/list", Get); // path is /absolute/path/{arg1}/{arg2}/list

    METHOD_LIST_END
    // your declaration of processing function maybe like this:
    // void get(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, int p1, std::string p2);
    // void your_method_name(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, double p1, int p2) const;
};
}
}
```

demo_v1_User.cc如下：
```c++
#include "demo_v1_User.h"

using namespace demo::v1;

// Add definition of your processing function here
```

### 使用

我们编辑一下这两个文件，然后再阐述它们。

demo_v1_User.h如下：

```c++
#pragma once

#include <drogon/HttpController.h>

using namespace drogon;

namespace demo
{
namespace v1
{
class User : public drogon::HttpController<User>
{
  public:
    METHOD_LIST_BEGIN
    // use METHOD_ADD to add your custom processing function here;
    METHOD_ADD(User::login,"/token?userId={1}&passwd={2}",Post);
    METHOD_ADD(User::getInfo,"/{1}/info?token={2}",Get);
    METHOD_LIST_END
    // your declaration of processing function maybe like this:
    void login(const HttpRequestPtr &req,
               std::function<void (const HttpResponsePtr &)> &&callback,
               std::string &&userId,
               const std::string &password);
    void getInfo(const HttpRequestPtr &req,
                 std::function<void (const HttpResponsePtr &)> &&callback,
                 std::string userId,
                 const std::string &token) const;
};
}
}
```
demo_v1_User.cc如下：
```c++
#include "demo_v1_User.h"

using namespace demo::v1;

// Add definition of your processing function here

void User::login(const HttpRequestPtr &req,
           std::function<void (const HttpResponsePtr &)> &&callback,
           std::string &&userId,
           const std::string &password)
{
    LOG_DEBUG<<"User "<<userId<<" login";
    //认证算法，读数据库，验证身份等...
    //...
    Json::Value ret;
    ret["result"]="ok";
    ret["token"]=drogon::utils::getUuid();
    auto resp=HttpResponse::newHttpJsonResponse(ret);
    callback(resp);
}
void User::getInfo(const HttpRequestPtr &req,
             std::function<void (const HttpResponsePtr &)> &&callback,
             std::string userId,
            const std::string &token) const
{
    LOG_DEBUG<<"User "<<userId<<" get his information";
    //验证token有效性等
    //读数据库或缓存获取用户信息
    Json::Value ret;
    ret["result"]="ok";
    ret["user_name"]="Jack";
    ret["user_id"]=userId;
    ret["gender"]=1;
    auto resp=HttpResponse::newHttpJsonResponse(ret);
    callback(resp);
}
```

每个`HttpController`类可以定义多个Http请求处理函数(handler)，由于函数数目可以任意多，所以通过虚函数重载是不现实的，我们需要把处理函数本身(而不是类)注册到框架里去。

从URL路径到处理函数的映射由宏完成，可以用`METHOD_ADD`宏或`ADD_METHOD_TO`宏添加多重路径映射，所有`METHOD_ADD`和`ADD_METHOD_TO`语句应夹在`METHOD_LIST_BEGIN`和`METHOD_LIST_END`宏语句之间。

`METHOD_ADD`宏会在路径映射中自动把**名字空间和类名**作为路径的前缀，所以，本例子中，login函数，被注册到了`/demo/v1/user/token`路径上，getInfo函数被注册到了`/demo/v1/user/xxx/info`路径上。后面的约束跟HttpSimpleController的PATH_ADD宏类似，不再赘述。

如果使用了自动的前缀，访问地址要包含命名空间和类名，此例中要使用`http://localhost/demo/v1/user/token?userid=xxx&passwd=xxx`或者`http://localhost/demo/v1/user/xxxxx/info?token=xxxx`来访问。

`ADD_METHOD_TO`宏的作用与前者几乎一样，除了它不会自动添加任何前缀，即这个宏注册的路径是一个绝对路径。

我们看到，`HttpController`提供了更为灵活的路径映射功能，并且可以注册多个处理函数，我们可以把一类功能放在一个类里。

另外可以看到，`METHOD_ADD`宏提供了参数映射的方法，我们可以把路径上的参数映射到函数的参数表里，由参数的数码对应形参的位置，非常方便，常见的可以由字符串类型转换的类型都可以作为参数(如std::string,int,float,double等等)，框架基于模板的类型推断会自动帮你转换类型，非常方便。注意左值引用必须是const类型。

同一个路径还可以注册多次，相互之间通过Http Method区分，这是合法的，并且是Restful API的通常做法，比如

```c++
 METHOD_LIST_BEGIN
     METHOD_ADD(Book::getInfo,"/{1}?detail={2}",Get);
     METHOD_ADD(Book::newBook,"/{1}",Post);
     METHOD_ADD(Book::deleteOne,"/{1}",Delete);
 METHOD_LIST_END
```

路径参数的占位符有多种写法:

* {}: 表示这个路径参数映射到处理函数的对应位置上，路径上的位置就是函数参数的位置。
* {1},{2}: 中间有个数字的，表示映射到数字指定的处理函数参数上。
* {anystring}: 中间的字符串没有实际作用，但可以提高程序的可读性，与`{}`等价。
* {1:anystring},{2:xxx}: 冒号前的数字表示位置，后面的字符串没有实际作用，但可以提高程序的可读性，与`{1}`,`{2}`等价。

推荐使用后两种写法，如果路径参数和函数参数顺序一直，使用第三种写法即可。容易知道，以下几种写法是等价的：

* "/users/{}/books/{}"
* "/users/{}/books/{2}"
* "/users/{user_id}/books/{book_id}"
* "/users/{1:user_id}/books/{2}"


**注意：路径匹配大小写不敏感，参数名字大小写敏感，参数值大小写保持原貌**

### 参数映射

通过前面的叙述我们知道，路径上的参数和问号后面的请求参数都可以映射到处理函数的参数列表里，目标参数的类型需要满足如下条件：

* 必须是值类型、常左值引用或非const右值引用中的一种，不能是非const的左值引用，推荐使用右值引用，这样用户可以随意处置它；

* int, long, long long, unsigned long, unsigned long long, float, double, long double等基础类型都可以作为参数类型；

* std::string类型；

* 任何可以使用`stringstream >>`操作符赋值的类型；

**另外，drogon框架还提供了从HttpRequestPtr对象到任意类型的参数的映射机制**，当你的handler参数列表中映射参数的数量多于路径上的参数时，后面多余的参数将由HttpRequestPtr对象转换得到，用户可以定义任意类型的转换，定义这种转换的方式是特化drogon命名空间的fromRequest模板(定义于HttpRequest.h头文件))，比如我们需要做一个创建新用户的RESTful的接口，我们定义用户的结构体如下：

```c++
namespace myapp{
struct User{
    std::string userName;
    std::string email;
    std::string address;
};
}
namespace drogon
{
template <>
inline myapp::User fromRequest(const HttpRequest &req)
{
    auto json = req.getJsonObject();
    myapp::User user;
    if(json)
    {
        user.userName = (*json)["name"].asString();
        user.email = (*json)["email"].asString();
        user.address = (*json)["address"].asString();
    }
    return user;
}

}
```

有了上面的定义和模板特化，我们就可以向下面这样定义路径和handler:

```c++
class UserController:public drogon::HttpController<UserController>
{
public:
    METHOD_LIST_BEGIN
        //use METHOD_ADD to add your custom processing function here;
        ADD_METHOD_TO(UserController::newUser,"/users",Post);
    METHOD_LIST_END
    //your declaration of processing function maybe like this:
    void newUser(const HttpRequestPtr &req,
                 std::function<void (const HttpResponsePtr &)> &&callback,
                 myapp::User &&pNewUser) const;
};
```

可以看到，第三个`myapp::User`类型的参数在映射路径上没有对应的占位符，框架会将它视为由`req`对象转换的参数，通过用户特化的函数模板得到这个参数，这都是drogon通过模板推导自动在编译期完成的，为用户的开发提供了极大便利。

更进一步，有些用户除了他们自定义类型的数据外，并不需要访问HttpRequestPtr对象，那么他可以把这个自定义的对象放在第一个参数的位置，框架也能正确完成映射，比如上面的例子也可以写成下面这样:

```c++
class UserController:public drogon::HttpController<UserController>
{
public:
    METHOD_LIST_BEGIN
        //use METHOD_ADD to add your custom processing function here;
        ADD_METHOD_TO(UserController::newUser,"/users",Post);
    METHOD_LIST_END
    //your declaration of processing function maybe like this:
    void newUser(myapp::User &&pNewUser,
                 std::function<void (const HttpResponsePtr &)> &&callback) const;
};
```

### 多路径映射

drogon支持在路径映射中使用正则表达式，在`{}`花括号以外的部分可以有限制的使用，比如

```c++
ADD_METHOD_TO(UserController::handler1,"/users/.*",Post); /// Match any path prefixed with `/users/`
ADD_METHOD_TO(UserController::handler2,"/{name}/[0-9]+",Post); ///Match any path composed with a name string and a number.
```

这种方法不支持子表达式，负向匹配等正则表达式，如果想使用他们，请用如下的方案。

### 正则表达式

上面的方法对正则表达式的支持比较有限，如果用户想自由使用正则表达式，drogon提供了`ADD_METHOD_VIA_REGEX`宏来实现这一点，比如

```c++
ADD_METHOD_VIA_REGEX(UserController::handler1,"/users/(.*)",Post); /// Match any path prefixed with `/users/` and map the rest of the path to a parameter of the handler1.
ADD_METHOD_VIA_REGEX(UserController::handler2,"/.*([0-9]*)",Post); /// Match any path that ends in a number and map that number to a parameter of the handler2.
ADD_METHOD_VIA_REGEX(UserController::handler3,"/(?!data).*",Post); /// Match any path that does not start with '/data'
```

可以看到，使用正则表达式也可以完成参数映射，所有子表达式匹配的字符串都会按顺序映射到handler的参数上。

**需要注意的是，使用正则表达式要注意匹配冲突（多个不同的handler都匹配），当冲突发生在同一个controller内部时，drogon只会执行第一个handler（先注册进框架的那个handler），当冲突发生在不同controller之间时，执行哪个handler是不确定的，因此用户需要避免这种冲突发生。**


# 04.3 [WebSocketController](CHN-04-3-控制器-WebSocketController)
