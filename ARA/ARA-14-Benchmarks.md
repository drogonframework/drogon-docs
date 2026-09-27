<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-14-性能测试) - [English](/ENG/ENG-14-Benchmarks)

## اختبارات الأداء المرجعية (Benchmarks)

بصفته إطار عمل لتطبيقات HTTP بلغة C++، يُعد الأداء أحد أهم الجوانب التي تحظى بالاهتمام والتركيز. يقدم هذا القسم اختبارات Drogon البسيطة والإنجازات التي حققها.

### بيئة الاختبار (Test environment)

* **نظام التشغيل:** Linux CentOS 7.4.
* **الجهاز:** خادم Dell بمُعالجين اثنين Intel(R) Xeon(R) CPU E5-2670 @ 2.60GHz (بإجمالي 16 نواة معالجة و 32 خيط تنفيذ Threads).
* **الذاكرة العشوائية (RAM):** 64 جيجابايت.
* **إصدار المجمع (gcc):** 7.3.0.

### خطة الاختبار والنتائج (Test plan and results)

نظراً لرغبتنا في اختبار الأداء الصافي لإطار عمل Drogon بشكل منفصل، حرصنا على تبسيط منطق المعالجة داخل المتحكم لأقصى درجة ممكنة، حيث أنشأنا متحكماً بسيطاً من نوع `HttpSimpleController` وقمنا بتسجيله على المسار `/benchmark`. يُرجع المتحكم النص `<p>Hello, world!</p>` لأي طلب يُرسل إليه. تم ضبط عدد خيوط التنفيذ لـ Drogon على 16 خيطاً. دالة المعالجة كالتالي (ويمكنك العثور على الكود المصدري كاملاً في المسار `drogon/examples/benchmark`):

``` c++
void BenchmarkCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback)
{
    // اكتب منطق تطبيقك هنا
    auto resp = HttpResponse::newHttpResponse();
    resp->setBody("<p>Hello, world!</p>");
    resp->setExpiredTime(0);
    callback(resp);
}
```

للمقارنة، وقع الاختيار على خادم Nginx لإجراء اختبارات المقارنة المباشرة، حيث جرى كتابة وحدة مخصصة `hello_world_module` وتجميعها مباشرة مع الكود المصدري لـ Nginx. كما تم ضبط معامل `worker_processes` في Nginx على 16.

أداة الاختبار المُستخدمة هي `httpress`، وهي أداة ممتازة ومخصصة لاختبارات الإجهاد والضغط العالي لخدمات HTTP.

قمنا بضبط معاملات أداة `httpress` واختبار كل مجموعة معاملات 5 مرات متتالية، وتسجيل الحد الأقصى والحد الأدنى لعدد الطلبات المُعاملة في الثانية الواحدة (QPS). وجاءت نتائج الاختبار كالتالي:

| أمر التشغيل (Command line) | الشرح والوصف (Description) | Drogon (ألف طلب/ثانية kQPS) | Nginx (ألف طلب/ثانية kQPS) |
| :-------------------------------------------: | :--------------------------------------------------------------: | :----------: | :---------: |
| <div class="ltr-content left-align p-0">`httpress -c 100 -n 1000000 -t 16 -k -q URL`</div> | 100 اتصال، 1 مليون طلب، 16 خيط تنفيذ، مع تفعيل Keep-Alive | 561 / 552 | 330 / 329 |
| <div class="ltr-content left-align p-0">`httpress -c 100 -n 1000000 -t 12 -q URL`</div> | 100 اتصال، 1 مليون طلب، 12 خيط تنفيذ، بدون Keep-Alive | 140 / 135 | 31 / 49 |
| <div class="ltr-content left-align p-0">`httpress -c 1000 -n 1000000 -t 16 -k -q URL`</div> | 1000 اتصال، 1 مليون طلب، 16 خيط تنفيذ، مع تفعيل Keep-Alive | 573 / 565 | 333 / 327 |
| <div class="ltr-content left-align p-0">`httpress -c 1000 -n 1000000 -t 16 -q URL`</div> | 1000 اتصال، 1 مليون طلب، 16 خيط تنفيذ، بدون Keep-Alive | 155 / 143 | 52 / 50 |
| <div class="ltr-content left-align p-0">`httpress -c 10000 -n 4000000 -t 16 -k -q URL`</div> | 10000 اتصال، 4 ملايين طلب، 16 خيط تنفيذ، مع تفعيل Keep-Alive | 512 / 508 | 316 / 314 |
| <div class="ltr-content left-align p-0">`httpress -c 10000 -n 1000000 -t 16 -q URL`</div> | 10000 اتصال، 1 مليون طلب، 16 خيط تنفيذ، بدون Keep-Alive | 143 / 141 | 43 / 40 |

كما يتضح من النتائج، عند استخدام خيار Keep-Alive من جانب العميل، يستطيع Drogon معالجة أكثر من 500,000 طلب في الثانية في الحالات التي يُرسل فيها الاتصال الواحد طلبات متعددة. وتُعد هذه النتيجة ممتازة جداً. أما في الحالة التي ينشئ فيها كل طلب اتصالا مفصلاً (بدون Keep-Alive)، يتم استهلاك وقت المعالج (CPU time) في عمليات إنشاء وإغلاق اتصالات TCP المتكررة، فينخفض معدل الإنتاجية إلى 140,000 طلب في الثانية، وهو أمر منطقي ومتوقع طبيعياً.

ومن السهل إدراك التفوق الواضح لـ Drogon مقارنة بـ Nginx في الاختبارات أعلاه.

الصورة التالية هي لقطة شاشة لإحدى جلسات الاختبار المباشرة:

![نتائج الاختبار](https://drogonframework.github.io/drogon-docs/images/benchmark.png)

# التالي: [التحليل السبي للتطبيق باستخدام coz (Causal profiling with coz)](/ARA/ARA-15-Coz)

</div>
