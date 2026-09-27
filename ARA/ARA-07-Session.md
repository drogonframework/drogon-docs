<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-07-会话) - [English](/ENG/ENG-07-Session)

# الجلسة (Session)

تُعد الجلسة (Session) مفهوماً مهماً جداً في تطبيقات الويب. وتُستخدم لحفظ حالة العميل (Client) على الخادم (Server). وعادة ما تعمل بالتكامل مع ملفات تعريف الارتباط (Cookies) الخاصة بالمتصفح، ويقدم إطار عمل Drogon دعماً كاملاً للجلسات. يقوم Drogon **بإيقاف** خيار الجلسات بشكل افتراضي، ويمكنك تفعيله أو إيقافه عبر التالي:

```c++
void disableSession();
void enableSession(const size_t timeout=0, Cookie::SameSite sameSite=Cookie::SameSite::kNull);
```

جميع الدوال أعلاه تُستدعى عبر الكائن المنفرد (Singleton) لـ `HttpAppFramework`. يُعبر معامل المهلة (timeout) عن الوقت الذي تصبح فيه الجلسة غير صالحة وتُقاس بالثواني، والقيمة الافتراضية هي 1200 ثانية. وهذا يعني أنه إذا لم يقم المستخدم بالوصول إلى تطبيق الويب لأكثر من 20 دقيقة، فإن الجلسة المناظرة ستصبح غير صالحة. وتعيين المهلة إلى 0 يعني أن Drogon سيحتفظ بجلسة المستخدم طوال فترة تشغيل التطبيق بالكامل.
أما المعامل `sameSite` فيقوم بتغيير خاصية SameSite في ترويسة استجابة HTTP المسمى Set-Cookie.

تأكد من أن العميل الخاص بك يدعم ملفات تعريف الارتباط (Cookies) قبل تفعيل ميزة الجلسة. وإلا، فإن Drogon سينشئ جلسة جديدة لكل طلب لا يحتوي على كعكة `SessionID`، مما يتسبب في استهلاك وهدر الذاكرة وموارد المعالجة.

### كائن الجلسة (Session object)

نوع كائن الجلسة في Drogon هو `drogon::Session`، وهو مشابه جداً لـ `HttpViewData`. حيث يمكنه الوصول إلى أي نوع من الكائنات عبر مفاتيح برمجية (Keywords)، كما يدعم القراءة والكتابة المتزامنة (Concurrent reading and writing). يُرجى الرجوع إلى وثائق فئة Session لمعرفة الاستخدام التفصيلي.

يقوم إطار عمل Drogon بتمرير كائن الجلسة إلى كائن `HttpRequest` ومن ثم يمرره إلى المستخدم. ويمكن للمستخدم الحصول على كائن الجلسة عبر الواجهة البرمجية التالية في فئة `HttpRequest`:

```c++
SessionPtr session() const;
```

ترجع هذه الواجهة مؤشراً ذكياً لكائن `Session`، والذي يمكن من خلاله الوصول إلى الكائنات المختلفة.

### أمثلة على استخدام الجلسات

سنضيف ميزة تتطلب دعماً للجلسات؛ على سبيل المثال، نريد الحد من معدل تكرار وصول المستخدم (Access frequency). فبعد الزيارة الأولى، إذا حاول الوصول مجدداً خلال 10 ثوانٍ، فستتم إعادة خطأ، وإلا فستُعاد استجابة بنجاح (ok). نحتاج إلى تسجيل وقت آخر زيارة في الجلسة، ثم مقارنته بوقت الزيارة الحالية لتحقيق هذه الميزة.

سننشئ فلتر لتنفيذ هذه الوظيفة، ولنفرض أن اسم الفئة هو TimeFilter، فيكون التنفيذ كالتالي:

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
            // يمكن الزيارة مجدداً بعد مرور 10 ثوانٍ؛
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

بعد ذلك نقوم بتسجيل تعبير لامبدا (Lambda expression) على المسار `/slow` ونربط به `TimeFilter` باستخدام الكود التالي:

```c++
drogon::HttpAppFramework::instance()
            .registerHandler("/slow",
                            [=](const HttpRequestPtr &req,
                                std::function<void (const HttpResponsePtr &)> &&callback)
                            {
                                Json::Value json;
                                json["result"]="ok";
                                auto resp=HttpResponse::newHttpJsonResponse(json);
                                callback(resp);
                            },
                            {Get,"TimeFilter"});
```

استدعاء واجهة إطار العمل لتفعيل الجلسة:

```c++
drogon::HttpAppFramework::instance().enableSession(1200);
```

أعد تجميع المشروع بالكامل باستخدام CMake، وتشغيل البرنامج التنفيذي webapp، ويمكنك رؤية النتيجة من خلال المتصفح.

# التالي: [قاعدة البيانات (Database)](/ARA/ARA-08-0-Database-General)

</div>
