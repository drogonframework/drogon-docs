<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-04-1-控制器-HttpSimpleController) - [English](/ENG/ENG-04-1-Controller-HttpSimpleController)

# المتحكم (Controller) - HttpSimpleController

يمكنك استخدام أداة سطر الأوامر `drogon_ctl` لإنشاء ملفات مصدرية لفئة متحكم مخصصة قائمة على `HttpSimpleController` بسرعة. تكون صيغة الأمر كالتالي:

```shell
drogon_ctl create controller <[namespace::]class_name>
```

دعنا ننشيء فئة متحكم واحدة باسم `TestCtrl`:

```shell
drogon_ctl create controller TestCtrl
```

كما ترى، تم إنشاء ملفين جديدين: TestCtrl.h و TestCtrl.cc. الآن، دعنا نلقي نظرة عليهما:

الملف TestCtrl.h:

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

الملف TestCtrl.cc:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void (const HttpResponsePtr &)> &&callback)
{
    // اكتب منطق التطبيق الخاص بك هنا
}
```

كل فئة تعتمد على HttpSimpleController يمكنها تعريف معالج طلبات HTTP واحد فقط، ويتم تحديده عبر تجاوز دالة وهمية (Virtual function override).

يتم تحديد التوجيه (Route) أو ما يُسمى التعيين (Mapping) من مسار URL إلى المعالج عبر الماكرو (Macro). يمكنك إضافة تعيينات مسارات متعددة باستخدام الماكرو `PATH_ADD`. يجب وضع جميع جمل `PATH_ADD` بين ماكرو `PATH_LIST_BEGIN` وماكرو `PATH_LIST_END`.

المعامل الأول هو المسار المراد تعيينه، وتُعد المعاملات التي تلي المسار قيوداً على هذا المسار. حالياً، يتم دعم نوعين من القيود: الأول هو النوع التعدادي `HttpMethod` والذي يُحدد طرق HTTP المسموح بها. والنوع الآخر هو اسم فئة `HttpFilter`. يمكن للمستخدم إعداد أي عدد من هذين النوعين من القيود، ولا توجد أي اشتراطات لترتيبها. بالنسبة للفلاتر (Filters)، يُرجى الرجوع إلى دليل [البرمجيات الوسيطة والفلاتر (Middleware and Filter)](/ARA/ARA-05-Middleware-and-Filter).

يمكن للمستخدمين تسجيل نفس المتحكم البسيط (Simple Controller) على مسارات متعددة، أو تسجيل متحكمات بسيطة متعددة على نفس المسار (باستخدام طرق HTTP مختلفة).

يمكنك تعريف متغير من نوع الفئة HttpResponse، ثم استخدام `callback()` لإرجاعه:

```c++
    // اكتب منطق التطبيق الخاص بك هنا
    auto resp=HttpResponse::newHttpResponse();
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody("Your Page Contents");
    callback(resp);
```

> **يتم تعيين المسار المذكور أعلاه إلى المعالج أثناء وقت التجميع (Compile time). في الواقع، يوفر إطار عمل Drogon أيضاً واجهة لإتمام عملية التعيين أثناء وقت التشغيل (Runtime). يتيح التعيين أثناء وقت التشغيل للمستخدم تعيين أو تعديل التعيينات عبر ملفات الإعدادات أو واجهات مستخدم أخرى دون الحاجة لإعادة تجميع البرنامج (لدواعي الأداء، يُحظر إضافة أي تعيين للمتحكمات بعد تشغيل الدالة `app().run()`).**

# التالي: [HttpController](/ARA/ARA-04-2-Controller-HttpController)

</div>
