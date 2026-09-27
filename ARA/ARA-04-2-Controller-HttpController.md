<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-04-2-控制器-HttpController) - [English](/ENG/ENG-04-2-Controller-HttpController)

# المتحكم (Controller) - HttpController

### الإنشاء الآلي (Generation)

يمكنك استخدام أداة سطر الأوامر `drogon_ctl` لإنشاء ملفات مصدرية لفئة متحكم مخصصة قائمة على `HttpController` بسرعة بصيغة الأمر كالتالي:

```shell
drogon_ctl create controller -h <[namespace::]class_name>
```

دعنا ننشئ فئة متحكم واحدة باسم `User` تحت نطاق الأسماء (Namespace) `demo::v1`:

```shell
drogon_ctl create controller -h demo::v1::User
```

كما ترى، تم إضافة ملفين جديدين في المجلد الحالي: demo_v1_User.h و demo_v1_User.cc.

الملف demo_v1_User.h كالتالي:

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
    // استخدم METHOD_ADD لإضافة دالة المعالجة المخصصة هنا؛
    // METHOD_ADD(User::get, "/{2}/{1}", Get); // المسار هو /demo/v1/User/{arg2}/{arg1}
    // METHOD_ADD(User::your_method_name, "/{1}/{2}/list", Get); // المسار هو /demo/v1/User/{arg1}/{arg2}/list
    // ADD_METHOD_TO(User::your_method_name, "/absolute/path/{1}/{2}/list", Get); // المسار هو /absolute/path/{arg1}/{arg2}/list

    METHOD_LIST_END
    // قد يكون الإعلان عن دالة المعالجة الخاصة بك بالشكل التالي:
    // void get(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, int p1, std::string p2);
    // void your_method_name(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, double p1, int p2) const;
};
}
}
```

الملف demo_v1_User.cc كالتالي:

```c++
#include "demo_v1_User.h"

using namespace demo::v1;

// أضف تعريف دالة المعالجة الخاصة بك هنا
```

### الاستخدام (Usage)

دعنا نعدل الملفين:

الملف demo_v1_User.h بعد التعديل:

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
    // استخدم METHOD_ADD لإضافة دالة المعالجة المخصصة هنا؛
    METHOD_ADD(User::login,"/token?userId={1}&passwd={2}",Post);
    METHOD_ADD(User::getInfo,"/{1}/info?token={2}",Get);
    METHOD_LIST_END
    // قد يكون الإعلان عن دالة المعالجة الخاصة بك بالشكل التالي:
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

الملف demo_v1_User.cc بعد التعديل:

```c++
#include "demo_v1_User.h"

using namespace demo::v1;

// أضف تعريف دالة المعالجة الخاصة بك هنا

void User::login(const HttpRequestPtr &req,
                 std::function<void (const HttpResponsePtr &)> &&callback,
                 std::string &&userId,
                 const std::string &password)
{
    LOG_DEBUG<<"User "<<userId<<" login";
    // خوارزمية المصادقة، قراءة قاعدة البيانات، التحقق، التعرف، إلخ...
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

    // التحقق من صحة التوكن (token)، إلخ.
    // قراءة قاعدة البيانات أو الـ Cache للحصول على معلومات المستخدم
    Json::Value ret;
    ret["result"]="ok";
    ret["user_name"]="Jack";
    ret["user_id"]=userId;
    ret["gender"]=1;
    auto resp=HttpResponse::newHttpJsonResponse(ret);
    callback(resp);
}
```

يمكن لكل فئة `HttpController` تعريف العديد من معالجات طلبات HTTP. ونظراً لأن عدد الدوال قد يكون كبيراً بلا حدود، فمن غير المنطقي التغلب عليها عبر التحميل الزائد للدوال الوهمية (Virtual functions overload). لذا نحتاج لتسجيل المعالج نفسه (وليس الفئة) في إطار العمل.

- #### تعيين المسارات (Path Mapping)

  يتم التعيين (Mapping) من مسار URL إلى المعالج عبر الماكرو (Macro). يمكنك إضافة تعيينات مسارات متعددة باستخدام الماكرو `METHOD_ADD` أو الماكرو `ADD_METHOD_TO`. يجب وضع جميع جمل `METHOD_ADD` و `ADD_METHOD_TO` بين ماكرو `METHOD_LIST_BEGIN` وماكرو `METHOD_LIST_END`.

  يقوم الماكرو `METHOD_ADD` تلقائياً بإضافة بادئة تتضمن نطاق الأسماء واسم الفئة في خريطة المسار. بناءً عليه، في هذا المثال، يتم تسجيل الدالة login على المسار `/demo/v1/user/token`، والدالة getInfo على المسار `/demo/v1/user/xxx/info`. القيود مماثلة تماماً لماكرو `PATH_ADD` في HttpSimpleController ولا حاجة لإعادة شرحها هنا.

  عند استخدام الماكرو `ADD_METHOD` وكانت الفئة تنتمي إلى نطاق أسماء معين، يجب إضافة نطاق الأسماء هذا إلى عنوان URL للوصول. في هذا المثال، نستخدم `http://localhost/demo/v1/user/token?userid=xxx&passwd=xxx` أو `http://localhost/demo/v1/user/xxxxx/info?token=xxxx`.

  يقوم الماكرو `ADD_METHOD_TO` بنفس عمل الماكرو السابق تقريباً، باستثناء أنه لا يضيف أي بادئات تلقائياً، أي أن المسار المسجل عبر هذا الماكرو يكون مساراً مطلقاً (Absolute path).

  نلاحظ أن `HttpController` يقدم آلية تعيين مسارات أكثر مرونة - حيث يمكننا وضع مجموعة من الدوال ذات الصلة داخل فئة واحدة.

  بالإضافة إلى ذلك، يمكنك ملاحظة أن الماكرو يوفر طريقة لتعيين المعاملات (Parameter mapping). حيث يمكننا تعيين معاملات الاستعلام (Query parameters) الموجودة على المسار إلى قائمة المعاملات للدالة. يتوافق ترتيب المعاملات في مسار URL مع موقع معامل الدالة، وهذا أمر مريح للغاية. جميع الأنواع الشائعة التي يمكن التحويل إليها من نوع النصوص (String) يمكن استخدامها كمعاملات للدالة (مثل std::string و int و float و double وغيرها)، وسيتكفل إطار عمل Drogon بتحويل النوع تلقائياً نيابة عنك. هذا سهل ومريح جداً للتطوير. لاحظ أن المراجع بقيمة يسارية (Lvalue references) يجب أن تكون من النوع الثابت `const`.

  يمكن تعيين المسار نفسه عدة مرات بشروط مختلفة، وذلك بالتمييز بينها عبر طريقة HTTP المستخدمة، وهو أمر قانوني وتطبيق شائع جداً في تصميم Restful API، مثل:

  ```c++
  METHOD_LIST_BEGIN
      METHOD_ADD(Book::getInfo,"/{1}?detail={2}",Get);
      METHOD_ADD(Book::newBook,"/{1}",Post);
      METHOD_ADD(Book::deleteOne,"/{1}",Delete);
  METHOD_LIST_END
  ```

  يمكن كتابة النماذج النائبة (Placeholders) لمعاملات المسار بعدة طرق:

  - `{}`: موقعه على المسار يحدد موقع معامل الدالة، مما يشير إلى أن معامل المسار يُعين إلى المعامل المقابل في معالج الدالة.
  - `{1},{2}`: معاملات المسار التي تحتوي على رقم بداخلها تُعين إلى معاملات المعالج المحددة بذلك الرقم.
  - `{anystring}`: النصوص المكتوبة هنا ليس لها تأثير فعلي، ولكنها تحسن من قابلية قراءة الكود. وهي تكافئ `{}`.
  - `{1:anystring},{2:xxx}`: الرقم قبل النقطتين يعبر عن الموقع، والنص بعد النقطتين ليس له تأثير ولكن يحسن قابلية القراءة. وهي تكافئ `{1}` و `{2}`.

  يُنصح باستخدام الطريقتين الأخيرتين، وإذا كانت معاملات المسار ومعاملات الدالة بنفس الترتيب، فإن الطريقة الثالثة كافية جداً. من السهل ملاحظة أن الأمثلة التالية متكافئة تماماً:

  - "/users/{}/books/{}"
  - "/users/{}/books/{2}"
  - "/users/{user_id}/books/{book_id}"
  - "/users/{1:user_id}/books/{2}"

  > **ملاحظة: مطابقة المسار ليست حساسة لحالة الأحرف (Case insensitive)، بينما أسماء المعاملات حساسة لحالة الأحرف. أما قيم المعاملات فيمكن إرسالها بحروف كبيرة أو صغيرة وتُمرر كما هي دون تغيير إلى المتحكم.**

- #### تعيين المعاملات (Parameter Mapping)

  من الوصف السابق، نعلم أن المعاملات على المسار ومعاملات الاستعلام بعد علامة الاستفهام يمكن تعيينها إلى قائمة معاملات دالة المعالجة. يلزم أن يستوفي نوع المعامل الهدف الشروط التالية:

  - يجب أن يكون نوع قيمة (Value type)، أو مرجع ثابت لقيمة يسارية (Const left value reference)، أو مرجع غير ثابت لقيمة يمنية (Non-const right value reference). ولا يمكن أن يكون مرجع غير ثابت لقيمة يسارية (Non-const left value reference). يُفضل استخدام مرجع قيمة يمنية (Right value reference) ليتسنى للمستخدم التصرف فيه بحرية.

  - الأنواع الأساسية مثل int و long و long long و unsigned long و unsigned long long و float و double و long double وغيرها يمكن استخدامها كأنواع معاملات.

  - std::string

  - أي نوع يمكن إسناده باستخدام معامل الاستخراج `stringstream >>`.

  > **بالإضافة إلى ذلك، يوفر إطار عمل Drogon أيضاً آلية تعيين من كائن HttpRequestPtr إلى أي نوع من المعاملات.** عندما يكون عدد معاملات التعيين في قائمة معاملات المعالج الخاص بك أكبر من عدد المعاملات المتاحة على المسار، فسيتم تحويل المعاملات الإضافية عبر كائن HttpRequestPtr. يمكن للمستخدم تعريف أي نوع تحويل. وطريقة تعريف هذا التحويل هي تخصص قالب `fromRequest` (المعرف في ملف الترويسة HttpRequest.h) داخل نطاق الأسماء `drogon`؛ على سبيل المثال، لنفترض أننا بحاجة لإعداد واجهة RESTful لإنشاء مستخدم جديد، فنقوم بتعريف هيكل المستخدم كالتالي:

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

  باستخدام التعريف وتخصص القالب أعلاه، يمكننا تعريف خريطة المسار والمعالج كالتالي:

  ```c++
  class UserController:public drogon::HttpController<UserController>
  {
  public:
      METHOD_LIST_BEGIN
          // استخدم METHOD_ADD لإضافة دالة المعالجة المخصصة هنا؛
          ADD_METHOD_TO(UserController::newUser,"/users",Post);
      METHOD_LIST_END
      // قد يكون الإعلان عن دالة المعالجة الخاصة بك بالشكل التالي:
      void newUser(const HttpRequestPtr &req,
                  std::function<void (const HttpResponsePtr &)> &&callback,
                  myapp::User &&pNewUser) const;
  };
  ```

  يمكن ملاحظة أن المعامل الثالث من النوع `myapp::User` ليس له عنصر نائب مناظر على مسار التعيين، فيعتبره إطار العمل معاملاً محولاً من الكائن `req` ويحصل على هذا المعامل عبر دالة قالب التخصص الخاصة بالمستخدم. هذا مريح جداً للمطورين.

  علاوة على ذلك، فإن بعض المطورين لا يحتاجون إلى الوصول إلى كائن HttpRequestPtr إلا من أجل الحصول على بيانات النوع المخصص لديهم. يمكنهم وضع الكائن المخصص في موقع المعامل الأول، وسيقوم إطار العمل بإتمام عملية التعيين بجميع خطواتها بشكل صحيح مثل المثال أعلاه. ويمكن كتابتها أيضاً بالطريقة التالية:

  ```c++
  class UserController:public drogon::HttpController<UserController>
  {
  public:
      METHOD_LIST_BEGIN
          // استخدم METHOD_ADD لإضافة دالة المعالجة المخصصة هنا؛
          ADD_METHOD_TO(UserController::newUser,"/users",Post);
      METHOD_LIST_END
      // قد يكون الإعلان عن دالة المعالجة الخاصة بك بالشكل التالي:
      void newUser(myapp::User &&pNewUser,
                  std::function<void (const HttpResponsePtr &)> &&callback) const;
  };
  ```

- #### تعيين المسارات المتعددة (Multiple Path Mapping)

  يدعم Drogon استخدام التعبيرات النمطية (Regular expressions) في تعيين المسارات، والتي يمكن استخدامها خارج الأقواس المزخرفة '{}'. على سبيل المثال:

  ```c++
  ADD_METHOD_TO(UserController::handler1,"/users/.*",Post); /// يطابق أي مسار يبدأ بـ `/users/`
  ADD_METHOD_TO(UserController::handler2,"/{name}/[0-9]+",Post); /// يطابق أي مسار يتكون من نص اسم ورقم.
  ```

- #### تعيين التعبيرات النمطية (Regular Expressions Mapping)

  الطريقة أعلاه ذات دعم محدود للتعبيرات النمطية. إذا أراد المستخدمون استخدام التعبيرات النمطية بحرية كاملة، يوفر Drogon ماكرو `ADD_METHOD_VIA_REGEX` لتحقيق ذلك، مثل:

  ```c++
  ADD_METHOD_VIA_REGEX(UserController::handler1,"/users/(.*)",Post); /// يطابق أي مسار يبدأ بـ `/users/` ويعين باقي المسار إلى معامل في handler1.
  ADD_METHOD_VIA_REGEX(UserController::handler2,"/.*([0-9]*)",Post); /// يطابق أي مسار ينتهي برقم ويعين ذلك الرقم إلى معامل في handler2.
  ADD_METHOD_VIA_REGEX(UserController::handler3,"/(?!data).*",Post); /// يطابق أي مسار لا يبدأ بـ '/data'
  ```

  كما يمكن ملاحظته، يمكن أيضاً إجراء تعيين المعاملات باستخدام التعبيرات النمطية، وسيتم تعيين جميع النصوص التي تطابقها التعبيرات الفرعية إلى معاملات المعالج بالترتيب.

  > **ينبغي الانتباه إلى أنه عند استخدام التعبيرات النمطية، يجب توخي الحذر لمنع تعارض المطابقة (مطابقة عدة معالجات مختلفة لنفس المسار). عند حدوث تعارضات داخل نفس المتحكم، ينفذ Drogon المعالج الأول فقط (الذي تم تسجيله أولاً في إطار العمل). وعند حدوث تعارضات بين متحكمات مختلفة، يكون من غير المؤكد أي معالج سيتم تنفيذه. لذلك يتوجب على المستخدمين تجنب هذه التعارضات.**

# التالي: [WebSocketController](/ARA/ARA-04-3-Controller-WebSocketController)

</div>
