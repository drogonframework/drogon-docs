<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-08-1-数据库-DbClient) - [English](/ENG/ENG-08-1-Database-DbClient)

# قاعدة البيانات (Database) - عميل قاعدة البيانات (DbClient)

### إنشاء كائن DbClient (DbClient Object Construction)

توجد طريقتان لإنشاء كائن DbClient. الأولى هي عبر الدوال الساكنة (Static methods) للفئة DbClient. ويمكنك رؤية التعريف في ملف الترويسة DbClient.h كالتالي:

```c++
#if USE_POSTGRESQL
    static std::shared_ptr<DbClient> newPgClient(const std::string &connInfo, const size_t connNum);
#endif
#if USE_MYSQL
    static std::shared_ptr<DbClient> newMysqlClient(const std::string &connInfo, const size_t connNum);
#endif
```

تُستخدم الواجهة أعلاه للحصول على المؤشر الذكي (Smart pointer) لكائن تنفيذ DbClient. المعامل `connInfo` هو نص الاتصال (Connection string)، حيث تُحدد فيه سلسلة من معاملات الاتصال بصيغة `key=value` (ولمزيد من التفاصيل، يُرجى الرجوع إلى التعليقات في ملف الترويسة). أما المعامل `connNum` فهو عدد اتصالات قاعدة البيانات التي يديرها DbClient، وله تأثير جوهري على التزامن، لذا يُرجى ضبطه وفقاً للحاجة الفعلية.

الكائن الناتج بالطريقة أعلاه يتوجب على المطور إيجاد طريقة للابقاء عليه طوال فترة التشغيل (Persist)، مثل وضعه في حاوية عامة (Global container). **أما إنشاء كائن مؤقت ثم تحريره بعد الاستخدام فهو حل غير موصى به إطلاقاً** للأسباب التالية:

- سينتج عن ذلك هدر الوقت في إنشاء الاتصالات وإغلاقها، مما يزيد من زمن التأخير (Latency) في النظام.
- الواجهة هي واجهة لامُعَطِّلة (Non-blocking interface) أي أنه عندما يحصل المطور على كائن DbClient، فإن الاتصال الذي يديره الكائن لا يكون قد استُقر بنجاح بعد. وإطار العمل لا يوفر (عمداً) دالة استدعاء لاحق للتأكد من نجاح إنشاء الاتصال. فهل سيتعين عليك إيقاف التنفيذ مؤقتاً (Sleep) قبل البدء بالاستعلام؟! هذا يتنافى تماماً مع الهدف الأساسي للإطار اللامتزامن.

بناءً عليه، ينبغي إنشاء كائنات DbClient في بداية تشغيل البرنامج والاحتفاظ بها واستخدامها طوال دورة حياة التطبيق. ومن الواضح أن هذه المهمة يمكن إنجازها بالكامل بواسطة إطار العمل. لذا يوفر إطار عمل Drogon طريقة الإنشاء الثانية، والتي تتم عبر ملف الإعدادات أو عبر الدالة `createDbClient()`. ولمعرف طريقة إعداد ملف التكوين، راجع دليل [db_clients](/ARA/ARA-11-Configuration-File?id=db_clients).

عند الحاجة، يتم الحصول على المؤشر الذكي لـ DbClient عبر واجهة إطار العمل التالية:

```c++
orm::DbClientPtr getDbClient(const std::string &name = "default");
```

المعامل `name` هو قيمة خيار الإعداد `name` في ملف التكوين لتمييز كائنات DbClient المختلفة داخل التطبيق نفسه. الاتصالات التي يديرها DbClient تقوم بإعادة الاتصال تلقائياً باستمرار، لذا لا داعي للقلق بشأن حالة الاتصال حيث تكون متصلة دائماً تقريباً. **ملاحظة**: لا يمكن استدعاء هذه الدالة قبل تشغيل `app.run()`, وإلا فسيحصل المطور على `shared_ptr` فارغ.

### واجهات التنفيذ (Execution Interface)

يوفر DbClient عدة واجهات مختلفة للمطورين، كما هو موضح أدناه:

```c++
/// طريقة لامتزامنة (Asynchronous method)
template <
        typename FUNCTION1,
        typename FUNCTION2,
        typename... Arguments>
void execSqlAsync(const std::string &sql,
                  FUNCTION1 &&rCallback,
                  FUNCTION2 &&exceptCallback,
                  Arguments &&... args) noexcept;

/// طريقة لامتزامنة عبر 'future'
template <typename... Arguments>
std::future<const Result> execSqlAsyncFuture(const std::string &sql,
                                             Arguments &&... args) noexcept;

/// طريقة متزامنة (Synchronous method)
template <typename... Arguments>
const Result execSqlSync(const std::string &sql,
                         Arguments &&... args) noexcept(false);

/// طريقة النمط المتدفق (Streaming-type method)
internal::SqlBinder operator<<(const std::string &sql);
```

ونظراً لأن عدد ونوع المعاملات المربوطة (Bound parameters) لا يمكن تحديده مسبقاً، فإن هذه الطرق عبارة عن قوالب دوال (Function templates).

توضح الخصائص الخاصة بهذه الطرق في الجدول التالي:

| الطريقة | متزامنة / لامتزامنة (Synchronous/Asynchronous) | مُعَطِّلة / لامُعَطِّلة (Blocking/Non-blocking) | الاستثناءات (Exceptions) |
| :---: | :---: | :---: | :---: |
| <div class="ltr-content left-align p-0">void execSqlAsync</div> | لامتزامنة (Asynchronous) | لامُعَطِّلة (Non-blocking) | لن تطلق أي استثناء (Will not throw an exception) |
| <div class="ltr-content left-align p-0">std::future<const Result\> execSqlAsyncFuture</div> | لامتزامنة (Asynchronous) | تُعَطِّل عند استدعاء الدالة get لـ future | قد تطلق استثناءً عند استدعاء الدالة get لـ future |
| <div class="ltr-content left-align p-0">const Result execSqlSync</div> | متزامنة (Synchronous) | مُعَطِّلة (Blocking) | قد تطلق استثناءً (May throw an exception) |
| <div class="ltr-content left-align p-0">internal::SqlBinder operator<<</div> | لامتزامنة (Asynchronous) | لامُعَطِّلة افتراضياً | لن تطلق أي استثناء |

قد يتسبب الجمع بين "لامتزامنة" و"مُعَطِّلة" في بعض الحيرة وبشكل عام فإن الطرق المتزامنة التي تتضمن الإدخال/الإخراج عبر الشبكة (Network IO) تكون مُعَطِّلة (Blocking)، بينما الطرق اللامتزامنة تكون لامُعَطِّلة (Non-blocking). ومع ذلك، يمكن للطرق اللامتزامنة أن تعمل في النمط المُعَطِّل أيضاً، مما يعني أنها ستعطل خيط التنفيذ حتى تنتهي دالة الاستدعاء اللاحق من التنفيذ. وعندما تعمل الطريقة اللامتزامنة لـ DbClient في النمط المُعَطِّل، سيتم تنفيذ دالة الاستدعاء اللاحق في خيط التنفيذ الخاص بالمستدعي، ثم تعود الدالة.

إذا كان تطبيقك يتضمن سيناريوهات التزامن الفائق (High-concurrency)، يُرجى استخدام الطرق اللامتزامنة اللامُعَطِّلة (Asynchronous non-blocking methods). أما إذا كان التطبيق في سيناريو تزامن منخفض (مثل صفحة إدارة أجهزة الشبكة)، فيمكنك اختيار الطرق المتزامنة لسهولتها ووضوحها المباشر.

- #### <div class="ltr-content">execSqlAsync</div>

  ```c++
  template <typename FUNCTION1,
          typename FUNCTION2,
          typename... Arguments>
  void execSqlAsync(const std::string &sql,
                  FUNCTION1 &&rCallback,
                  FUNCTION2 &&exceptCallback,
                  Arguments &&... args) noexcept;
  ```

  هذه هي الواجهة اللامتزامنة الأكثر استخداماً، وتعمل في النمط اللامُعَطِّل (Non-blocking mode).

  المعامل `sql` هو نص جملة SQL. وإذا كانت هناك عناصر نائبة (Placeholders) لربط المعاملات، استخدم قواعد العناصر النائبة الخاصة بقاعدة البيانات المناظرة. على سبيل المثال، العناصر النائبة في PostgreSQL هي `$1, $2 ...`، بينما في MySQL تكون `?` بدون أرقام.

  المعامل غير المحدد `args` يمثل المعاملات المربوطة، ويمكن أن يكون صفراً أو أكثر. ويطابق عدد المعاملات عدد العناصر النائبة في جملة SQL. ويمكن أن تكون الأنواع كالتالي:

  - الأنواع الصحيحة (Integer types): يمكن أن تكون أعداداً صحيحة بأطوال مختلفة، ويجب أن تطابق نوع الحقل في قاعدة البيانات.
  - أنواع الأعداد العائمة (Floating point types): يمكن أن تكون `float` أو `double`، ويجب أن تطابق نوع الحقل في قاعدة البيانات.
  - الأنواع النصية (String types): يمكن أن تكون `std::string` أو `const char[]`، وتناظر النوع النصي في قاعدة البيانات أو الأنواع الأخرى التي يمكن تمثيلها بنصوص.
  - أنواع التاريخ (Date types): نوع `trantor::Date`، ويناظر أنواع التاريخ والوقت والختم الزمني (datetime, date, timestamp) في قاعدة البيانات.
  - الأنواع الثنائية (Binary types): نوع `std::vector<char>`، ويناظر نوع `bytea` في PostgreSQL أو نوع `blob` في MySQL.

  يمكن تمرير هذه المعاملات كقيم يسارية (Lvalues) أو قيم يمينية (Rvalues)، وتستوعب المتغيرات أو الثوابت المباشرة، وللمطور الحرية في استخدامها.

  المعاملان `rCallback` و `exceptCallback` يمثلان دالة الاستدعاء اللاحق للنتيجة ودالة الاستدعاء اللاحق للاستثناءات على التوالي، ولهما تعريف ثابت كالتالي:

  - دالة الاستدعاء اللاحق للنتيجة: نوع الاستدعاء هو `void (const Result &)`، ويمكن تمرير الكائنات القابلة للاستدعاء المطابقة لهذا النوع مثل `std::function` و `lambda` وغيرها.
  - دالة الاستدعاء اللاحق للاستثناءات: نوع الاستدعاء هو `void (const DrogonDbException &)`، ويمكن تمرير مختلف الكائنات القابلة للاستدعاء المطابقة لهذا النوع.

  بعد نجاح تنفيذ جملة SQL، يتم تغليف نتيجة التنفيذ بواسطة الفئة `Result` وتمريرها للمطور عبر دالة الاستدعاء اللاحق للنتيجة، وإذا حدث أي استثناء أثناء التنفيذ، تُنفذ دالة الاستدعاء اللاحق للاستثناءات، ويمكن للمطور الحصول على معلومات الاستثناء من كائن `DrogonDbException`.

  إليك مثالاً توضيحياً:

  ```c++
  auto clientPtr = drogon::app().getDbClient();
  clientPtr->execSqlAsync("select * from users where org_name=$1",
                              [](const drogon::orm::Result &result) {
                                  std::cout << result.size() << " rows selected!" << std::endl;
                                  int i = 0;
                                  for (auto row : result)
                                  {
                                      std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
                                  }
                              },
                              [](const DrogonDbException &e) {
                                  std::cerr << "error:" << e.base().what() << std::endl;
                              },
                              "default");
  ```

  من المثال يتضح أن الكائن `Result` عبارة عن حاوية متوافقة مع معايير مكتبة C++ القياسية (std)، وتدعم المكررات (Iterators)، ويمكن الحصول على كائن كل صف عبر حلقة نطاق (Range loop). ولمعرفة الواجهات المختلفة لكائنات `Result` و `Row` و `Field` يُرجى الرجوع إلى الكود المصدري.

  الفئة `DrogonDbException` هي الفئة الأساسية لجميع استثناءات قاعدة البيانات. يُرجى الرجوع إلى التعليقات في الكود المصدري.

- #### <div class="ltr-content">execSqlAsyncFuture</div>

  ```c++
  template <typename... Arguments>
  std::future<const Result> execSqlAsyncFuture(const std::string &sql,
                                              Arguments &&... args) noexcept;
  ```

  تتغاضى الواجهة اللامتزامنة القائمة على future عن المعاملين الخاصين بالاستدعاء اللاحق في الواجهة السابقة. واستدعاء هذه الواجهة يعيد كائن `future` فوراً. ويجب على المطور استدعاء الدالة `get()` لكائن future للحصول على النتيجة المعادة. أما الاستثناءات فيتم التعامل معها عبر آلية `try/catch`. وإذا لم تكن الدالة `get()` داخل كتلة `try/catch` ولم تكن هناك كتلة `try/catch` في مكدس الاستدعاء بالكامل، فسينتهي البرنامج ويخرج عند حدوث استثناء في تنفيذ SQL.

  على سبيل المثال:

  ```c++
  auto f = clientPtr->execSqlAsyncFuture("select * from users where org_name=$1",
                                      "default");
  try
  {
      auto result = f.get(); // التعطيل حتى الحصول على النتيجة أو التقاط الاستثناء.
      std::cout << result.size() << " rows selected!" << std::endl;
      int i = 0;
      for (auto row : result)
      {
          std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
      }
  }
  catch (const DrogonDbException &e)
  {
      std::cerr << "error:" << e.base().what() << std::endl;
  }
  ```

- #### <div class="ltr-content">execSqlSync</div>

  ```c++
  template <typename... Arguments>
  const Result execSqlSync(const std::string &sql,
                          Arguments &&... args) noexcept(false);
  ```

  الواجهة المتزامنة (Synchronous interface) هي الأبسط والأكثر وضوحاً، حيث تأخذ نص SQL والمعاملات المربوطة كمدخلات، وتُعيد كائن `Result`. ويعطل الاستدعاء خيط التنفيذ الحالي (Blocking)، كما يطلق استثناءً عند حدوث خطأ، لذا يجب الانتباه للالتقاط باستخدام `try/catch`.

  مثال:

  ```c++
  try
  {
      auto result = clientPtr->execSqlSync("update users set user_name=$1 where user_id=$2",
                                          "test",
                                          1); // التعطيل حتى الحصول على النتيجة أو التقاط الاستثناء.
      std::cout << result.affectedRows() << " rows updated!" << std::endl;
  }
  catch (const DrogonDbException &e)
  {
      std::cerr << "error:" << e.base().what() << std::endl;
  }
  ```

- #### <div class="ltr-content">operator<<</div>

  ```c++
  internal::SqlBinder operator<<(const std::string &sql);
  ```

  واجهة النمط المتدفق (Streaming interface) لها طبيعة خاصة حيث تدعم إدخال جملة SQL والمعاملات بالتتابع عبر المعامل `<<`، وتحدد دالة الاستدعاء اللاحق للنتيجة ودالة الاستدعاء اللاحق للاستثناءات عبر المعامل `>>`. على سبيل المثال، يمكن إعادة كتابة مثال الاستعلام السابق باستخدام النمط المتدفق كالتالي:

  ```c++
  *clientPtr  << "select * from users where org_name=$1"
              << "default"
              >> [](const drogon::orm::Result &result)
                  {
                      std::cout << result.size() << " rows selected!" << std::endl;
                      int i = 0;
                      for (auto row : result)
                      {
                          std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
                      }
                  }
              >> [](const DrogonDbException &e)
                  {
                      std::cerr << "error:" << e.base().what() << std::endl;
                  };
  ```

  هذا الاستخدام يكافئ تماماً الواجهة اللامتزامنة اللامُعَطِّلة الأولى، ويعتمد اختيار أي منهما على تفضيلات المطور. وإذا أردت أن تعمل في النمط المُعَطِّل، يمكنك استخدام `<<` لإدخال المعامل `Mode::Blocking`.

  بالإضافة إلى ذلك، توفر واجهة النمط المتدفق استخداماً خاصاً، فباستخدام دالة استدعاء لاحق خاصة للنتيجة، يمكن لإطار العمل تمرير النتيجة إلى المطور صَفاً صَفاً (Row by row). ونوع الاستدعاء لهذه الدالة يكون كالتالي:

  ```c++
  void (bool,Arguments...);
  ```

  عندما يكون المعامل المنطقي (bool parameter) الأول قيمته `true`، فإن ذلك يعني أن النتيجة عبارة عن صف فارغ، أي أن جميع النتائج قد أُعيدت بالكامل، وهذا هو الاستدعاء اللاحق الأخير.
  وتأتي بعد ذلك سلسلة من المعاملات المناظرة لقيمة كل عمود في سجل الصف، ويقوم إطار العمل بتحويل الأنواع تلقائياً، مع مراعاة مطابقة الأنواع من قبل المطور. ويمكن أن تكون هذه الأنواع مراجع ثابتة لقيم يسارية (Const lvalue references)، أو مراجع لقيم يمينية (Rvalue references)، أو أنواع قيم مباشرة.

  دعنا نعيد كتابة المثال السابق باستخدام هذه الدالة:

  ```c++
  int i = 0;
  *clientPtr  << "select user_name, user_id from users where org_name=$1"
              << "default"
              >> [&i](bool isNull, const std::string &name, int64_t id)
                      {
                      if (!isNull)
                          std::cout << i++ << ": user name is " << name << ", user id is " << id << std::endl;
                      else
                          std::cout << i << " rows selected!" << std::endl;
                      }
              >> [](const DrogonDbException &e)
                  {
                      std::cerr << "error:" << e.base().what() << std::endl;
                  };
  ```

  يمكن ملاحظة أن قيم الحقلين `user_name` و `user_id` في جملة الاستعلام تُسند مباشرة إلى المتغيرين `name` و `id` داخل دالة الاستدعاء اللاحق، دون الحاجة لمعالجة هذه التحويلات يدوياً، مما يوفر سهولة ومرونة كبيرة.

> **ملاحظة: من المهم التأكيد على أنه في البرمجة اللامتزامنة يجب على المطور الانتباه إلى المتغير `i` في المثال أعلاه. حيث يجب التأكد من صحة وسريان المتغير `i` عند حدوث الاستدعاء اللاحق لأنه يلتقط عبر المرجع (Reference). وسُتستدعى دالة الاستدعاء اللاحق في خيط تنفيذ آخر، وقد تكون البيئة الحالية (Context) قد انتهت عند حدوث الاستدعاء. وعادة ما يستخدم المبرمجون المؤشرات الذكية (Smart pointers) للاحتفاظ بالمتغيرات المؤقتة ثم التقاطها عبر الاستدعاء اللاحق لضمان سريان وصحة المتغيرات.**

### ملخص (Summary)

يمتلك كل كائن DbClient خيط تنفيذ أو أكثر من خيوط حلقات الأحداث (EventLoop threads) للتحكم في الإدخال/الإخراج لاتصالات قاعدة البيانات، واستقبال الطلبات عبر واجهة لامتزامنة أو متزامنة، وإعادة النتيجة عبر دالة الاستدعاء اللاحق.

الواجهات المُعَطِّلة لـ DbClient تقوم فقط بتعطيل خيط تنفيذ المستدعي، وطالما أن خيط تنفيذ المستدعي ليس هو خيط حلقة الأحداث (EventLoop thread)، فلن يؤثر ذلك على التشغيل الطبيعي لخيط EventLoop. وعند استدعاء دالة الاستدعاء اللاحق، يُنفذ الكود الداخلي للدالة على خيط EventLoop. لذلك، ينبغي تجنب تنفيذ أي عمليات مُعَطِّلة (Blocking operations) داخل دالة الاستدعاء اللاحق، وإلا فسيؤثر ذلك على أداء التزامن لقراءة وكتابة قاعدة البيانات. ويجب على كل من لديه دراية ببرمجة الإدخال/الإخراج اللامُعَطِّلة (Non-blocking I/O) استيعاب هذا القيد جيدا.

# التالي: [المعاملات (Transaction)](/ARA/ARA-08-2-Database-Transaction)

</div>
