<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-09-插件) - [English](/ENG/ENG-09-1-File-Handler)

# معالج الملفات (File Handler)

تحليل الملفات هي عملية استخراج الملف (أو الملفات) من طلبات POST ذات البيانات متعددة الأجزاء (Multipart-data) وتحويلها إلى كائن `HttpFile` عبر استخدام `MultiPartParser`. وفيما يلي بعض التفاصيل المتعلقة بذلك:

## كائن `MultiPartParser`

  #### الملخص:
  هو الكائن الذي ستستخدمه لاستخراج ملفات الطلب وتخزينها مؤقتاً.

- ### `parse(const std::shared_ptr<HttpRequest> &req)`

  #### الملخص:
  يستقبل كائن الطلب كمعامل (Parameter)، ويقرأ ويتعرف على الملفات المرفقة (إن وجُدت)، ثم ينقلها إلى متغير `MultiPartParser`.

  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);
  }
  ```

- ### `getFiles()`

  #### الملخص:
  يجب استدعاؤها بعد الدالة `parse()`، وتُعيد ملفات الطلب بصيغة `std::vector<HttpFile>`.

  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);

      // التحقق مما إذا كانت هناك ملفات
      if (fileParser.getFiles().empty()) {
          // لم يتم العثور على ملفات
      }

      size_t num_of_files = fileParser.getFiles().size();
  }
  ```

- ### `getParameters()`

  #### الملخص:
  يجب استدعاؤها بعد الدالة `parse()`، وتُعيد قائمة الأجزاء الأخرى المضمنة في نموذج MultiPartData.

  #### المخرجات:
  `std::unordered_map<std::string, std::string>` (مفتاح، قيمة).

  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);

      if (!fileParser.getFiles().empty()) {
          for (const auto &header : fileParser.getParameters()){
              header.first;  // مفتاح نموذج الإدخال
              header.second; // قيمة مفتاح نموذج الإدخال
          }
      }
  }
  ```

- ### `getParameter<Typename T>(const std::string &key)`

  #### الملخص:
  يجب استدعاؤها بعد الدالة `parse()`، وهي النسخة الفردية المخصصة من الدالة `getParameters()`.

  #### المدخلات:
  نوع الكائن المتوقع (سيتم تحويله تلقائياً)، وقيمة المفتاح الخاص بالمعامل.

  #### المخرجات:
  محتوى المعامل المناظر للمفتاح بالنوع المحدد، وإذا لم يكن موجوداً، ستُعيد القيمة الافتراضية للكائن T.

  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);

      std::string email = fileParser.getParameter<std::string>("email_form");

      // القيمة الافتراضية للنوع string هي ""
      if (email.empty()) {
          // لم يتم العثور على email_form
      }
  }
  ```

## كائن `HttpFile`

  #### الملخص:
  هو الكائن الذي يمثل الملف في الذاكرة والمُستخدم بواسطة `MultiPartParser`.

- ### `getFileName()`

  #### الملخص:
  اسمها يعبر عن وظيفتها مباشرة؛ حيث تجلب الاسم الأصلي للملف الذي تم استقباله.

  #### المخرجات:
  `std::string`.

  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);

      std::string filename = fileParser.getFiles()[0].getFileName();
  }
  ```

- ### `fileLength()`

  #### الملخص:
  تجلب حجم الملف.

  #### المخرجات:
  `size_t`.

  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);

      size_t filesize = fileParser.getFiles()[0].fileLength();
  }
  ```

- ### `getFileExtension()`

  #### الملخص:
  تجلب امتداد الملف.

  #### المخرجات:
  `std::string`.

  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);

      std::string file_extension = fileParser.getFiles()[0].getFileExtension();
  }
  ```

- ### `getMd5()`

  #### الملخص:
  تجلب تجزئة MD5 (MD5 hash) للملف للتحقق من سلامته.

  #### المخرجات:
  `std::string`.

- ### `save()`

  #### الملخص:
  تحفظ الملف في نظام الملفات. المجلد المخصص لحفظ الملف هو `UploadPath` المحدد في ملف `config.json` (أو ما يكافئه). المسار الكامل يكون:
  ```c++ 
  drogon::app().getUploadPath() + "/" + this->getFileName()
  ```
  أو للتبسيط، يتم حفظه كـ: `UploadPath/filename`.

- ### `save(const std::string &path)`

  #### الملخص:
  نسخة الدالة في حال عدم حذف المعامل حيث تستخدم المعامل `&path` بدلاً من `UploadPath`.
  
  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);

      // مسار نسبي (Relative path)
      fileParser.getFiles()[0].save("./"); // يكتب الملف في المجلد نفسه على الخادم، مع الاحتفاظ بالاسم الأصلي

      // مسار مطلق (Absolute path)
      fileParser.getFiles()[0].save("/home/user/downloads/"); // يكتب الملف في المجلد المحدد، مع الاحتفاظ بالاسم الأصلي
  }
  ```

- ### `saveAs(const std::string &path)`

  #### الملخص:
  تكتب الملف في مسار المعامل باسم جديد (وتتجاهل الاسم الأصلي).
  
  #### مثال:
  ```c++
  #include "mycontroller.h"

  using namespace drogon;

  void mycontroller::postfile(const HttpRequestPtr &req, std::function<void (const HttpResponsePtr &)> &&callback) {
      // طلبات Post فقط (نموذج ملفات File Form)

      MultiPartParser fileParser;
      fileParser.parse(req);

      // مسار نسبي (Relative path)
      fileParser.getFiles()[0].saveAs("./image.png"); // المسار نفسه الخاص بالخادم
      /* هذا مجرد مثال، لا تفعل ذلك في التطبيق الفعلي لتجنب كتابة امتداد الملف دون التحقق مما إذا كان ينتمي حقاً لنوع png */

      // مسار مطلق (Absolute path)
      fileParser.getFiles()[0].saveAs("/home/user/downloads/anyname." + fileParser.getFiles()[0].getFileExtension());
  }
  ```

# التالي: [الإضافات (Plugins)](/ARA/ARA-10-Plugins)

</div>
