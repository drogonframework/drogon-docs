<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-17-协程) - [English](/ENG/ENG-17-Coroutines)

# الروتينات الفرعية المشتركة (Coroutines)

يدعم Drogon [روتينات C++ الفرعية المشتركة (Coroutines)][1] بدءاً من الإصدار 1.4. وتوفر هذه الروتينات طريقة لتسطيح تسلسل التحكم (Control flow) للاستدعاءات اللامتزامنة، أي الهروب من كابوس الاستدعاءات العكسية المتداخلة (Callback hell). وبفضلها، تصبح البرمجة اللامتزامنة بسهولة وسلاسة البرمجة المتزامنة نفسها.

### المصطلحات (Terminology)

هذه الصفحة ليست مخصصة لشرح مفهوم الروتينات الفرعية المشتركة أو كيفية عملها من الصفر، بل لتوضيح كيفية استخدامها داخل Drogon. تميل المصطلحات المعتادة إلى التداخل لأن الدوال العادية (Subroutines) تستخدم المصطلحات نفسها التي تستخدمها الروتينات الفرعية المشتركة، على الرغم من وجود اختلاف طفيف في المعنى بينهما. كما أن حقيقة أن روتينات C++ الفرعية المشتركة يمكنها التصرف كالدوال العادية تسهم في هذا التداخل. ولتقليل اللبس، سنعتمد المصطلحات التالية (وهي ليست مثالية تماماً، لكنها كافية وواضحة):

* **الروتين الفرعي المشترك (Coroutine):** هو دالة يمكنها إيقاف التنفيذ مؤقتاً (Suspend) ثم استئنافه لاحقاً (Resume).
* **الإرجاع (Return):** يعني إنهاء الدالة لتنفيذها وإعطاء قيمة مرتجعة للمستدعي، أو إنشاء الروتين الفرعي المشترك لكائن *قابل للاستئناف* (Resumable) يمكن استخدامه لاستئناف عمل الروتين.
* **التوليد/الإخراج (Yield):** يحدث عندما ينتج الروتين الفرعي المشترك نتيجة لصالح المستدعي.
* **co-return:** تعني أن الروتين الفرعي المشترك يُخرج النتيجة ثم يخرج ويتم عمله.
* **(co-)awaiting:** تعني أن خيط التنفيذ (Thread) ينتظر إخراج النتيجة من الروتين الفرعي المشترك. ويكون إطار العمل حراً في استغلال خيط التنفيذ لأغراض أخرى أثناء الانتظار.

### تفعيل الروتينات الفرعية المشتركة (Enabling coroutines)

ميزة الروتينات الفرعية المشتركة في Drogon محتواة بالكامل داخل ملفات الترويسة فقط (Header-only). وهذا يعني أنه يمكن للتطبيق استخدام الروتينات حتى لو تم بناء Drogon نفسه بدون دعمها. وتعتمد طريقة التفعيل على المجمع المستخدم (Compiler):
* في مجمع GCC (الإصدار 10 فما فوق)، يمكن تفعيلها عبر ضبط الخيارين `-std=c++20 -fcoroutines`.
* مع MSVC (تم اختباره على MSVC 19.25)، يمكن تفعيلها باستعمال `/std:c++latest` ويجب **عدم** ضبط `/await`.

لاحظ أن تطبيق Drogon للروتينات الفرعية المشتركة لن يعمل على مجمع Clang (حتى إصدار Clang 12.0). يقوم GCC 11 بتفعيل الروتينات افتراضياً عند تفعيل C++20. ورغم أن GCC 10 يستطيع تجميع الروتينات، إلا أنه يحتوي على خطأ في المجمع يؤدي إلى عدم تحرير إطارات الروتينات المتداخلة (Nested coroutine frames)، مما قد يسبب تسريباً في الذاكرة (Memory leak).

### استخدام الروتينات الفرعية المشتركة (Using coroutines)

ينتهي اسم كل روتين فرعي مشترك في Drogon باللاحقة `Coro`. على سبيل المثال، تتحول `db->execSqlSync()` إلى `db->execSqlCoro()`، وتتحول `client->sendRequest()` إلى `client->sendRequestCoro()`، وهكذا. تُرجع جميع الروتينات الفرعية المشتركة كائناً *قابلاً للانتظار* (Awaitable object). ثم يؤدي استخدام `co_await` على هذا الكائن إلى الحصول على القيمة المباشرة. ويكون إطار العمل حراً في استخدام خيط التنفيذ لمعالجة عمليات الإدخال/الإخراج والمهام الأخرى أثناء انتظار وصول النتائج — وهذه هي روعة الروتينات الفرعية المشتركة، حيث يبدو الكود متزامناً بالعين، ولكنه ينفذ لامتزامناً في الواقع.

على سبيل المثال، الاستعلام عن عدد المستخدمين الموجودين في قاعدة البيانات:

```cpp
app.registerHandler("/num_users",
    [](HttpRequestPtr req, std::function<void(const HttpResponsePtr&)> callback) -> Task<>
    //                                  يجب تحديد نوع الإرجاع ككائن قابل للاستئناف ^^^
{
    auto sql = app().getDbClient();
    try
    {
        auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
        size_t num_users = result[0][0].as<size_t>();
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(std::to_string(num_users));
        callback(resp);
    }
    catch(const DrogonDbException &err)
    {
        // تعمل الاستثناءات مثل الواجهات المتزامنة تماماً
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(err.base().what());
        callback(resp);
    }
    // لا داعي لإرجاع أي شيء! هذا روتين فرعي مشترك يخرج `void`
    // وهو ما يشار إليه بنوع الإرجاع Task<void>
    co_return; // إذا أردت استخدامها (ليست اختيارية وليست إجبارية)، استخدم co_return
}
```

لاحظ بعض النقاط الهامة:

1. أي معالج يستدعي روتيناً فرعياً مشتركاً **يجب** أن يُرجع كائناً *قابلاً للاستئناف* (Resumable)، مما يحول المعالج نفسه إلى روتين فرعي مشترك.
2. تحل `co_return` محل `return` العادية داخل الروتين الفرعي المشترك.
3. تُمرر معظم المعاملات بالقيمة (By value).

الكائن *القابل للاستئناف* (Resumable) هو كائن يتبع معيار الروتينات الفرعية المشتركة. لا تقلق كثيراً بشأن التفاصيل، فقط اعلم أنه إذا أردت أن يخرج الروتين قيمة من النوع `T`، فإن نوع الإرجاع سيكون `Task<T>`.

إن تمرير معظم المعاملات بالقيمة هو نتيجة مباشرة لكون الروتينات لامتزامنة، فمن المستحيل تتبع متى ينتهي نطاق المرجع (Reference) لأن الكائن قد يُدمر أثناء انتظار الروتين الفرعي المشترك، أو قد يعيش المرجع على خيط تنفيذ آخر فتحدث تدميرات غير متوقعة أثناء التنفيذ.

من المنطقي إلغاء استخدام الاستدعاء العكسي (Callback) والاستعاضة عنه باستخدام `co_return` المباشرة. وهذا الأمر مدعوم، ولكنه قد يتسبب في انخفاض الإنتاجية بنسبة تصل إلى 8% تحت ظروف معينة. يُرجى أخذ هذا الانخفاض في الأداء بعين الاعتبار وتحديد ما إذا كان مرتبطاً بحالة الاستخدام لديك. ونقدم المثال نفسه مجدداً:

```cpp
app.registerHandler("/num_users",
    [](HttpRequestPtr req) -> Task<HttpResponsePtr>)
    //                     أصبح يرجع استجابة الآن ^^^
{
    auto sql = app().getDbClient();
    try
    {
        auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
        size_t num_users = result[0][0].as<size_t>();
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(std::to_string(num_users));
        co_return resp;
    }
    catch(const DrogonDbException &err)
    {
        // تعمل الاستثناءات مثل الواجهات المتزامنة تماماً
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(err.base().what());
        co_return resp;
    }
}
```

استدعاء الروتينات الفرعية المشتركة من متحكمات مقابس الويب (WebSocket controllers) غير مدعوم حالياً. إذا كنت بحاجة إلى هذه الميزة، يمكنك فتح تذكرة (Issue) على مستودع المشروع.

### أخطاء شائعة (Common pitfalls)

هناك بعض الأخطاء الشائعة التي قد تقع فيها عند استخدام الروتينات الفرعية المشتركة:

- #### إطلاق الروتينات مع التقاط دوال لامبدا من دالة (Launching coroutines with lambda capture from a function)

  تمتلك التقاطات تعبيرات لامبدا (Lambda captures) والروتينات الفرعية المشتركة دورات حياة منفصلة، حيث يعيش الروتين حتى يُدمر إطار الروتين (Coroutine frame)، بينما تُدمر دوال لامبدا عادة فور استدعائها مباشرة. وبسبب الطبيعة اللامتزامنة، يمكن أن تكون دورة حياة الروتين أطول بكثير من دالة لامبدا (كما في عمليات تنفيذ استعلامات SQL مثلاً). وتُدمر دالة لامبدا فور انتظار الاستعلام (والعودة إلى حلقة الأحداث لمعالجة أحداث أخرى)، بينما يظل إطار الروتين ينتظر استجابة SQL. وبالتالي ستكون دالة لامبدا قد دُمرت بالفعل عند اكتمال الاستعلام.

  بدلاً من كتابة كود خاطئ كالتالي:

  ```cpp
  app().getLoop()->queueInLoop([num] -> AsyncTask {
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE last_login < CURRENT_TIMESTAMP - INTERVAL $1 DAY", std::to_string(num));
      // كائن لامبدا والتقاطاته تُدمر فوراً عند الانتظار. لقد دُمرت عند هذه النقطة
      LOG_INFO << "Remove old customers that have no activity for more than " << num << "days"; // خطأ استخدام الذاكرة بعد تحريرها (use-after-free)
  });
  // كود سيئ، وسيؤدي للانهيار (Crash)
  ```

  يوفر Drogon الدالة `async_func` التي تغلف دالة لامبدا لضمان استمرار دورة حياتها:

  ```cpp
  app().getLoop()->queueInLoop(async_func([num] -> Task<void> {
  //                             ^^^^^^^^^^^^^^^^^^^^^^^^^ التغليف باستخدام async_func وإرجاع Task<>
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE last_login < CURRENT_TIMESTAMP - INTERVAL $1 DAY", std::to_string(num));
      LOG_INFO << "Remove old customers that have no activity for more than " << num << "days";
  }));
  // كود جيد وسليم
  ```

- #### تمرير/التقاط المراجع إلى الروتينات الفرعية المشتركة من دالة (Passing/capturing references into coroutines from function)

  من الممارسات الجيدة في C++ تمرير الكائنات بالمراجع لتجنب النسخ غير الضروري. ومع ذلك، فإن تمرير الكائنات بالمراجع إلى روتين فرعي مشترك من دالة عادية يسبب مشاكل شائعة. والسبب هو أن الروتين لامتزامن ويمكن أن يمتلك دورة حياة أطول بكثير من الدالة العادية. على سبيل المثال، الكود التالي سينهار:

  ```cpp
  void removeCustomers(const std::string& customer_id)
  {
      async_run([&customer_id] {
          //      ^^^^ لا تقم بتمرير/التقاط الكائنات بالمراجع داخل روتين فرعي مشترك
          // إلا إذا كنت متأكداً أن الكائن يتملك دورة حياة أطول من الروتين نفسه

          auto db = app().getDbClient();
          co_await db->execSqlCoro("DELETE FROM customers WHERE customer_id = $1", customer_id);
          // ينتهي نطاق `customer_id` فور انتظار استعلام SQL. ينهار البرنامج هنا
          co_await db->execSqlCoro("DELETE FROM orders WHERE customer_id = $1", customer_id);
      });
  }
  ```

  ومع ذلك، يُعد تمرير الكائنات كمراجع **من روتين فرعي مشترك آخر** ممارسة جيدة ومستحسنة:

  ```cpp
  Task<> removeCustomers(const std::string& customer_id)
  {
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE customer_id = $1", customer_id);
      co_await db->execSqlCoro("DELETE FROM orders WHERE customer_id = $1", customer_id);
  }

  Task<> findUnwantedCustomers()
  {
      auto db = app().getDbClient();
      auto list = co_await db->execSqlCoro("SELECT customer_id from customers "
          "WHERE customer_score < 5;");
      for(const auto& customer : list)
          co_await removeCustomers(customer["customer_id"].as<std::string>());
          //                               ^^^^^^^^^^^^^^^^^
          // هذا الأمر ممتاز ومفضل تماماً رغم أنه مرجع ثابت (const reference)
          // لأننا نستدعيه من داخل روتين فرعي مشترك آخر
  }
  ```

[1]: https://en.cppreference.com/w/cpp/language/coroutines

# التالي: [خادم البيانات Redis (Redis)](/ARA/ARA-18-Redis)

</div>
