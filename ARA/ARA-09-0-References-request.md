<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-09-插件) - [English](/ENG/ENG-09-0-References-request)

# مرجع الطلب (Request References)

يمثل المؤشر من نوع `HttpRequest` والمسمى عادةً `req` في أمثلة هذا التوثيق البيانات المضمنة في الطلب المستلم أو المرسل بواسطة Drogon. وفيما يلي بعض الدوال التي يمكنك من خلالها التفاعل مع هذا الكائن:

- ### `isOnSecureConnection()`

  #### الملخص:
  دالة تُعيد ما إذا كان الطلب قد تم عبر اتصال آمن (https).

  #### المدخلات:
  لا يوجد.

  #### المخرجات:
  نوع منطقي (`bool`).

- ### `getMethod()`

  #### الملخص:
  دالة تُعيد طريقة الطلب (Request method). وتعد مفيدة للتمييز بين طرق الطلب المختلفة إذا كان المعالج (Handle) الواحد يسمح بأكثر من نوع.

  #### المدخلات:
  لا يوجد.

  #### المخرجات:
  كائن طريقة الطلب `HttpMethod`.

  #### أمثلة:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      if (req->getMethod() == HttpMethod::Get) {
        // افعل شيئاً ما
      } else if (req->getMethod() == HttpMethod::Post) {
        // افعل شيئاً آخر
      }
  }
  ```

- ### `getParameter(const std::string &key)`

  #### الملخص:
  دالة تُعيد قيمة المعامل بناءً على المعرف (Key). ويتغير سلوكها بناءً على نوع الطلب سواء كان Get أو Post.

  #### المدخلات:
  معرف المعامل من النوع النصي (`string`).

  #### المخرجات:
  محتوى المعامل بصيغة نصية (`string`).

  #### أمثلة:
  في طلبات نوع Get:
  ```c++
  #include "mycontroller.h"
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // [https://mysite.com/an-path/?id=5](https://mysite.com/an-path/?id=5)
      std::string id = req->getParameter("id");
      // أو 
      long id = std::strtol(req->getParameter("id"));
  }
  ```
  أو في طلبات نوع Post:
  ```c++
  #include "mycontroller.h"
  #include <string>

  using namespace drogon;

  void mycontroller::loginHandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // يحتوي الطلب على نموذج تسجيل دخول (Form Login)
      std::string email = req->getParameter("email");
       
      std::string password = req->getParameter("password");
  }
  ```

- ### `getPath()`

  #### الدالة المماثلة:
  `path()`

  #### الملخص:
  دالة تُعيد مسار الطلب. وتكون مفيدة إذا كنت تستخدم `ADD_METHOD_VIA_REGEX` أو أنواعاً أخرى من العناوين الديناميكية (Dynamic URLs) في [المتحكم](/ARA/ARA-04-2-Controller-HttpController).

  #### المدخلات:
  لا يوجد.

  #### المخرجات:
  نص يمثل مسار الطلب.

  #### أمثلة:
  ```c++
  #include "mycontroller.h"
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // [https://mysite.com/an-path/?id=5](https://mysite.com/an-path/?id=5)
      std::string url = req->getPath();
      
      // url = /an-path/
  }
  ```

- ### `getBody()`

  #### الدالة المماثلة:
  `body()`

  #### الملخص:
  دالة تُعيد محتوى متن الطلب (Request body) إن وجد.

  #### المدخلات:
  لا يوجد.

  #### المخرجات:
  نص يمثل متن الطلب (إن وجد).

- ### `getHeader(std::string key)`

  #### الملخص:
  دالة تُعيد ترويسة الطلب (Request header) بناءً على المعرف.

  #### المدخلات:
  معرف الترويسة من النوع النصي (`string`).

  #### المخرجات:
  محتوى الترويسة بصيغة نصية.

  #### أمثلة:
  ```c++
  #include "mycontroller.h"
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      if (req->getHeader("Host") != "mysite.com") {
        // إرجاع خطأ HTTP 403
      }
  }
  ```

- ### `headers()`

  #### الملخص:
  دالة تُعيد جميع ترويسات الطلب.

  #### المدخلات:
  لا يوجد.

  #### المخرجات:
  قاموس خريطة غير مرتبة (`unordered_map`) يحتوي على الترويسات.

  #### أمثلة:
  ```c++
  #include "mycontroller.h"
  #include <unordered_map>
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      for (const std::pair<const std::string, const std::string> &header : req->headers()) {
        auto header_key = header.first;
        auto header_value = header.second;
      }
  }
  ```

- ### `getCookie()`

  #### الملخص:
  دالة تُعيد كعكة/ملف (Cookie) تعريف ارتباط الطلب بناءً على المعرف.

  #### المدخلات:
  لا يوجد.

  #### المخرجات:
  قيمة الكعكة (Cookie) بصيغة نصية.

- ### `cookies()`

  #### الملخص:
  دالة تُعيد جميع كعكات/ملفات (Cookies) تعريف ارتباط الطلب.

  #### المدخلات:
  لا يوجد.

  #### المخرجات:
  قاموس خريطة غير مرتبة (`unordered_map`) يحتوي على الكعكات (Cookies).

  #### أمثلة:
  ```c++
  #include "mycontroller.h"
  #include <unordered_map>
  #include <string>

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      for (const std::pair<const std::string, const std::string> &header : req->cookies()) {
        auto cookie_key = header.first;
        auto cookie_value = header.second;
      }
  }
  ```

- ### `getJsonObject()`

  #### الملخص:
  دالة تحول قيمة متن الطلب إلى كائن Json (عادةً في طلبات POST).

  #### المدخلات:
  لا يوجد.

  #### المخرجات:
  كائن Json.

  #### أمثلة:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::anyhandle(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // body = {"email": "test@gmail.com"}
      auto jsonData = *req->getJsonObject();

      std::string email = jsonData["email"].asString();
  }
  ```

## أمور مفيدة (Useful Things)

###### ما يلي ليس دوال تابعة لكائن طلب HTTP مباشرة، ولكنها أمور مفيدة يمكنك القيام بها لمعالجة الطلبات التي تستقبلها:

### تحليل طلبات الملفات (Parsing File Request)

```c++
#include "mycontroller.h"

using namespace drogon;

void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
    // طلبات Post فقط (نموذج ملفات File Form)

    MultiPartParser file;
    file.parse(req);

    if (file.getFiles().empty()) {
      // لم يتم العثور على ملفات
    }

    // الحصول على الملف الأول وحفظه
    const HttpFile archive = file.getFiles()[0];
    archive.saveAs("/tmp/" + archive.getFileName());
  }
```
لمزيد من المعلومات حول تحليل الملفات، راجع دليل: [معالج الملفات (File Handler)](/ARA/ARA-09-1-File-Handler).

# التالي: [معالج الملفات (File Handler)](/ARA/ARA-09-1-File-Handler)

</div>
