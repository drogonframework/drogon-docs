<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-08-3-数据库-ORM) - [English](/ENG/ENG-08-3-Database-ORM)

# قاعدة البيانات (Database) - تخطيط الكائنات العلائقية (ORM)

### النموذج (Model)

لاستخدام تخطيط الكائنات العلائقية (ORM) في Drogon، تحتاج أولاً إلى إنشاء فئات النماذج (Model classes). يوفر برنامج سطر الأوامر `drogon_ctl` في Drogon القدرة على إنشاء فئات النماذج. يقرأ البرنامج معلومات الجداول من قاعدة بيانات يحددها المستخدم ويقوم تلقائياً بتوليد ملفات مصدرية متعددة لفئات النماذج بناءً على هذه المعلومات. وعندما يستخدم المطور النموذج، يُرجى تضمين ملف الترويسة المناظر.

ومن الواضح أن كل فئة نموذج (Model class) تطابق جدولاً محدداً في قاعدة البيانات، ويطابق كل كائن (Instance) من فئة النموذج سجلاً واحداً من السجلات في الجدول.

أمر إنشاء فئات النماذج كالتالي:

```shell
drogon_ctl create model <model_path>
```

المعامل الأخير هو المسار المخصص لتخزين فئات النماذج. ويجب أن يحتوي هذا المسار على ملف إعدادات باسم `model.json` لتكوين معاملات اتصال `drogon_ctl` بقاعدة البيانات. وهو ملف بصيغة JSON يدعم التعليقات. وفيما يلي مثال على ذلك:

```json
{
  "rdbms": "postgresql",
  "host": "127.0.0.1",
  "port": 5432,
  "dbname": "test",
  "user": "test",
  "passwd": "",
  "tables": [],
  "relationships": {
      "enabled": false,
      "items": []
  }
}
```

المعاملات التي يتم تكوينها تطابق ملف إعدادات التطبيق. يُرجى الرجوع إلى دليل [ملف الإعدادات](/ARA/ARA-11-Configuration-File?id=db_clients).

خيار الإعداد `tables` فريد ومخصص لإعدادات النماذج. وهو عبارة عن مصفوفة من النصوص. يمثل كل نص اسم الجدول المراد تحويله إلى فئة نموذج. وإذا كان هذا الخيار فارغاً، فستُستخدم جميع الجداول لتوليد فئات النماذج.

تم إنشاء مجلد النماذج `models` وملف `model.json` المناظر له مسبقاً في مجلد المشروع الذي تم إنشاؤه باستخدام الأمر `drogon_ctl create project`. يمكن للمطور تعديل ملف الإعدادات وإنشاء فئات النماذج باستخدام الأمر `drogon_ctl`.

### واجهة فئة النموذج (Model Class Interface)

هناك نوعان رئيسيان من الواجهات التي يستخدمها المطور مباشرة: واجهات الحصول (Getter interfaces) وواجهات التعيين (Setter interfaces).

يوجد نوعان من واجهات الحصول (Getter interfaces):

- واجهة `getColumnName` تحصل على المؤشر الذكي (Smart pointer) للحقل. والقيمة المعادة هي مؤشر بدلاً من قيمة مباشرة، وتُستخدم أساساً للحقول التي تتضمن قيماً فارغة (NULL fields). يمكن للمطور تحديد ما إذا كان الحقل فارغاً (NULL) عبر التحقق مما إذا كان المؤشر فارغاً.
- واجهة `getValueOfColumnName` تحضر القيمة الفعلية كما يتضح من اسمها. ولأسباب تتعلق بالكفاءة والأداء، تعيد الواجهة مرجعاً ثابتاً (Const reference). وإذا كان الحقل المناظر فارغاً (NULL)، تعيد الواجهة القيمة الافتراضية المحددة عبر معامل الدالة.

بالإضافة إلى ذلك، يمتلك نوع الكتل الثنائية (blob, bytea) واجهة خاصة وهي `getValueOfColumnNameAsString` تحمل البيانات الثنائية داخل كائن `std::string` وتعدها للمطور.

تُستخدم واجهة التعيين (Setter interface) لتعيين قيمة الحقل المناظر `setColumnName`، وتتطابق أنواع المعاملات مع أنواع الحقول. ولا تمتلك الحقول التي تُولد تلقائياً (مثل المفاتيح التلقائية الزيادة Auto-incrementing primary keys) واجهة تعيين.

تُستخدم الواجهة `toJson()` لتحويل كائن النموذج إلى كائن JSON. ويتم ترميز نوع الكتل الثنائية بصيغة base64. يُرجى تجربة ذلك بنفسك.

تمثل الأعضاء الساكنة (Static members) لفئة النموذج معلومات الجدول. على سبيل المثال، يمكن الحصول على اسم كل حقل عبر العضو الساكن `Cols`، وهو مريح للاستخدام في المحررات التي تدعم الإكمال التلقائي.

### قالب الفئة Mapper (Mapper Class Template)

يتم إجراء التخطيط (Mapping) بين كائن النموذج وجدول قاعدة البيانات عبر قالب الفئة Mapper. يغلف قالب الفئة Mapper العمليات الشائعة مثل الإضافة والحذف والتعديل، ليتسنى للمطور إجراء هذه العمليات دون الحاجة لكتابة جمل SQL.

إنشاء كائن Mapper بسيط للغاية. معامل القالب هو نوع النموذج الذي تريد الوصول إليه. ويمتلك دالة الإنشاء (Constructor) معاملاً واحداً فقط، وهو المؤشر الذكي لـ `DbClient` المذكور سابقاً. وكما ذكرنا سابقاً، تُعد الفئة `Transaction` فئة فرعية من `DbClient`، لذا يمكنك أيضاً إنشاء كائن Mapper بمؤشر ذكي لمعاملة برمجية، مما يعني أن تخطيط Mapper يدعم المعاملات البرمجية أيضاً.

ومثل `DbClient`، يوفر Mapper أيضاً واجهات لامتزامنة ومتزامنة. الواجهة المتزامنة مُعَطِّلة (Blocking) وقد تطلق استثناءً. بينما كائن future المعاد يُعَطِّل عند استدعاء `get()` وقد يطلق استثناءً. أما الواجهة اللامتزامنة العادية فلا تطلق استثناءً، بل تعيد النتيجة عبر دالتي استدعاء لاحق (دالة الاستدعاء اللاحق للنتيجة ودالة الاستدعاء اللاحق للاستثناءات). ونوع دالة الاستدعاء اللاحق للاستثناءات يطابق النوع الموجود في واجهة `DbClient`. وتنقسم دالة الاستدعاء اللاحق للنتيجة أيضاً إلى عدة فئات وفقاً لوظيفة الواجهة. وفيما يلي القائمة (حيث T هي معامل القالب، والذي يمثل نوع النموذج):

| اسم الواجهة (Method) | القيمة المعادة (Return value) | المعاملات (Parameter) | الاستدعاء اللاحق للنتيجة (Result callback) | مُعَطِّلة / لامُعَطِّلة (Blocking/Non-blocking) | الاستثناءات (Exception) | الشرح (Description) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| <div class="ltr-content left-align p-0">T findByPrimaryKey</div> | `T` | قيمة المفتاح الرئيسي | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | البحث عن كائن النموذج بناءً على قيمة المفتاح الرئيسي، ويطلق استثناءً في حال عدم وجود نتيجة. |
| <div class="ltr-content left-align p-0">void findByPrimaryKey</div> | `void` | قيمة المفتاح الرئيسي، ودالتان للاستدعاء اللاحق | `void(T)` | لامُعَطِّلة (Non-blocking) | لا يوجد | البحث عن كائن النموذج بناءً على قيمة المفتاح الرئيسي، ويدخل دالة الاستدعاء اللاحق للاستثناءات عند عدم وجود نتيجة. |
| <div class="ltr-content left-align p-0">findFutureByPrimaryKey</div> | `future<T>` | قيمة المفتاح الرئيسي | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | البحث عن كائن النموذج بناءً على قيمة المفتاح الرئيسي عبر future، ويطلق استثناءً عند عدم وجود نتيجة. |
| <div class="ltr-content left-align p-0">vector<T> findAll</div> | `vector<T>` | لا يوجد | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | إرجاع جميع الصفوف الموجودة في الجدول. |
| <div class="ltr-content left-align p-0">void findAll</div> | `void` | دالتان للاستدعاء اللاحق | `void(vector<T>)` | لامُعَطِّلة (Non-blocking) | لا يوجد | المفهوم نفسه أعلاه (إرجاع جميع الصفوف لامتزامناً). |
| <div class="ltr-content left-align p-0">findFutureAll</div> | `future<vector<T>>` | لا يوجد | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | المفهوم نفسه أعلاه (إرجاع جميع الصفوف عبر future). |
| <div class="ltr-content left-align p-0">size_t count</div> | `size_t` | كائن المعايير (Criteria object) | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | إرجاع عدد الصفوف التي تطابق المعايير والشروط. |
| <div class="ltr-content left-align p-0">void count</div> | `void` | كائن المعايير، ودالتان للاستدعاء اللاحق | `void(const size_t)` | لامُعَطِّلة (Non-blocking) | لا يوجد | المفهوم نفسه أعلاه (إرجاع عدد الصفوف لامتزامناً). |
| <div class="ltr-content left-align p-0">countFuture</div> | `future<size_t>` | كائن المعايير (Criteria object) | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | المفهوم نفسه أعلاه (إرجاع عدد الصفوف عبر future). |
| <div class="ltr-content left-align p-0">T findOne</div> | `T` | كائن المعايير (Criteria object) | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | إرجاع صف واحد مطابق للشرط، ويطلق استثناءً إذا كان عدد الصفوف المطابقة أكبر أو أقل من صف واحد. |
| <div class="ltr-content left-align p-0">void findOne</div> | `void` | كائن المعايير، ودالتان للاستدعاء اللاحق | `void(T)` | لامُعَطِّلة (Non-blocking) | لا يوجد | إرجاع صف واحد مطابق للشرط، وفي حال العثور على أكثر أو أقل من صف يُنفذ دالة الاستدعاء اللاحق للاستثناءات. |
| <div class="ltr-content left-align p-0">findFutureOne</div> | `future` | كائن المعايير (Criteria object) | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | إرجاع صف واحد مطابق للشرط عبر future، ويطلق استثناءً إذا كان عدد الصفوف المطابقة أكبر أو أقل من صف واحد. |
| <div class="ltr-content left-align p-0">vector<T> findBy</div> | `vector<T>` | كائن المعايير (Criteria object) | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | إرجاع 0 أو أكثر من الصفوف المطابقة للمعايير والشروط. |
| <div class="ltr-content left-align p-0">void findBy</div> | `void` | كائن المعايير، ودالتان للاستدعاء اللاحق | `void(vector<T>)` | لامُعَطِّلة (Non-blocking) | لا يوجد | المفهوم نفسه أعلاه (إرجاع الصفوف المطابقة لامتزامناً). |
| <div class="ltr-content left-align p-0">findFutureBy</div> | `future<vector<T>>` | كائن المعايير (Criteria object) | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | المفهوم نفسه أعلاه (إرجاع الصفوف المطابقة عبر future). |
| <div class="ltr-content left-align p-0">insert</div> | `void` | `T&` | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | إدراج صف بيانات، وتحديث الحقول التلقائية في المعامل. |
| <div class="ltr-content left-align p-0">insert</div> | `void` | `T&`، ودالتان للاستدعاء اللاحق | `void(T)` | لامُعَطِّلة (Non-blocking) | لا يوجد | إدراج صف بيانات، وتحديث الحقول التلقائية في معاملات دالة الاستدعاء اللاحق. |
| <div class="ltr-content left-align p-0">insertFuture</div> | `future` | `T&` | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | إدراج صف بيانات، وتحديث الحقول التلقائية داخل كائن `future.get()`. |
| <div class="ltr-content left-align p-0">size_t update</div> | `size_t` | `const T&` | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | تحديث صف بيانات، وإرجاع عدد الصفوف المحدثة (1 أو 0)، ويجب أن يحتوي الجدول على مفتاح الرئيسي. |
| <div class="ltr-content left-align p-0">void update</div> | `void` | `const T&`، ودالتان للاستدعاء اللاحق | `void(const size_t)` | لامُعَطِّلة (Non-blocking) | لا يوجد | المفهوم نفسه أعلاه (تحديث صف بيانات لامتزامناً). |
| <div class="ltr-content left-align p-0">updateFuture</div> | `future<size_t>` | `const T&` | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | المفهوم نفسه أعلاه (تحديث صف بيانات عبر future). |
| <div class="ltr-content left-align p-0">size_t deleteOne</div> | `size_t` | `const T&` | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | حذف صف بيانات، وإرجاع عدد الصفوف المحذوفة (1 أو 0)، ويجب أن يحتوي الجدول على مفتاح الرئيسي. |
| <div class="ltr-content left-align p-0">void deleteOne</div> | `void` | `const T&`، ودالتان للاستدعاء اللاحق | `void(const size_t)` | لامُعَطِّلة (Non-blocking) | لا يوجد | المفهوم نفسه أعلاه (حذف صف بيانات لامتزامناً). |
| <div class="ltr-content left-align p-0">deleteFuture</div> | `future<size_t>` | `const T&` | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | المفهوم نفسه أعلاه (حذف صف بيانات عبر future). |
| <div class="ltr-content left-align p-0">size_t deleteBy</div> | `size_t` | كائن المعايير (Criteria object) | لا يوجد | مُعَطِّلة (Blocking) | يطلق استثناءً (Throw) | حذف الصفوف المطابقة للشروط وإرجاع عدد الصفوف المحذوفة. |
| <div class="ltr-content left-align p-0">void deleteBy</div> | `void` | كائن المعايير، ودالتان للاستدعاء اللاحق | `void(const size_t)` | لامُعَطِّلة (Non-blocking) | لا يوجد | المفهوم نفسه أعلاه (حذف الصفوف المطابقة لامتزامناً). |
| <div class="ltr-content left-align p-0">deleteFutureBy</div> | `future<size_t>` | كائن المعايير (Criteria object) | لا يوجد | تُعَطِّل عند استدعاء الدالة `get()` | يطلق استثناءً عند استدعاء الدالة `get()` | المفهوم نفسه أعلاه (حذف الصفوف المطابقة عبر future). |

> **ملاحظة: عند استخدام المعاملات (Transactions)، لا يتسبب الاستثناء بالضرورة في التراجع (Rollback). ولن يتم التراجع عن المعاملات في الحالات التالية: عندما لا تعثر الواجهة `findByPrimaryKey` على صف مطابق، أو عندما تعثر الواجهة `findOne` على سجلات أقل أو أكثر من سجل واحد، حيث سيطلق Mapper استثناءً أو يدخل دالة الاستدعاء اللاحق للاستثناءات بنوع استثناء `UnexpectedRows`. وإذا كان منطق العمل يتطلب التراجع في هذه الحالة، يُرجى استدعاء الواجهة `rollback()` صراحة.**

### معايير الاستعلام (Criteria)

في القسم السابق، تطلبت العديد من الواجهات إدخال كائن معايير الاستعلام كمعامل. كائن معايير الاستعلام هو كائن من الفئة Criteria، ويعبر عن شرط معين مثل أن يكون الحقل أكبر من أو يساوي أو أصغر من قيمة معينة، أو شرطاً مثل `is Null`.

```c++
template <typename T>
Criteria(const std::string &colName, const CompareOperator &opera, T &&arg)
```

دالة الإنشاء لكائن Criteria بسيطة جداً. عادةً ما يكون المعامل الأول هو اسم الحقل، والمعامل الثاني هو قيمة تعدادية (Enumeration value) والتي تمثل نوع المقارنة، والمعامل الثالث هو القيمة المراد مقارنتها. وإذا كان نوع المقارنة IsNull أو IsNotNull، فلا داعي للمعامل الثالث.

مثال:

```c++
Criteria("user_id",CompareOperator::EQ,1);
```

يوضح المثال أعلاه أن شرط الاستعلام هو أن يكون الحقل `user_id` مساوياً للرقم 1. وفي العمل الميداني، نفضل كتابتها كالتالي:

```c++
Criteria(Users::Cols::_user_id,CompareOperator::EQ,1);
```

هذا يكافئ المثال السابق تماماً، لكن هذا الأسلوب يسمح باستخدام الإكمال التلقائي في المحرر، وهو أكثر كفاءة وأقل عرضة للأخطاء.

تدعم الفئة Criteria أيضاً شروط WHERE المخصصة جنباً إلى جنب مع دالة إنشاء مخصصة.

```c++
template <typename... Arguments>
explicit Criteria(const CustomSql &sql, Arguments &&...args)
```

المعامل الأول هو كائن `CustomSql` لجمل SQL تحتوي على عناصر نائبة `$?`، بينما الفئة `CustomSql` عبارة عن غلاف لكائن `std::string`. المعامل الثاني غير المحدد هو حزمة معاملات تمثل المعاملات المربوطة، وتسلك تماماً سلوك المعاملات المربوطة في [execSqlAsync](/ARA/ARA-08-1-Database-DbClient?id=execsqlasync).

مثال:

```c++
Criteria(CustomSql("tags @> $?"), "cloud");
```

تمتلك الفئة `CustomSql` أيضاً صيغة ثابث نصي معرف من قبل المستخدم (User-defined string literal)، لذا نوصي بكتابتها بالشكل التالي بدلاً من ذلك:

```c++
Criteria("tags @> $?"_sql, "cloud");
```

هذا يكافئ الأسلوب السابق تماماً.

تدعم كائنات Criteria العمليات المنطقية AND و OR. حيث يُنشئ مجموع كائنين من Criteria كائناً جديداً، مما يسهل إنشاء شروط متداخلة (Nested conditions). على سبيل المثال:

```c++
Mapper<Users> mp(dbClientPtr);
auto users = mp.findBy(
(Criteria(Users::Cols::_user_name,CompareOperator::Like,"%Smith")&&Criteria(Users::Cols::_gender,CompareOperator::EQ,0))
||(Criteria(Users::Cols::_user_name,CompareOperator::Like,"%Johnson")&&Criteria(Users::Cols::_gender,CompareOperator::EQ,1))
));
```

يقوم البرنامج أعلاه بالاستعلام عن جميع الرجال الذين يحملون الاسم Smith أو النساء اللاتي يحملن الاسم Johnson من جدول `users`.

### الواجهة المتسلسلة لـ Mapper (Mapper's Chain Interface)

يقدم قالب الفئة Mapper أيضاً دعماً لبعض قيود SQL الشائعة مثل limit و offset وغيرها، وتوفرها على هيئة واجهة متسلسلة (Chained interface)، مما يعني أن المطور يمكنه ربط عدة قيود متتالية أثناء الكتابة. وبعد تنفيذ أي من الواجهات المذكورة في القسم السابق، تُسحَب وتُمسَح هذه القيود، أي أنها تكون صالحة لعملية واحدة فقط:

```c++
Mapper<Users> mp(dbClientPtr);
auto users = mp.orderBy(Users::Cols::_join_time).limit(25).offset(0).findAll();
```

يقوم هذا البرنامج باختيار قائمة المستخدمين من جدول `users` لإرجاع الصفحة الأولى بمعدل 25 صفاً في كل صفحة.

وبشكل أساسي، يعبر اسم الواجهة المتسلسلة عن وظيفتها، لذا لن أطيل في التفاصيل هنا. يُرجى الرجوع إلى ملف الترويسة `Mapper.h`.

### التحويل (Convert)

خيار الإعداد `convert` فريد ومخصص لإعدادات النماذج. فهو يضيف طبقة تحويل قبل أو بعد قراءة قيمة من قاعدة البيانات أو كتابتها إليها. يتكون الكائن من مفتاح منطقي (Boolean key) `enabled` لتفعيل هذه الوظيفة أو تعطيلها. وتتكون مصفوفة كائنات `items` من المفاتيح التالية:

- `table`: اسم الجدول الذي يحتوي على العمود.
- `column`: اسم العمود.
- `method`: كائن يحتوي على:
  - `after_db_read`: نص يمثل اسم الدالة التي تُستدعى بعد القراءة من قاعدة البيانات، والتوقيع البرمجي (Signature) هو: `void([const] std::shared_ptr<type> [&])`.
  - `before_db_write`: نص يمثل اسم الدالة التي تُستدعى قبل الكتابة إلى قاعدة البيانات، والتوقيع البرمجي هو: `void([const] std::shared_ptr<type> [&])`.
- `includes`: مصفوفة نصوص تمثل أسماء ملفات التضمين المحاطة بعلامات `" "` أو `< >`.

### العلاقات (Relationships)

يمكن تكوين العلاقات بين جداول قاعدة البيانات عبر الخيار `relationships` في ملف الإعدادات `model.json`. ونحن نستخدم التكوين اليدوي بدلاً من الاكتشاف التلقائي للمفاتيح الخارجية (Foreign keys) في الجدول؛ لأن عدم استخدام المفاتيح الخارجية في المشاريع الفعلية أمر شائع جداً.

إذا كان الخيار `enable` بـ `true`، فستضيف فئات النماذج الموُلدة الواجهات المناظرة وفقاً لإعدادات `relationships`.

هناك ثلاثة أنواع من العلاقات: 'يمتلك واحداً (has one)'، و'يمتلك العديد (has many)'، و'متعدد إلى متعدد (many to many)'.

- #### يمتلك واحداً (has one)

  تُمثل `has one` علاقة واحد إلى واحد (One-to-one relationship). حيث يمكن ربط سجل في الجدول الأصلي بسجل في الجدول الهدف، والعكس صحيح. على سبيل المثال، يمتلك جدول `products` وجدول `skus` علاقة واحد إلى واحد، ويمكننا تعريفها كالتالي:

  ```json
  {
    "type": "has one",
    "original_table_name": "products",
    "original_table_alias": "product",
    "original_key": "id",
    "target_table_name": "skus",
    "target_table_alias": "SKU",
    "target_key": "product_id",
    "enable_reverse": true
  }
  ```

  ومنها:

  - "type": يوضح أن هذه العلاقة هي واحد إلى واحد.
  - "original_table_name": اسم الجدول الأصلي (ستُضاف الدالة المناظرة إلى النموذج المناظر لهذا الجدول).
  - "original_table_alias": الاسم المستعار (الاسم في الدالة، ونظراً لأن علاقة واحد إلى واحد تكون بالمفرد، نضبطها إلى `product`)، وإذا كان هذا الخيار فارغاً يُستخدم اسم الجدول لتوليد اسم الدالة.
  - "original_key": المفتاح المربوط في الجدول الأصلي.
  - "target_table_name": اسم الجدول الهدف.
  - "target_table_alias": الاسم المستعار للجدول الهدف، وإذا كان هذا الخيار فارغاً يُستخدم اسم الجدول لتوليد اسم الدالة.
  - "target_key": المفتاح المربوط في الجدول الهدف.
  - "enable_reverse": يوضح ما إذا كان سيتم توليد علاقة عكسية تلقائياً، أي إضافة دالة للحصول على سجلات الجدول الأصلي داخل فئة النموذج المناظرة للجدول الهدف.

  وفقاً لهذه الإعدادات، ستُضاف الدالة التالية إلى فئة النموذج المناظرة لجدول `products`:

  ```c++
      /// Relationship interfaces
      void getSKU(const DbClientPtr &clientPtr,
                  const std::function<void(Skus)> &rcb,
                  const ExceptionCallback &ecb) const;
  ```

  هذه واجهة لامتزامنة تعيد كائن SKU المربوط بالمنتج الحالي داخل دالة الاستدعاء اللاحق.

  في الوقت نفسه، ونظراً لضبط الخيار `enable_reverse` على `true`، ستُضاف الدالة التالية إلى فئة النموذج المناظرة لجدول `skus`:

  ```c++
      /// Relationship interfaces
      void getProduct(const DbClientPtr &clientPtr,
                      const std::function<void(Products)> &rcb,
                      const ExceptionCallback &ecb) const;
  ```

- #### يمتلك العديد (has many)

  تُمثل `has many` علاقة واحد إلى متعدد (One-to-many relationship). وفي مثل هذه العلاقة، يحتوي الجدول الذي يمثل 'المتعدد' عادةً على حقل مرتبط بالمفتاح الرئيسي لجدول آخر. على سبيل المثال، يمتلك جدول المنتجات والتقييمات عادةً علاقة واحد إلى متعدد، ويمكننا تعريفها كالتالي:

  ```json
  {
    "type": "has many",
    "original_table_name": "products",
    "original_table_alias": "product",
    "original_key": "id",
    "target_table_name": "reviews",
    "target_table_alias": "",
    "target_key": "product_id",
    "enable_reverse": true
  }
  ```

  معنى كل خيار أعلاه يطابق المثال السابق، لذا لن أكرره هنا، ونظراً لوجود تقييمات متعددة للمنتج الواحد، فلا حاجة لإنشاء اسم مستعار لـ `reviews`. ووفقاً لهذا الإعداد، وبعد تشغيل `drogon_ctl create model` ستُضاف الواجهة التالية إلى النموذج المناظر لجدول `products`:

  ```c++
      void getReviews(const DbClientPtr &clientPtr,
                      const std::function<void(std::vector<Reviews>)> &rcb,
                      const ExceptionCallback &ecb) const;
  ```

  وفي النموذج المناظر لجدول `reviews` ستُضاف الواجهة التالية:

  ```c++
      void getProduct(const DbClientPtr &clientPtr,
                      const std::function<void(Products)> &rcb,
                      const ExceptionCallback &ecb) const;
  ```

- #### متعدد إلى متعدد (many to many)

  كما يتضح من الاسم، تُمثل `many to many` علاقة متعدد إلى متعدد (Many-to-many relationship). وعادةً ما تتطلب علاقة متعدد إلى متعدد جدولاً وسيطاً (Pivot table). يطابق كل سجل في الجدول الوسيط سجلاً في الجدول الأصلي وسجلاً آخر في الجدول الهدف. على سبيل المثال، يمتلك جدول `products` وجدول `carts` علاقة متعدد إلى متعدد، والتي يمكن تعريفها كالتالي:

  ```json
  {
    "type": "many to many",
    "original_table_name": "products",
    "original_table_alias": "",
    "original_key": "id",
    "pivot_table": {
      "table_name": "carts_products",
      "original_key": "product_id",
      "target_key": "cart_id"
    },
    "target_table_name": "carts",
    "target_table_alias": "",
    "target_key": "id",
    "enable_reverse": true
  }
  ```

  بالنسبة للجدول الوسيط، هناك خيار إعداد إضافي هو `pivot_table`. والخيارات بداخله سهلة الفهم لذا سنتجاوز شرحها هنا.

  ستضيف فئة النموذج لـ `products` المُولدة وفقاً لهذا التكوين الدالة التالية:

  ```c++
      void getCarts(const DbClientPtr &clientPtr,
                    const std::function<void(std::vector<std::pair<Carts,CartsProducts>>)> &rcb,
                    const ExceptionCallback &ecb) const;
  ```

  بينما ستضيف فئة النموذج لجدول `carts` الدالة التالية:

  ```c++
      void getProducts(const DbClientPtr &clientPtr,
                      const std::function<void(std::vector<std::pair<Products,CartsProducts>>)> &rcb,
                      const ExceptionCallback &ecb) const;
  ```

### متحكمات واجهات RESTful البرمجية (Restful API controllers)

يمكن لـ `drogon_ctl` أيضاً توليد متحكمات بنمط RESTful لكل نموذج (أو جدول) أثناء إنشاء النماذج، بحيث يمكن للمطورين توليد واجهات برمجة تطبيقات (APIs) يمكنها الإضافة والحذف والتعديل والبحث في الجداول دون الحاجة لكتابة أي كود برمجي (Zero coding). وتدعم هذه الواجهات العديد من الوظائف مثل الاستعلام حسب المفتاح الرئيسي، والاستعلام حسب الشروط، والترتيب حسب حقول معينة، وإرجاع حقول محددة، وتعيين اسم مستعار لكل حقل لإخفاء هيكل الجدول. ويتم التحكم في ذلك عبر الخيار `restful_api_controllers` في ملف `model.json`. وتحتوي هذه الخيارات على التعليقات المناظرة لها داخل ملف JSON.

وتجدر الإشارة إلى أن متحكم كل جدول مصمم ليتكون من فئة أساسية (Base class) وفئة فرعية (Subclass). ومن بينها، ترتبط الفئة الأساسية والجدول ارتباطاً وثيقاً، بينما تُستخدم الفئة الفرعية لتنفيذ منطق العمل الخاص أو تعديل تنسيق الواجهة. وتكمن فائدة هذا التصميم في أنه عندما يتغير هيكل الجدول، يمكن للمطورين تحديث الفئة الأساسية فقط دون تجاوز الفئة الفرعية أو الكتابة فوقها (عبر ضبط الخيار `generate_base_only` على `true`).

# التالي: [عميل قاعدة البيانات السريع (FastDbClient)](/ARA/ARA-08-4-Database-FastDbClient)

</div>
