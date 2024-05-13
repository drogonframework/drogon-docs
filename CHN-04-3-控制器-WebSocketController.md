[English](ENG-04-3-Controller-WebSocketController) | [简体中文](CHN-04-3-控制器-WebSocketController)

顾名思义，`WebSocketController`用于处理websocket逻辑。websocket是基于HTTP的一种长连接方案，在websocket建立之初，有一次HTTP格式的请求和应答交换，建立完成后，所有的消息在websocket上传输，消息由固定的格式包装，但消息的内容和收发次序没有任何要求，完全由用户定义。

### 生成

可以由`drogon_ctl`工具快速生成`WebSocketController`的源文件，命令格式如下：

```shell
drogon_ctl create controller -w <[namespace::]class_name>
```

假设我们要通过websocket实现一个简单的回声功能，即服务端只是简单的把客户端发来的消息再发回去，通过`drogon_ctl`创建`WebSocketController`的实现类EchoWebsock，如下：

```shell
drogon_ctl create controller -w EchoWebsock
```

该命令会生成EchoWebsock.h和EchoWebsock.cc两个文件，

```c++
//EchoWebsock.h
#pragma once
#include <drogon/WebSocketController.h>
using namespace drogon;
class EchoWebsock:public drogon::WebSocketController<EchoWebsock>
{
  public:
    void handleNewMessage(const WebSocketConnectionPtr&,
                          std::string &&,
                          const WebSocketMessageType &) override;
    void handleNewConnection(const HttpRequestPtr &,
                             const WebSocketConnectionPtr&) override;
    void handleConnectionClosed(const WebSocketConnectionPtr&) override;
    WS_PATH_LIST_BEGIN
    //list path definitions here;
    WS_PATH_LIST_END
};
```

```c++
#include "EchoWebsock.h"
void EchoWebsock::handleNewMessage(const WebSocketConnectionPtr &wsConnPtr,std::string &&message)
{
    //write your application logic here
}
void EchoWebsock::handleNewConnection(const HttpRequestPtr &req,const WebSocketConnectionPtr &wsConnPtr)
{
    //write your application logic here
}
void EchoWebsock::handleConnectionClosed(const WebSocketConnectionPtr &wsConnPtr)
{
    //write your application logic here
}
```

### 使用

* #### 路径映射

  编辑后内容如下：

  ```c++
  //EchoWebsock.h
  #pragma once
  #include <drogon/WebSocketController.h>
  using namespace drogon;
  class EchoWebsock:public drogon::WebSocketController<EchoWebsock>
  {
  public:
      virtual void handleNewMessage(const WebSocketConnectionPtr&,
                                  std::string &&,
                                  const WebSocketMessageType &)override;
      virtual void handleNewConnection(const HttpRequestPtr &,
                                      const WebSocketConnectionPtr&)override;
      virtual void handleConnectionClosed(const WebSocketConnectionPtr&)override;
      WS_PATH_LIST_BEGIN
      //list path definitions here;
      WS_PATH_ADD("/echo");
      WS_PATH_LIST_END
  };
  ```

  ```c++
  //EchoWebsock.cc
  #include "EchoWebsock.h"
  void EchoWebsock::handleNewMessage(const WebSocketConnectionPtr &wsConnPtr,std::string &&message)
  {
      //write your application logic here
      wsConnPtr->send(message);
  }
  void EchoWebsock::handleNewConnection(const HttpRequestPtr &req,const WebSocketConnectionPtr &wsConnPtr)
  {
      //write your application logic here
  }
  void EchoWebsock::handleConnectionClosed(const WebSocketConnectionPtr &wsConnPtr)
  {
      //write your application logic here
  }
  ```

  首先，在这个例子中，通过`WS_PATH_ADD`宏把这个控制器注册到了`/echo`路径上，`WS_PATH_ADD`宏的用法跟之前介绍的其他控制器的宏类似，也可以注册路径并且附带若干[中间件和过滤器](CHN-05-中间件和过滤器)。由于websocket在框架中单独处理，所以它可以和前两种控制器的路径重复而不会相互影响。

  其次，本例中三个虚函数的实现，只有handleNewMessage有实质内容，只是简单的把收到的消息通过send接口发回客户端。把这个控制器编译进框架，就可以看到效果，请各位自己试验吧。

  > **注意：和通常的HTTP协议一样，http的websocket可以被旁路还原，如果需要安全保障，应由https提供加密功能，当然，用户自己在服务端和客户端完成加密和解密也是可以的，只是https更方便，底层都由drogon处理，用户只需关心业务逻辑。**

  用户自定义的`WebSocketController`类继承自`drogon::WebSocketController`类模板，模板参数是子类类型，用户需自己实现如下三个虚函数来对websocket的建立、关闭和消息进行处理：

  ```c++
  virtual void handleNewConnection(const HttpRequestPtr &req,const WebSocketConnectionPtr &wsConn);
  virtual void handleNewMessage(const WebSocketConnectionPtr &wsConn,std::string &&message,
  const WebSocketMessageType &);
  virtual void handleConnectionClosed(const WebSocketConnectionPtr &wsConn);
  ```

  容易知道：

  * handleNewConnection在websocket建立之后被调用，req是客户端发来的建立请求，这时候框架已经返回了response，用户可以做的是通过req获得一些额外信息，比如token之类。wsConn是这个websocket对象的智能指针，常用的接口后面再谈。
  * handleNewMessage在websocket收到新的消息之后被调用，消息存储在message变量里，注意这个message是完整的消息净荷，框架已经做完了消息的解封包和解码等工作，用户直接处理消息本身即可。
  * handleConnectionClosed在websocket连接关闭之后调用，用户可以做一些收尾工作。

* #### 接口

  WebSocketConnection对象常用接口如下：

  ```c++
  //发送websocket消息，消息的编码和封包都由框架负责，这里直接发送消息的净荷
  void send(const char *msg,uint64_t len);
  void send(const std::string &msg);

  //本websocket的本端和远端地址
  const trantor::InetAddress &localAddr() const;
  const trantor::InetAddress &peerAddr() const;

  //本weosocket的连接状态
  bool connected() const;
  bool disconnected() const;

  //关闭本websocket
  void shutdown();//close write
  void forceClose();//close

  //设置和获取本websocket的上下文，由用户存入一些业务数据，
  //any类型意味着可以存取任意类型的对象。
  void setContext(const any &context);
  const any &getContext() const;
  any *getMutableContext();
  ```

# 05 [中间件和过滤器](CHN-05-中间件和过滤器)
