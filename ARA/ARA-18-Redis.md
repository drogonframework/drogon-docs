<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-17-Redis) - [English](/ENG/ENG-18-Redis)

# خادم البيانات Redis

يدعم إطار العمل Drogon التعامل مع Redis، وهو خادم تخزين بيانات عالي السرعة يعمل في الذاكرة العشوائية (In-memory data store)، ويُستخدم بشكل واسع كمخزن مؤقت لقواعد البيانات (Database cache) أو كمرسل ومستقبل للرسائل (Message broker). وكعادة كل شيء في Drogon، فإن اتصالات Redis تتم بطريقة لامتزامنة (Asynchronous)، مما يضمن تشغيل Drogon بقدرة تزامن عالية جداً حتى تحت ظروف الضغط والإجهاد الفائق.

يعتمد دعم Redis داخل Drogon على مكتبة `hiredis` المساعدة. وبالتالي لن يتوفر دعم Redis إذا لم تكن مكتبة hiredis مثبتة ومتاحة أثناء تجميع إطار العمل.

### إنشاء العميل (Creating a client)

يمكن إنشاء كائنات عملاء Redis واسترجاعها برمجياً عبر الكائن العام `app()`:

```c++
app().createRedisClient("127.0.0.1", 6379);
...
// بعد استدعاء app.run()
RedisClientPtr redisClient = app().getRedisClient();
```

كما يمكن إنشاء عملاء Redis وتكوينهم عبر ملف الإعدادات `config.json`:

```json
    "redis_clients": [
        {
            //name: اسم العميل، والقيمة الافتراضية هي 'default'
            //"name":"",
            //host: عنوان IP الخاص بالخادم، والقيمة الافتراضية هي 127.0.0.1
            "host": "127.0.0.1",
            //port: منفذ الخادم، والقيمة الافتراضية هي 6379
            "port": 6379,
            //passwd: كلمة المرور، افتراضياً فارغة ''
            "passwd": "",
            //db: رقم مؤشر قاعدة البيانات داخل Redis، وافتراضياً هو 0
            "db": 0,
            //is_fast: القيمة الافتراضية false. إذا كانت true يصبح العميل أسرع، ولكن لا يمكن استدعاء أي واجهة متزامنة منه ولا يمكن استخدامه خارج خيوط الإدخال/الإخراج وخيط التنفيذ الرئيسي
            "is_fast": false,
            //number_of_connections: القيمة الافتراضية 1. إذا كان is_fast بـ true يمثل هذا الرقم عدد الاتصالات لكل خيط إدخال/إخراج، وإلا فهو إجمالي عدد جميع الاتصالات
            "number_of_connections": 1,
            //timeout: القيمة الافتراضية -1.0 بالثواني، وتمثل مهلة تنفيذ الأمر. القيمة الصفرية أو السالبة تعني عدم وجود مهلة زمنية
            "timeout": -1.0
        }
    ]
```

### استخدام Redis (Using Redis)

تنفذ الواجهة `execCommandAsync` أوامر Redis بطريقة لامتزامنة. وتأخذ 3 معاملات على الأقل، المعاملان الأول والثاني عبارة عن دالتي استدعاء عكسي (Callbacks) تُستدعيان عند نجاح أمر Redis أو فشله، والمعامل الثالث هو نص الأمر نفسه. يمكن أن يكون النص صيغة تنسيق مطابقة للغة C (C-style format string)، وتكون باقي المعاملات هي القيم المراد إدراجها داخل نص التنسيق. على سبيل المثال، لتحديد مفتاح باسم `name` بقيمة `drogon`:

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {},
    [](const std::exception &err) {
        LOG_ERROR << "حدث خطأ ما! " << err.what();
    },
    "set name drogon");
```

أو ضبط المفتاح `myid` بقيمة `587d-4709-86e4` باستعمال نصوص التنسيق:

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {},
    [](const std::exception &err) {
        LOG_ERROR << "حدث خطأ ما! " << err.what();
    },
    "set myid %s", "587d-4709-86e4");
```

ويمكن استغلال الواجهة نفسها `execCommandAsync` لاسترجاع البيانات وقراءتها من Redis:

```c++
redisClient->execCommandAsync(
    [](const drogon::nosql::RedisResult &r) {
        if (r.type() == RedisResultType::kNil)
            LOG_INFO << "تعذر العثور على متغير مرتبط بالمفتاح 'name'";
        else
            LOG_INFO << "الاسم هو " << r.asString();
    },
    [](const std::exception &err) {
        LOG_ERROR << "حدث خطأ ما! " << err.what();
    },
    "get name");
```

### المعاملات والعمليات المركبة (Transaction)

تتيح المعاملات (Transactions) في Redis تنفيذ أوامر متعددة في خطوة واحدة مجمعة. تُنفذ جميع الأوامر داخل المعاملة بالترتيب، ولا يمكن لأي أوامر مبعوثة من عملاء آخرين أن تتخلل أو تتداخل **في منتصف** عملية تنفيذ المعاملة. لاحظ أن المعاملة هنا ليست ذرية بالكامل (Not atomic)، وهذا يعني أنه بعد استلام أمر التنفيذ `exec` ستبدأ المعاملة في الدخول لصلب التنفيذ، وإذا فشل أي أمر فردي داخل المعاملة، فستستمر باقي الأوامر الأخرى في التنفيذ دون توقف، حيث لا تدعم معاملات Redis عمليات التراجع الإلغائي (Rollback).

تنشئ الواجهة `newTransactionAsync` معاملة جديدة، وتُستخدم هذه المعاملة تماماً مثل كائن `RedisClient` العادي. وفي النهاية تقوم الدالة `RedisTransaction::execute` بتأكيد وتنفيذ المعاملة المذكورة:

```c++
redisClient->newTransactionAsync([](const RedisTransactionPtr &transPtr) {
    transPtr->execCommandAsync(
        [](const drogon::nosql::RedisResult &r) { /* نجح هذا الأمر الفردي */ },
        [](const std::exception &err) { /* فشل هذا الأمر الفردي */ },
    "set name drogon");

    transPtr->execute(
        [](const drogon::nosql::RedisResult &r) { /* نجحت المعاملة بالكامل */ },
        [](const std::exception &err) { /* فشلت المعاملة */ });
});
```

### الروتينات الفرعية المشتركة (Coroutines)

يدعم عملاء Redis الروتينات الفرعية المشتركة. ينبغي استخدام مجمع GCC 11 أو إصدار أحدث، واستعمال `cmake -DCMAKE_CXX_FLAGS="-std=c++20"` لتفعيلها. راجع دليل [الروتينات الفرعية المشتركة (Coroutines)](/ARA/ARA-17-Coroutines) لمزيد من التفاصيل.

```c++
try
{
    auto transaction = co_await redisClient->newTransactionCoro();
    co_await transaction->execCommandCoro("set zzz 123");
    co_await transaction->execCommandCoro("set mening 42");
    co_await transaction->executeCoro();
}
catch(const std::exception& e)
{
    LOG_ERROR << "فشل اتصال Redis: " << e.what();
}
```

# التالي: [إطار عمل الاختبارات (Testing Framework)](/ARA/ARA-19-Testing-Framework)

</div>
