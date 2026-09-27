<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-04-3-控制器-WebSocketController) - [English](/ENG/ENG-04-3-Controller-WebSocketController)

# المتحكم (Controller) - WebSocketController

كما يتضح من الاسم، يُستخدم `WebSocketController` لمعالجة منطق WebSocket. يُعد WebSocket مخطط اتصال مستمر يقوم على بروتوكول HTTP. في بداية اتصال WebSocket، تحدث عملية تبادل لطلبات واستجابات بصيغة HTTP. وبعد إنشاء الاتصال، تُنقل جميع الرسائل عبر اتصال WebSocket. يتم تغليف الرسالة في نسق ثابت، ولا توجد قيود على محتوى الرسالة أو ترتيب نقل الرسائل.

### الإنشاء الآلي (Generation)

يمكن إنشاء الملف المصدري لـ `WebSocketController` باستخدام أداة `drogon_ctl`. صيغة الأمر كالتالي:

```shell
drogon_ctl create controller -w <[namespace::]class_name>
```

لنفترض أننا نريد تنفيذ دالة صدى (Echo) بسيطة عبر WebSocket، حيث يقوم الخادم ببساطة بإعادة إرسال الرسالة التي يستقبلها من العميل. يمكننا إنشاء فئة التنفيذ `EchoWebsock` للمتحكم `WebSocketController` عبر `drogon_ctl` كما يلي:

```shell
drogon_ctl create controller -w EchoWebsock
```

سينتج عن هذا الأمر ملفان هما EchoWebsock.h و EchoWebsock.cc كالتالي:

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
    // اذكر تعريفات المسارات هنا؛
    WS_PATH_LIST_END
};
```

```c++
//EchoWebsock.cc
#include "EchoWebsock.h"
void EchoWebsock::handleNewMessage(const WebSocketConnectionPtr &wsConnPtr,std::string &&message)
{
    // اكتب منطق التطبيق الخاص بك هنا
}
void EchoWebsock::handleNewConnection(const HttpRequestPtr &req,const WebSocketConnectionPtr &wsConnPtr)
{
    // اكتب منطق التطبيق الخاص بك هنا
}
void EchoWebsock::handleConnectionClosed(const WebSocketConnectionPtr &wsConnPtr)
{
    // اكتب منطق التطبيق الخاص بك هنا
}
```

### الاستخدام (Usage)

- #### تعيين المسارات (Path Mapping)

  بعد التعديل:

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
      // اذكر تعريفات المسارات هنا؛
      WS_PATH_ADD("/echo");
      WS_PATH_LIST_END
  };
  ```

  ```c++
  //EchoWebsock.cc
  #include "EchoWebsock.h"
  void EchoWebsock::handleNewMessage(const WebSocketConnectionPtr &wsConnPtr,std::string &&message)
  {
      // اكتب منطق التطبيق الخاص بك هنا
      wsConnPtr->send(message);
  }
  void EchoWebsock::handleNewConnection(const HttpRequestPtr &req,const WebSocketConnectionPtr &wsConnPtr)
  {
      // اكتب منطق التطبيق الخاص بك هنا
  }
  void EchoWebsock::handleConnectionClosed(const WebSocketConnectionPtr &wsConnPtr)
  {
      // اكتب منطق التطبيق الخاص بك هنا
  }
  ```

  أولاً، في هذا المثال، تم تسجيل المتحكم على المسار `/echo` عبر ماكرو `WS_PATH_ADD`. استخدام ماكرو `WS_PATH_ADD` يماثل الماكروهات الخاصة بالمتحكمات الأخرى المقدمة سابقاً. كما يمكن تسجيل المسار مع عدة فلاتر [Filters](/ARA/ARA-05-Middleware-and-Filter). ونظراً لأن WebSocket يتولى إطار العمل معالجته بشكل منفصل، فيمكن أن يتكرر مع مسارات المتحكمات السابقة (`HttpSimpleController` و `HttpApiController`) دون أن يؤثر أي منها على الآخر.

  ثانياً، في تنفيذ الدوال الوهمية الثلاث في هذا المثال، تملك الدالة `handleNewMessage` الفاعلية فقط، حيث تقوم ببساطة بإعادة إرسال الرسالة المستلمة إلى العميل عبر واجهة `send`. عند تجميع هذا المتحكم داخل إطار العمل، ستتمكن من رؤية النتيجة، ويُرجى تجربة ذلك بنفسك.

  **ملاحظة: مثل بروتوكول HTTP المعتاد، يمكن التنصت على حركة مرور HTTP WebSocket. إذا كانت الأمانة مطلوبة، فيجب توفير التشفير عبر HTTPS. بالطبع، يمكن للمستخدمين أيضاً إتمام التشفير وفك التشفير على جانب الخادم والعميل، لكن استخدام HTTPS أكثر ملاءمة. تتولى الطبقة السفلية في `drogon` المعالجة، بينما لا يحتاج المستخدم سوى إلى الاهتمام بمنطق العمل.**

  ترث فئة متحكم WebSocket المعرفة من قبل المستخدم من قالب الفئة `drogon::WebSocketController`. معامل القالب هو نوع الفئة الفرعية. يحتاج المستخدم إلى تنفيذ الدوال الوهمية الثلاث التالية لمعالجة إنشاء اتصال WebSocket، وإغلاقه، واستقبال الرسائل:

  ```c++
  virtual void handleNewConnection(const HttpRequestPtr &req,const WebSocketConnectionPtr &wsConn);
  virtual void handleNewMessage(const WebSocketConnectionPtr &wsConn,std::string &&message,
  const WebSocketMessageType &);
  virtual void handleConnectionClosed(const WebSocketConnectionPtr &wsConn);
  ```

  من السهل معرفة أن:

  - `handleNewConnection`: يُستدعى بعد إنشاء اتصال WebSocket. يمثل `req` طلب الإنشاء المرسل من العميل. في هذه المرحلة، يكون إطار العمل قد أعاد الاستجابة بالفعل. كل ما يمكن للمستخدم فعله هو الحصول على بعض المعلومات الإضافية عبر `req` مثل الـ `token`. يمثل `wsConn` مؤشراً ذكياً لكائن WebSocket هذا، وسنتناول الواجهات الشائعة الاستخدام لاحقاً.
  - `handleNewMessage`: يُستدعى بعد استقبال WebSocket لرسالة جديدة. تُحفظ الرسالة في متغير `message`. لاحظ أن الرسالة تمثل الحمولة الفعالة (Payload)؛ حيث يكون إطار العمل قد أنهى فك التغليف وفك التشفير للرسالة، ويمكن للمستخدم معالجة الرسالة نفسها مباشرة.
  - `handleConnectionClosed`: يُستدعى بعد إغلاق اتصال WebSocket، ويمكن للمستخدم القيام ببعض أعمال الإنهاء.

### الواجهات (Interfaces)

  الواجهات الشائعة لكائن WebSocketConnection كالتالي:
  
  ```c++
  // إرسال رسالة websocket، وتكون عملية التشفير والتغليف
  // للرسالة من مسؤولية إطار العمل
  void send(const char *msg,uint64_t len);
  void send(const std::string &msg);

  // العنوان المحلي والبعيد لـ websocket
  const trantor::InetAddress &localAddr() const;
  const trantor::InetAddress &peerAddr() const;

  // حالة الاتصال لـ websocket
  bool connected() const;
  bool disconnected() const;

  // إغلاق websocket
  void shutdown();// إغلاق الكتابة
  void forceClose();// إغلاق قسري

  // إعداد والحصول على سياق websocket، وتخزين بعض بيانات العمل الخاصة بالمستخدمين.
  // النوع any يعني أنه يمكنك تخزين أي نوع من الكائنات.
  void setContext(const any &context);
  const any &getContext() const;
  any *getMutableContext();
  ```

# التالي: [البرمجيات الوسيطة والفلاتر (Middleware and Filter)](/ARA/ARA-05-Middleware-and-Filter)

</div>
