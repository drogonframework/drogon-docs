<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-03-快速開始) - [English](/ENG/ENG-03-Quick-Start)

# بداية سريعة (Quick Start)

## موقع استاتيكي (Static Site)

دعنا نبدأ بمثال بسيط يقدم كيفية استخدام Drogon. في هذا المثال، سننشئ مشروعاً باستخدام أداة سطر الأوامر `drogon_ctl`:

```shell
drogon_ctl create project your_project_name
```

ستجد العديد من المجلدات المفيدة المثبتة بالفعل داخل مجلد المشروع:

<div class="rtl-code">

```console
┤── build                         مجلد البناء التجميعي (Build folder)
┤── CMakeLists.txt                ملف إعدادات cmake للمشروع
┤── config.json                   ملف إعدادات تطبيق Drogon
┤── controllers                   المجلد المخصص لحفظ ملفات المصدر للمتحكمات (Controllers)
┤── filters                       المجلد المخصص لحفظ ملفات الفلاتر (Filters)
┤── main.cc                       البرنامج الرئيسي
┤── models                        مجلد ملفات نماذج قواعد البيانات (Models)
│   ┘── model.json
┘── views                         المجلد المخصص لحفظ ملفات العرض csp (Views)
```

</div>

يمكن للمستخدمين وضع مختلف الملفات (مثل المتحكمات، الفلاتر، العروض، وغيرها) داخل المجلدات المخصصة لها. ولمزيد من السهولة وتقليل الأخطاء، نوصي بشدة أن ينشئ المستخدمون مشاريع تطبيقات الويب الخاصة بهم باستخدام الأمر `drogon_ctl`. لمزيد من التفاصيل، راجع دليل [drogon_ctl](/ARA/ARA-12-drogon_ctl-Command).

دعنا نلقي نظرة على الملف الرئيسي main.cc:

```c++
#include <drogon/HttpAppFramework.h>
int main() {
    // تحديد عنوان ومنفذ مستمع HTTP
    drogon::app().addListener("0.0.0.0",80);
    // تحميل ملف الإعدادات
    //drogon::app().loadConfigFile("../config.json");
    // تشغيل إطار عمل HTTP، وتتوقف هذه الدالة للانتقال إلى حلقة الأحداث الداخلية (Event loop)
    drogon::app().run();
    return 0;
}
```

ثم قم ببناء مشروعك كما يلي:

```shell
cd build
cmake ..
make
```

بعد اكتمال التجميع، قم بتشغيل الملف التنفيذي `./your_project_name`.

الآن، سنضيف ملفاً استاتيكياً واحداً باسم index.html إلى مسار الجذر لـ HTTP:

```shell
echo '<h1>Hello Drogon!</h1>' >>index.html
```

مسار الجذر التلقائي هو `"./"`، ولكن يمكن تغييره عبر ملف config.json. لمزيد من التفاصيل، راجع دليل [ملف الإعدادات](/ARA/ARA-11-Configuration-File). يمكنك حينها زيارة هذه الصفحة عبر الرابط `"http://localhost"` أو `"http://localhost/index.html"` (أو عبر عنوان IP الخاص بالخادم الذي يعمل عليه تطبيق الويب).

![مرحباً Drogon!](https://drogonframework.github.io/drogon-docs/images/hellodrogon.png)

إذا لم يتمكن الخادم من العثور على الصفحة التي طلبتها، فسينتهي بك المطاف إلى صفحة الخطأ 404:
![صفحة 404](https://drogonframework.github.io/drogon-docs/images/notfound.png)

> **ملاحظة: تأكد من أن جدار الحماية (Firewall) في خادمك يراعي السماح بالمنفذ 80، وإلا فلن تتمكن من رؤية هذه الصفحات. (وهناك طريقة أخرى وهي تغيير المنفذ من 80 إلى 1024 أو أحدث لتجنب ظهور رسالة الخطأ التالية):**

```console
FATAL Permission denied (errno=13) , Bind address failed at 0.0.0.0:80 - Socket.cc:67
```

يمكننا نسخ مجلدات وملفات أي موقع استاتيكي إلى مجلد بدء التشغيل الخاص بتطبيق الويب الشغال هذا، ثم يمكننا الوصول إليها عبر المتصفح. أنواع الملفات المدعومة في Drogon تلقائياً هي:

- html
- js
- css
- xml
- xsl
- txt
- svg
- ttf
- otf
- woff2
- woff
- eot
- png
- jpg
- jpeg
- gif
- bmp
- ico
- icns

يوفر Drogon أيضاً واجهات برمجية لتغيير أنواع الملفات المسموح بها. للتفاصيل، يرجى الرجوع إلى [واجهة برمجة تطبيقات HttpAppFramework API](API-HttpAppFramework).

## موقع ديناميكي (Dynamic Site)

دعنا نرى كيف نضيف متحكمات (Controllers) إلى هذا التطبيق، ونجعل المتحكم يستجيب بالمعطيات.

يمكن للمستخدم استخدام أداة سطر الأوامر drogon_ctl لتوليد ملفات المصدر الخاصة بالمتحكم. دعنا ننفذ الأمر داخل مجلد `controllers`:

```shell
drogon_ctl create controller TestCtrl
```

كما ترى، تم إنشاء ملفين جديدين: TestCtrl.h و TestCtrl.cc.

محتوى الملف TestCtrl.h كالتالي:

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
    // اذكر تعريفات المسارات هنا؛
    //PATH_ADD("/path","filter1","filter2",HttpMethod1,HttpMethod2...);
    PATH_LIST_END
};
```

ومحتوى الملف TestCtrl.cc كالتالي:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void (const HttpResponsePtr &)> &&callback)
{
    // اكتب منطق التطبيق الخاص بك هنا
}
```

دعنا نعدل الملفين لنحضر المتحكم للرد باستجابة بسيطة وهي "Hello World!".

الملف TestCtrl.h بعد التعديل:

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
    // اذكر تعريفات المسارات هنا

    // مثال
    //PATH_ADD("/path","filter1","filter2",HttpMethod1,HttpMethod2...);

    PATH_ADD("/",Get,Post);
    PATH_ADD("/test",Get);
    PATH_LIST_END
};
```

نستخدم `PATH_ADD` لربط المسارين `'/'` و `'/test'` بالدوال المعالجة، وإضافة قيود المسار الاختيارية (هنا تم تحديد طرق HTTP المسموح بها).

الملف TestCtrl.cc بعد التعديل:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void (const HttpResponsePtr &)> &&callback)
{
    // اكتب منطق التطبيق الخاص بك هنا
    auto resp=HttpResponse::newHttpResponse();
    // ملاحظة: الثابت المسمى أدناه في الفئة هو "k200OK" (بمعنى 200 OK)، وليس "k2000K".
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody("Hello World!");
    callback(resp);
}
```

أعد تجميع هذا المشروع باستخدام CMake، ثم شغل الملف التنفيذي `./your_project_name`:

```shell
`cd ../build`  
`cmake ..`  
`make`  
`./your_project_name`  
```

اكتب `"http://localhost/"` أو `"http://localhost/test"` في شريط عنوان المتصفح، وستشاهد عبارة "Hello World!" على المتصفح.

> **ملاحظة: إذا كان خادمك يحتوي على موارد استاتيكية وديناميكية معاً، فإن Drogon يعطي الأولوية للموارد الديناميكية أولاً. في هذا المثال، تكون الاستجابة لطلب `GET http://localhost/` هي `Hello World!` (القادمة من ملف المتحكم `TestCtrl`) بدلاً من `Hello Drogon!` (القادمة من الملف الاستاتيكي index.html).**

نلاحظ أن إضافة متحكم إلى التطبيق عملية بسيطة للغاية. كل ما عليك فعله هو إضافة ملف المصدر المقابل. حتى الملف الرئيسي لا يحتاج إلى أي تعديل. هذا التصميم ضعيف الاقتران (Loosely coupled design) فعال جداً لتطوير تطبيقات الويب.

> **ملاحظة: لا يفرض Drogon أي قيود على موقع ملفات المصدر للمتحكمات. يمكنك أيضاً حفظها في "./" (المجلد الرئيسي للمشروع)، أو حتى تعريف مجلد جديد داخل `CMakeLists.txt`. ويُفضل استخدام مجلد controllers لتسهيل عملية الإدارة.**

# التالي: [المتحكم (Controller) - مقدمة](/ARA/ARA-04-0-Controller-Introduction)

</div>
