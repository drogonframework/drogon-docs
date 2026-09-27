<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-19-测试框架) - [English](/ENG/ENG-19-Testing-Framework)

## إطار عمل الاختبارات (Testing Framework)

يُعد `DrogonTest` إطار عمل اختبارات خفيف ومدمج داخل Drogon لتمكين إجراء الاختبارات اللامتزامنة والمتزامنة بسهولة. يُستخدم الإطار في اختبارات الوحدة (Unittests) واختبارات التكامل (Integration tests) الخاصة بـ Drogon نفسه، ولكن يمكن استخدامه أيضاً لاختبار التطبيقات المبنية باستخدام Drogon. القواعد البرمجية لـ DrogonTest مستوحاة من إطاري [GTest](https://github.com/google/googletest) و [Catch2](https://github.com/catchorg/Catch2).

لست ملزماً باستخدام DrogonTest لتطبيقك، إذ يمكنك استخدام أي إطار عمل ترتاح له، ولكنه يظل خياراً ممتازاً ومتاحاً.

### الاختبارات الأساسية (Basic testing)

لنبدأ بمثال بسيط: لديك دالة متزامنة تحسب مجموع الأعداد الطبيعية حتى قيمة معينة، وتريد اختبار صحتها.

```c++
// إخبار DrogonTest بتوليد دالة `test::run()`. تُعرف هذه القيمة في الملف الرئيسي فقط
#define DROGON_TEST_MAIN
#include <drogon/drogon_test.h>

int sum_all(int n)
{
    int result = 1;
    for(int i=2;i<n;i++) result += i;
    return result;
}

DROGON_TEST(Sum)
{
    CHECK(sum_all(1) == 1);
    CHECK(sum_all(2) == 3);
    CHECK(sum_all(3) == 6);
}

int main(int argc, char** argv)
{
    return drogon::test::run(argc, argv);
}
```

بعد التجميع والتشغيل... حسناً، لقد اجتاز الاختبار، لكن هناك خطأ واضح، أليس كذلك؟ كان ينبغي أن تكون نتيجة `sum_all(0)` هي 0. يمكننا إضافة ذلك إلى اختبارنا:

```c++
DROGON_TEST(Sum)
{
    CHECK(sum_all(0) == 0);
    CHECK(sum_all(1) == 1);
    CHECK(sum_all(2) == 3);
    CHECK(sum_all(3) == 6);
}
```

الآن يفشل الاختبار مع الرسالة التالية:

```text
In test case Sum
↳ /path/to/your/test/main.cc:47  FAILED:
  CHECK(sum_all(0) == 0)
With expansion
  1 == 0
```

لاحظ أن إطار العمل قام بطباعة حالة الاختبار التي فشلت مع القيمة الفعلية على طرفي التعبير، مما يسمح لنا برؤية ما حدث فوراً. والحل بسيط:

```c++
int sum_all(int n)
{
    int result = 0;
    for(int i=1;i<n;i++) result += i;
    return result;
}
```

### أنواع التأكيدات (Types of assertions)

يأتي DrogonTest مع مجموعة متنوعة من التأكيدات والإجراءات. يفحص الماكرو الأساسي `CHECK()` ببساطة ما إذا كان التعبير يُقيم إلى `true`، وإذا لم يكن كذلك، يقوم بالطباعة على وحدة التحكم (Console). أما `CHECK_THROWS()` فيفحص ما إذا كان التعبير يطلق استثناءً، وإذا لم يطلق، يُطبع على وحدة التحكم.. إلخ. من ناحية أخرى، يفحص `REQUIRE()` ما إذا كان التعبير صحيحاً، فإن لم يكن، يقوم بعمل `return` فوراً لمنع تنفيذ التعبيرات التالية بعد الاختبار.

| الإجراء عند الفشل / التعبير | صحيح (`true`) | يطلق استثناءً | لا يطلق استثناءً | يطلق نوعاً معيناً من الاستثناءات |
| ------------------------- | ---------- | ----------------- | ------------------ | -------------------- |
| لا شيء (متابعة) | `CHECK` | `CHECK_THROWS` | `CHECK_NOTHROW` | `CHECK_THROWS_AS` |
| الإرجاع (`return`) | `REQUIRE` | `REQUIRE_THROWS` | `REQUIRE_NOTHROW` | `REQUIRE_THROWS_AS` |
| إرجاع الروتين (`co_return`) | `CO_REQUIRE` | `CO_REQUIRE_THROWS` | `CO_REQUIRE_NOTHROW` | `CO_REQUIRE_THROWS_AS` |
| إنهاء العملية (Kill process) | `MANDATE` | `MANDATE_THROWS` | `MANDATE_NOTHROW` | `MANDATE_THROWS_AS` |

لنقتبس مثالاً عملياً بسيطاً: لنفترض أنك تختبر ما إذا كان محتوى ملف ما يطابق ما تتوقعه. لا فائدة من الاستمرار في الاختبار إذا فشل البرنامج في فتح الملف. لذا، يمكننا استخدام `REQUIRE` لاختصار وتقليل الكود المكرر.

```c++
DROGON_TEST(TestContent)
{
    std::ifstream in("data.txt");
    REQUIRE(in.is_open());
    // بدلاً من كتابة:
    // CHECK(in.is_open() == true);
    // if(in.is_open() == false)
    //    return;

    ...
}
```

بالمثل، يعمل `CO_REQUIRE` مثل `REQUIRE` تماماً، ولكنه مخصص للروتينات الفرعية المشتركة (Coroutines). كما يمكن استخدام `MANDATE` عندما تفشل عملية ما وتؤدي إلى تعديل حالة عامة (Global state) غير قابلة للاسترداد، حيث يكون الشيء المنطقي الوحيد حينها هو إيقاف الاختبار بالكامل.

### الاختبارات اللامتزامنة (Asynchronous testing)

بما أن Drogon هو إطار عمل شبكي لامتزامن، فمن الطبيعي أن يدعم DrogonTest اختبار الدوال اللامتزامنة. يتتبع DrogonTest سياق الاختبار عبر المتغير `TEST_CTX`، وكل ما عليك هو التقاط المتغير **بالقيمة (by value)**. على سبيل المثال، اختبار ما إذا كانت واجهة برمجة تطبيقات بعيدة (Remote API) تنجح وترجع كائن JSON:

```c++
DROGON_TEST(RemoteAPITest)
{
    auto client = HttpClient::newHttpClient("http://localhost:8848");
    auto req = HttpRequest::newHttpRequest();
    req->setPath("/");
    client->sendRequest(req, [TEST_CTX](ReqResult res, const HttpResponsePtr& resp) {
        // لا يمكننا فعل شيء إذا لم يصل الطلب إلى الخادم
        // أو إذا أنشأ الخادم بيانات تالفة.
        REQUIRE(res == ReqResult::Ok);
        REQUIRE(resp != nullptr);

        CHECK(resp->getStatusCode() == k200OK);
        CHECK(resp->contentType() == CT_APPLICATION_JSON);
    });
}
```

يجب تغليف الروتينات الفرعية المشتركة داخل `AsyncTask` أو استدعاؤها عبر `sync_wait` نظراً لعدم وجود دعم أصلي للروتينات والتوافق مع C++14/17 في إطار عمل الاختبارات.

```c++
DROGON_TEST(RemoteAPITestCoro)
{
    auto api_test = [TEST_CTX]() {
        auto client = HttpClient::newHttpClient("http://localhost:8848");
        auto req = HttpRequest::newHttpRequest();
        req->setPath("/");

        auto resp = co_await client->sendRequestCoro(req);
        CO_REQUIRE(resp != nullptr);
        CHECK(resp->getStatusCode() == k200OK);
        CHECK(resp->contentType() == CT_APPLICATION_JSON);
    };

    sync_wait(api_test());
}
```

### بدء حلقة أحداث Drogon (Starting Drogon's event loop)

تتطلب بعض الاختبارات تشغيل حلقة أحداث Drogon (Event loop). على سبيل المثال، تعمل عملاء HTTP على حلقة الأحداث العامة لـ Drogon ما لم يُحدد غير ذلك. يتعامل الهيكل النمطي التالي مع العديد من الحالات الحرجية ويضمن تشغيل حلقة الأحداث قبل بدء أي اختبار:

```c++
int main(int argc, char** argv)
{
    std::promise<void> p1;
    std::future<void> f1 = p1.get_future();

    // بدء حلقة الأحداث الرئيسية على خيط تنفيذ آخر
    std::thread thr([&]() {
        // إدراج الوعد في قائمة الانتظار ليتم تحقيقه بعد بدء حلقة الأحداث
        app().getLoop()->queueInLoop([&p1]() { p1.set_value(); });
        app().run();
    });

    // لا يتحقق المستقبل إلا بعد بدء حلقة الأحداث
    f1.get();
    int status = drogon::test::run(argc, argv);

    // إرسال أمر لإيقاف حلقة الأحداث والانتظار
    app().getLoop()->queueInLoop([]() { app().quit(); });
    thr.join();
    return status;
}
```

### التكامل مع CMake (CMake integration)

مثل معظم إطارات عمل الاختبار، يمكن لـ DrogonTest التكامل مع CMake. تقوم الدالة `ParseAndAddDrogonTests` بإضافة الاختبارات التي تراها في ملف المصدر إلى إطار عمل CTest الخاص بـ CMake.

```cmake
find_package(Drogon REQUIRED) # يتولى أيضاً تحميل ParseAndAddDrogonTests
add_executable(mytest main.cpp)
target_link_libraries(mytest PRIVATE Drogon::Drogon)
ParseAndAddDrogonTests(mytest)
```

الآن يمكن تشغيل الاختبار عبر نظام البناء (Makefile في هذه الحالة):

```bash
❯ make test
Running tests...
Test project path/to/your/test/build/
      Start  1: Sum
 1/1  Test  #1: Sum ....................................   Passed    0.00 sec
```

# التالي: [قواعد بيانات SQLite3 (SQLite3)](/ARA/ARA-20-SQLite3)

</div>
