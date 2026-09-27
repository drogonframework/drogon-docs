<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-12-drogon_ctl命令) - [English](/ENG/ENG-12-drogon_ctl-Command)

# أوامر drogon_ctl البرمجية (drogon_ctl - Command)

بعد تجميع إطار العمل **Drogon** وتثبيته، يُوصى بإنشاء مشروعك الأول باستخدام برنامج سطر الأوامر `drogon_ctl` الذي يتم تثبيته جنباً إلى جنب مع إطار العمل، وللتسهيل يتوفر خيار الأمر المختصر `dg_ctl`، حيث يمكن للمطور الاختيار بينهما وفقاً لتفضيلاته.

الوظيفة الرئيسية للبرنامج هي تسهيل إنشاء ملفات مشاريع Drogon المختلفة للمطورين. يمكنك استخدام الأمر `dg_ctl help` لعرض الوظائف التي يدعمها البرنامج كالتالي:

```console
$ dg_ctl help
usage: drogon_ctl [-v | --version] [-h | --help] <command> [<args>]
commands list:
create                  create some source files(Use 'drogon_ctl help create' for more information)
press                   Do stress testing(Use 'drogon_ctl help press' for more information)
help                    display this message
version                 display version of this tool
```

### الأمر الفرعي للمصادقة والإصدار (Version subcommand)

يُستخدم الأمر الفرعي `version` لطباعة إصدار Drogon المثبت حالياً على النظام كالتالي:

```console
$ dg_ctl version
     _
  __| |_ __ ___   __ _  ___  _ __
 / _` | '__/ _ \ / _` |/ _ \| '_ \
| (_| | | | (_) | (_| | (_) | | | |
 \__,_|_|  \___/ \__, |\___/|_| |_|
                 |___/

A utility for drogon
Version: 1.9.13
Git commit: 
Compilation: 
  Compiler: c++
  Compiler ID: GNU
  Compilation flags: -O3 -DNDEBUG -std=c++17 -I/usr/include/jsoncpp -I/opt/drogon/1.9.13/include
Libraries: 
  postgresql: yes  (pipeline mode: yes)
  mariadb: yes
  sqlite3: yes
  ssl/tls backend: OpenSSL
  brotli: yes
  hiredis: yes
  c-ares: yes
  yaml-cpp: yes
```

### الأمر الفرعي للإنشاء (Create subcommand)

يُستخدم الأمر الفرعي `create` لإنشاء كائنات وملفات مختلفة، وهو الوظيفة الرئيسية لـ `drogon_ctl` حالياً. استخدم الأمر `dg_ctl help create` لطباعة تعليمات المساعدة التفصيلية لهذا الأمر كالتالي:

```console
$ dg_ctl help create
Use create command to create some source files of drogon webapp

Usage:drogon_ctl create <view|controller|filter|project|model> [-options] <object name>

drogon_ctl create view <csp file name> [-o <output path>] [-n <namespace>] [--path-to-namespace] //create HttpView source files from csp files, namespace is prefixed of path-to-namespace

drogon_ctl create controller [-s] <[namespace::]class_name> //create HttpSimpleController source files

drogon_ctl create controller -h <[namespace::]class_name> //create HttpController source files

drogon_ctl create controller -w <[namespace::]class_name> //create WebSocketController source files

drogon_ctl create controller -r <[namespace::]class_name> [--resource=...]//create restful controller source files

drogon_ctl create filter <[namespace::]class_name> //create a filter named class_name

drogon_ctl create plugin <[namespace::]class_name> //create a plugin named class_name

drogon_ctl create project <project_name> //create a project named project_name

drogon_ctl create model <model_path> [-o <output path>] [ --clear-output][--table=<table_name>] [-f]//create model classes in model_path
```

- #### إنشاء العرض (View creation)

  يُستخدم الأمر `dg_ctl create view` لتوليد ملفات المصدر من ملفات csp، راجع قسم [العرض (View)](/ARA/ARA-06-View). وبشكل عام، لا داعي لاستخدام هذا الأمر مباشرة، حيث يُفضل إعداد ملف CMake لتنفيذ هذا الأمر تلقائياً. مثال الأمر كالتالي بفرض أن اسم ملف csp هو `UsersList.csp`:

  ```shell
  dg_ctl create view UsersList.csp
  ```

- #### إنشاء المتحكم (Controller creation)

  يُستخدم الأمر `dg_ctl create controller` لمساعدة المطور في إنشاء الملفات المصدرية للمتحكمات. ويمكن إنشاء المتحكمات الثلاثة المدعومة حالياً في Drogon عبر هذا الأمر.

  - الأمر الخاص بإنشاء متحكم بسيط `HttpSimpleController` كالتالي:

  ```shell
  dg_ctl create controller SimpleControllerTest
  dg_ctl create controller webapp::v1::SimpleControllerTest
  ```

  المعامل الأخير هو اسم فئة المتحكم، ويمكن إلحاق مساحة أسماء (Namespace) كسابقة له.

  - الأمر الخاص بإنشاء متحكم `HttpController` كالتالي:

  ```shell
  dg_ctl create controller -h ControllerTest
  dg_ctl create controller -h api::v1::ControllerTest
  ```

  - الأمر الخاص بإنشاء متحكم مقابس الويب `WebSocketController` كالتالي:

  ```shell
  dg_ctl create controller -w WsControllerTest
  dg_ctl create controller -w api::v1::WsControllerTest
  ```

- #### إنشاء الفلتر (Filter creation)

  يُستخدم الأمر `dg_ctl create filter` لمساعدة المطور في إنشاء الملفات المصدرية للفلاتر، راجع قسم [البرمجيات الوسيطة والفلاتر (Middleware and Filter)](/ARA/ARA-05-Middleware-and-Filter).

  ```shell
  dg_ctl create filter LoginFilter
  dg_ctl create filter webapp::v1::LoginFilter
  ```

- #### إنشاء المشروع (Create project)

  أفضل طريقة لإنشاء مشروع تطبيق Drogon جديد هي عبر الأمر `drogon_ctl` كالتالي:

  ```shell
  dg_ctl create project ProjectName
  ```

  بعد تنفيذ الأمر، سيتم إنشاء مجلد مشروع كامل داخل المجلد الحالي باسم `ProjectName`، ويمكن للمطور تجميع المشروع مباشرة داخل مجلد البناء (`cmake .. && make`). وبالطبع لا يحتوي المشروع في هذه المرحلة على أي منطق أعمال (Business logic).

  هيكل مجلدات المشروع يكون كالتالي:

  ```console
  ├── build                         مجلد التجميع والبناء
  ├── CMakeLists.txt                ملف إعدادات CMake الخاص بالمشروع
  ├── cmake_modules                 سكربتات CMake للبحث عن المكتبات الخارجية
  │   ├── FindJsoncpp.cmake
  │   ├── FindMySQL.cmake
  │   ├── FindSQLite3.cmake
  │   └── FindUUID.cmake
  ├── config.json                   ملف إعدادات تطبيق Drogon، يُرجى الرجوع إلى دليل ملف الإعدادات
  ├── controllers                   المجلد المخصص لتخزين الملفات المصدرية للمتحكمات
  ├── filters                       المجلد المخصص لتخزين ملفات الفلاتر
  ├── main.cc                       البرنامج الرئيسي
  ├── models                        المجلد المخصص لملفات نماذج قاعدة البيانات
  │   └── model.json
  ├── tests                         المجلد المخصص لاختبارات الوحدة والتكامل
  │   └── test_main.cc              نقطة الدخول الرئيسية للاختبارات
  └── views                         المجلد المخصص لتخزين ملفات العرض csp (لا تطلب إنشاء ملفات مصدرية يدوياً، حيث يتم معالجة ملفات csp تلقائياً لتوليد ملفات المصدر عند تجميع المشروع)
  ```

- #### إنشاء النماذج (Create models)

  استخدم الأمر `dg_ctl create model` لإنشاء الملفات المصدرية لنماذج قواعد البيانات. المعامل الأخير هو المجلد الذي تُخزن فيه النماذج. ويجب أن يحتوي هذا المجلد على ملف إعدادات النموذج باسم `model.json` لإخبار `dg_ctl` بآلية الاتصال بقاعدة البيانات والجداول المراد تخطيطها.

  على سبيل المثال، إذا أردت إنشاء النماذج داخل مجلد المشروع المذكور أعلاه، نفذ الأمر التالي داخل مجلد المشروع:

  ```shell
  dg_ctl create model models
  ```

  سيقوم هذا الأمر بتنبيه المطور بأن الملفات سيتم استبدالها صراحة، وبعد أن يدخل المطور الحرف `y` سيتم توليد كافة ملفات النماذج.

  الملفات المصدرية الأخرى التي تحتاج إلى الإشارة إلى فئات النماذج يجب أن تتضمن ملفات الترويسة الخاصة بالنموذج، مثل:

  ```c++
  #include "models/User.h"
  ```

  لاحظ أنه يتم تضمين اسم مجلد النماذج للتمييز بين مصادر البيانات المتعددة داخل المشروع نفسه. راجع دليل [تخطيط الكائنات العلائقية (ORM)](/ARA/ARA-08-3-Database-ORM).

### اختبار الإجهاد والتحمل (Stress Testing)

يمكن استخدام الأمر `dg_ctl press` لإجراء اختبار الإجهاد والضغط، وهناك عدة خيارات متوفرة لهذا الأمر:

- `-n {num}` تحديد عدد الطلبات (الافتراضي: 1).
- `-t {num}` تحديد عدد خيوط التنفيذ (الافتراضي: 1)، ويُفضل ضبط الرقم على عدد نوى المعالج للوصول إلى الأداء الأقصى.
- `-c {num}` تحديد عدد الاتصالات المتزامنة (الافتراضي: 1).
- `-q` إخفاء شريط مؤشر التقدم (الافتراضي: عدم الإخفاء).

على سبيل المثال، يمكن للمطورين اختبار خادم HTTP بالشكل التالي:

```shell
dg_ctl press -n 1000000 -t 4 -c 1000 -q http://localhost:8080/
dg_ctl press -n 1000000 -t 4 -c 1000 https://www.domain.com/path/to/be/tested
```

# التالي: [البرمجة موجهة الجوانب (AOP Aspect-Oriented Programming)](/ARA/ARA-13-AOP-Aspect-Oriented-Programming)

</div>
