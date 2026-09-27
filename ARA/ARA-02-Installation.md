<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-02-安装) - [English](/ENG/ENG-02-Installation)

# التثبيت (Installation)

يقدم هذا القسم شرحاً لعملية التثبيت على نظم Ubuntu 24.04 و CentOS 7.5 و MacOS 12.2 كأمثلة. والأنظمة الأخرى تماثلها في الخطوات.

## متطلبات النظام (System Requirements)

* ألا يقل إصدار نواة Linux عن 2.6.9، بنواة 64-bit.
* ألا يقل إصدار gcc عن 5.4.0، ويُنصح باستخدام الإصدار 11 أو ما هو أحدث.
* استخدام cmake كأداة بناء، وألا يقل إصدار cmake عن 3.5.
* استخدام git كأداة لإدارة الإصدارات.

## اعتماديات المكتبات (Library Dependencies)

* **مدمجة (Built-in)**
  * trantor: مكتبة شبكات بلغة C++ تعتمد على الإدخال/الإخراج اللامُعَطِّل (Non-blocking I/O)، طوّرها أيضاً مؤلف إطار عمل Drogon نفسه، ومُضمنة كموديول فرعي (Git submodule)، ولا تتطلب التثبيت المسبق.
* **إلزامية (Mandatory)**
  * jsoncpp: مكتبة C++ لمعالجة JSON، ويجب ألا يقل إصدارها عن **1.7**.
  * libuuid: مكتبة C لتوليد المعرفات الفريدة الموحدة (UUID).
  * zlib: تُستخدم لدعم نقل البيانات المضغوطة.
* **اختيارية (Optional)**
  * boost: يجب ألا يقل إصدارها عن **1.61**، وتكون مطلوبة فقط إذا كان مترجم C++ لا يدعم معيار C++17 أو إذا كانت مكتبة STL لا تدعم مكتبة `std::filesystem` بالكامل.
  * OpenSSL: بعد تثبيتها، سيدعم Drogon بروتوكول HTTPS، وبدونها سيدعم HTTP فقط.
  * c-ares: بعد تثبيتها، سيكون Drogon أكثر كفاءة مع الاستعلام عن أسماء النطاقات (DNS).
  * libbrotli: بعد تثبيتها، سيدعم Drogon ضغط البيانات بواسطة Brotli عند إرسال استجابات HTTP.
  * مكتبات تطوير العملاء لكل من PostgreSQL و MariaDB و SQLite3: عند تثبيت واحدة أو أكثر منها، سيدعم Drogon الاتصال بقواعد البيانات المقابلة.
  * hiredis: بعد تثبيتها، سيدعم Drogon الاتصال بقواعد بيانات Redis.
  * gtest: بعد تثبيتها، يمكن تجميع واختبار وحدات الكود (Unit tests).
  * yaml-cpp: بعد تثبيتها، سيدعم Drogon ملفات الإعدادات بصيغة YAML.

## أمثلة تجهيز النظام (System Preparation Examples)

#### Ubuntu 24.04

* البيئة الأساسية

  ```shell
  sudo apt install git gcc g++ cmake
  ```

* jsoncpp

  ```shell
  sudo apt install libjsoncpp-dev
  ```

* uuid

  ```shell
  sudo apt install uuid-dev
  ```

* zlib

  ```shell
  sudo apt install zlib1g-dev
  ```

* OpenSSL (اختياري، إذا كنت تريد دعم HTTPS)

  ```shell
  sudo apt install openssl libssl-dev
  ```

#### Arch Linux

* البيئة الأساسية

  ```shell
  sudo pacman -S git gcc make cmake
  ```

* jsoncpp

  ```shell
  sudo pacman -S jsoncpp
  ```

* uuid

  ```shell
  sudo pacman -S uuid
  ```

* zlib

  ```shell
  sudo pacman -S zlib
  ```

* OpenSSL (اختياري، إذا كنت تريد دعم HTTPS)

  ```shell
  sudo pacman -S openssl libssl

#### CentOS 7.5

* البيئة الأساسية

  ```shell
  yum install git
  yum install gcc
  yum install gcc-c++
  ```

  ```shell
  # إصدار cmake التلقائي المتاح قديم جداً، استخدم التثبيت من المصدر
  git clone https://github.com/Kitware/CMake
  cd CMake/
  ./bootstrap && make && make install
  ```

  ```shell
# ترقية gcc
  yum install centos-release-scl
  yum install devtoolset-11
  scl enable devtoolset-11 bash
  ```

  > **ملاحظة: الأمر `scl enable devtoolset-11 bash` يفعل إصدار gcc الجديد بشكل مؤقت حتى نهاية الجلسة الحالية فقط. إذا أردت استخدامه دائماً، يمكنك تشغيل الأمر `echo "scl enable devtoolset-11 bash" >> ~/.bash_profile` وسيقوم النظام بتفعيل إصدار gcc الجديد تلقائياً عند كل إعادة تشغيل.**

* jsoncpp

  ```shell
  git clone https://github.com/open-source-parsers/jsoncpp
  cd jsoncpp/
  mkdir build
  cd build
  cmake ..
  make && make install
  ```

* uuid

  ```shell
  yum install libuuid-devel
  ```

* zlib

  ```shell
  yum install zlib-devel
  ```

* OpenSSL (اختياري، إذا كنت تريد دعم HTTPS)

  ```shell
  yum install openssl-devel
  ```

#### MacOS 12.2

* البيئة الأساسية

  جميع الأدوات الأساسية مدمجة في نظام MacOS، كل ما تحتاجه هو تحديثها.

  ```shell
# ترقية gcc
  brew upgrade
  ```

* jsoncpp

  ```shell
  brew install jsoncpp
  ```

* uuid

  ```shell
  brew install ossp-uuid
  ```

* zlib

  ```shell
  yum install zlib-devel
  ```

* OpenSSL (اختياري، إذا كنت تريد دعم HTTPS)

  ```shell
  brew install openssl
  ```

#### Windows

* البيئة الأساسية (Visual Studio 2019)
  قم بتثبيت Visual Studio 2019 Professional، على أن تتضمن على الأقل الخيارات التالية:
  * أدوات بناء MSVC C++ (MSVC C++ building tools)
  * حزمة تطوير البرمجيات Windows 10 SDK
  * أدوات C++ CMake الخاصة بـ Windows
  * محول الاختبار لـ Google Test (Test adaptor for Google Test)

يمكن لمدير الحزم `conan` توفير كافة الاعتماديات التي يحتاجها مشروع Drogon. إذا كان Python مثبتاً على نظامك، يمكنك تثبيت `conan` عبر أمر pip:

```shell
pip install conan
```
> بالطبع يمكن تحميل ملف التثبيت المباشر لـ `conan` من [الموقع الرسمي](https://conan.io/) وتثبيته.

أنشئ ملفاً باسم `conanfile.txt` وأضف المحتوى التالي إليه:

* jsoncpp

  ```txt
  [requires]
  jsoncpp/1.9.4
  ```

* uuid

  لا يتطلب تثبيتاً، حيث تتضمن حزمة Windows 10 SDK مكتبة uuid بالفعل.

* zlib

  ```txt
  [requires]
  zlib/1.2.11
  ```

* OpenSSL (اختياري، إذا كنت تريد دعم HTTPS)

  ```txt
  [requires]
  openssl/1.1.1t
  ```

## بيئة قواعد البيانات (اختياري)

> **ملاحظة: المكتبات التالية ليست إلزامية. يمكنك اختيار تثبيت قاعدة بيانات واحدة أو أكثر وفقاً لاحتياجاتك الفعلية.**

> **ملاحظة: إذا أردت تطوير تطبيق الويب الخاص بك مع قاعدة بيانات، يرجى تثبيت بيئة تطوير قاعدة البيانات أولاً قبل تثبيت Drogon، وإلا فستواجه مشكلة `NO DATABASE FOUND`.**

* #### PostgreSQL

  يلزم تثبيت مكتبة C الأصلية لـ PostgreSQL المسماة libpq. خطوات التثبيت كالتالي:

  * `ubuntu 16`: `sudo apt-get install postgresql-server-dev-all`
  * `ubuntu 18`: `sudo apt-get install postgresql-all`
  * `ubuntu 24`: `sudo apt-get install postgresql-all`
  * `arch`: `sudo pacman -S postgresql`
  * `centOS 7`: `yum install postgresql-devel`
  * `MacOS`: `brew install postgresql`
  * `Windows conanfile`: `libpq/13.4`

* #### MySQL

  لا تدعم المكتبة الأصلية لـ MySQL القراءة والكتابة اللامتزامنة (Asynchronous). ومن لحسن الحظ أن لـ MySQL إصداراً آخر هو MariaDB يتكفل بصيانته مجتمع المطورين الأصليين، وهذا الإصدار متوافق مع MySQL وتدعم مكتبة التطوير الخاصة به القراءة والكتابة اللامتزامنة. لذلك يستعين Drogon بمكتبة تطوير MariaDB لتوفير الدعم الصحيح لـ MySQL. كأفضل ممارسة، ينبغي ألا يحتوي نظام التشغيل لديك على كل من MySQL و MariaDB مثبتين معاً في نفس الوقت.

  خطوات تثبيت MariaDB كالتالي:

  * `ubuntu 18.04`: `sudo apt install libmariadbclient-dev`
  * `ubuntu 24.04`: `sudo apt install libmariadb-dev-compat libmariadb-dev`
  * `arch`: `sudo pacman -S mariadb`
  * `centOS 7`: `yum install mariadb-devel`
  * `MacOS`: `brew install mariadb`
  * `Windows conanfile`: `libmariadb/3.1.13`

* #### SQLite3

  * `ubuntu`: `sudo apt-get install libsqlite3-dev`
  * `arch`: `sudo pacman -S sqlite3`
  * `centOS`: `yum install sqlite-devel`
  * `MacOS`: `brew install sqlite3`
  * `Windows conanfile`: `sqlite3/3.36.0`

* #### Redis
  * `ubuntu`: `sudo apt-get install libhiredis-dev`
  * `arch`: `sudo pacman -S redis`
  * `centOS`: `yum install hiredis-devel`
  * `MacOS`: `brew install hiredis`
  * `Windows conanfile`: `hiredis/1.0.0`

> **ملاحظة: بعض الأوامر أعلاه تقوم بتثبيت مكتبات التطوير (Development libraries) فقط. إذا كنت تريد تثبيت خادم قواعد البيانات نفسه (Server)، يرجى البحث بنفسك في محرك البحث Google.**

## تثبيت Drogon (Drogon Installation)

بافتراض أن بيئة التشغيل واعتماديات المكتبات المذكورة أعلاه أصبحت جاهزة بالكامل، فإن عملية التثبيت بسيطة جداً.

* #### التثبيت من المصدر في Linux

  ```shell
  cd $WORK_PATH
  git clone https://github.com/drogonframework/drogon
  cd drogon
  git submodule update --init
  mkdir build
  cd build
  cmake ..
  make && sudo make install
  ```

  > النمط التلقائي يُجمع نسخة التنقيح (Debug version). إذا أردت تجميع النسخة النهائية (Release version)، ينبغي تمرير المعامل التالي لأمر cmake:

  ```shell
  cmake -DCMAKE_BUILD_TYPE=Release ..
  ```

  بعد اكتمال التثبيت، سيتم تثبيت الملفات التالية في النظام (يمكن تغيير مسار التثبيت عبر الخيار CMAKE_INSTALL_PREFIX):

  * ملف الترويسة لـ drogon يُثبَّت في المسار `/usr/local/include/drogon`.
  * ملف مكتبة drogon وهو `libdrogon.a` يُثبَّت في المسار `/usr/local/lib`.
  * أداة سطر الأوامر الخاصة بـ Drogon وهي `drogon_ctl` تُثبَّت في المسار `/usr/local/bin`.
  * ملف الترويسة لـ trantor يُثبَّت في المسار `/usr/local/include/trantor`.
  * ملف مكتبة trantor وهو `libtrantor.a` يُثبَّت في المسار `/usr/local/lib`.

* #### التثبيت من المصدر في Windows

  1. تحميل كود drogon المصدري:

      افتح مربع البحث في شريط المهام في نظام Windows، وابحث عن ​`x64 Native Tools`، واختر ​`x64 Native Tools Command Prompt for VS 2019​` كأداة لسطر الأوامر.
      ```dos
      cd $WORK_PATH
      git clone https://github.com/drogonframework/drogon
      cd drogon
      git submodule update --init
      ```

  2. تثبيت الاعتماديات:

     تثبيت الاعتماديات عبر `conan`:

      ```dos
      mkdir build
      cd build
      conan profile detect --force
      conan install .. -s compiler="msvc" -s compiler.version=193  -s compiler.cppstd=17 -s build_type=Debug  --output-folder . --build=missing
      ```

      > يمكنك تعديل ملف `conanfile.txt` لتغيير إصدارات الاعتماديات.

  3. التجميع والتثبيت:

      ```dos
      cmake ..  -DCMAKE_BUILD_TYPE=Debug -DCMAKE_TOOLCHAIN_FILE="conan_toolchain.cmake" -DCMAKE_POLICY_DEFAULT_CMP0091=NEW -DCMAKE_INSTALL_PREFIX="D:"
      cmake --build . --parallel --target install
      ```

  > **ملاحظة: يجب التأكد من تطابق نوع البناء (Build type) بين كل من conan و cmake.**

  بعد اكتمال التثبيت، سيتم تثبيت الملفات التالية في النظام (يمكن تغيير موقع التثبيت عبر خيار `CMAKE_INSTALL_PREFIX`):

  * ملف الترويسة لـ drogon يُثبَّت في المسار `D:/include/drogon`.
  * ملف مكتبة drogon وهو `drogon.dll` يُثبَّت في المسار `D:/bin`.
  * أداة سطر الأوامر لـ Drogon وهي `drogon_ctl.exe` تُثبَّت في المسار `D:/bin`.
  * ملف الترويسة لـ trantor يُثبَّت في المسار `D:/include/trantor`.
  * ملف مكتبة trantor وهو `trantor.dll` يُثبَّت في المسار `D:/lib`.

  أضف المجلدات `bin` و `cmake` إلى متغير البيئة `path`:
  ```
  D:\bin
  ```
  ```
  D:\lib\cmake\Drogon
  ```
  ```
  D:\lib\cmake\Trantor
  ```

* #### التثبيت عبر vcpkg في Windows

  [رابط الشرح المرئي](https://www.youtube.com/watch?v=0ojHvu0Is6A)

  **تثبيت vcpkg:**

  1. تثبيت `vcpkg` عبر `git`:

     ```
     git clone https://github.com/microsoft/vcpkg
     cd vcpkg
     ./bootstrap-vcpkg.bat
     ```

     > ملاحظة: لتحديث vcpkg لديك، يكفي أن تكتب الأمر `git pull`.

  2. أضف مسار `vcpkg` إلى متغير البيئة **_path_** في نظام Windows.
  3. للتأكد من تثبيت vcpkg بشكل صحيح، اكتب الأمر `vcpkg` أو `vcpkg.exe`.

  **الآن قم بتثبيت Drogon:**

  1. لتثبيت إطار عمل drogon، اكتب:

     * بنواة 32-Bit: `vcpkg install drogon`
     * بنواة 64-Bit: `vcpkg install drogon:x64-windows`
     * حزمة موسعة: `vcpkg install jsoncpp:x64-windows zlib:x64-windows openssl:x64-windows sqlite3:x64-windows libpq:x64-windows libpqxx:x64-windows drogon[core,ctl,sqlite3,postgres,orm]:x64-windows`

     ملاحظات:

     * إذا ظهر لك خطأ يفيد بعدم تثبيت حزمة ما، قم بتثبيتها مباشرة، مثل:

       zlib : `vcpkg install zlib` أو `vcpkg install zlib:x64-windows` لنظام 64-Bit.

     * للتحقق مما تم تثبيته بالفعل:

       ```
       vcpkg list
       ```

     * لاستخدام `drogon_ctl` اكتب `vcpkg install drogon[ctl]` (لنظام 32-bit) أو `vcpkg install drogon[ctl]:x64-windows` (لنظام 64-bit). اكتب `vcpkg search drogon` لعرض المزيد من خيارات التثبيت المتاحة.

  2. لإضافة الأمر **_drogon_ctl_** واعتمادياته، يتطلب الأمر إضافة بعض المتغيرات إلى متغيرات البيئة. باتباع هذا الدليل، تحتاج فقط إلى إضافة المسارات التالية:

     ```
     C:\dev\vcpkg\installed\x64-windows\tools\drogon
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\bin
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\lib
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\include
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\share
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\debug\bin
     ```

     ```
     C:\dev\vcpkg\installed\x64-windows\debug\lib
     ```

     إلى **_متغيرات البيئة (Environment Variables)_** في نظام Windows. ثم أعد تشغيل نافذة **_powershell_**.

  3. أعد فتح نافذة **_powershell_** واكتب: `drogon_ctl` أو `drogon_ctl.exe`. إذا ظهر لك الخرج التالي، فكل شيء يعمل بنجاح:
     ```
     usage: drogon_ctl [-v | --version] [-h | --help] <command> [<args>]
     commands list:
     create                  create some source files(Use 'drogon_ctl help create' for more information)
     help                    display this message
     press                   Do stress testing(Use 'drogon_ctl help press' for more information)
     version                 display version of this tool
     ```
     إذا ظهرت لك هذه المخرجات، فأنت جاهز للبدء.

  > ملاحظة:
  > يجب أن تكون على دراية ببناء مكتبات C++ باستخدام المترجمات المنفصلة مثل `gcc` أو `g++` (عبر **_[msys2](https://www.msys2.org/) أو [mingw-w64](https://www.mingw-w64.org/) أو [tdm-gcc](https://jmeubank.github.io/tdm-gcc/download/)_**) أو باستخدام مترجم Microsoft Visual Studio.

  > يُفضل استخدام **_make.exe/nmake.exe/ninja.exe_** كمولد لـ cmake؛ لأن سلوك إعدادها وبنائها يطابق بيئة Linux مع make، وهو ما يقلل من حدوث الأخطاء عند الانتقال بين أنظمة التشغيل المختلفة إذا كنت تخطط للرفع والنشر على بيئة Linux.

* #### استخدام صورة Docker (Docker Image)

  نوفر أيضاً صورة جاهزة ومبنية مسبقاً على [Docker Hub](https://hub.docker.com/r/drogonframework/drogon). تحتوي بيئة Docker على كافة اعتماديات Drogon وإطار العمل نفسه مثبتين بالفعل، مما يتيح للمستخدمين بناء تطبيقات قائمة على Drogon بشكل مباشر.

* #### استخدام حزمة Nix (Nix Package)

  تتوفر حزمة Nix لإطار عمل Drogon تم إصدارها بدءاً من الإصدار 21.11.

  > **إذا لم تقم بتثبيت Nix بعد:** يمكنك اتباع التعليمات المتاحة على [موقع NixOS](https://nixos.org/download.html).

  يمكنك استخدام الحزمة عن طريق إضافة ملف `shell.nix` التالي إلى مجلد مشروعك الرئيسي:

  ```
  { pkgs ? import <nixpkgs> {} }:
  pkgs.mkShell {
    nativeBuildInputs = with pkgs; [
      cmake
    ];

    buildInputs = with pkgs; [
      drogon
    ];

  }
  ```

  ادخل إلى البيئة البرمجية بتشغيل الأمر `nix-shell`. سيؤدي هذا إلى تثبيت Drogon وتنقلك إلى بيئة تحتوي على جميع اعتمادياته.

  تحتوي حزمة Nix على عدة خيارات يمكنك ضبطها حسب حاجتك:

  | الخيار (Option) | القيمة الافتراضية (Default value) |
  | :---------------: | :----------------------------------: |
  | sqliteSupport   | true                               |
  | postgresSupport | false                              |
  | redisSupport    | false                              |
  | mysqlSupport    | false                              |

  فيما يلي مثال لكيفية تغيير هذه القيم:

  ```
    buildInputs = with pkgs; [
      (drogon.override {
        sqliteSupport = false;
      })
    ];
  ```

* #### استخدام CPM.cmake

  يمكنك استخدام [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake) لتضمين كود drogon المصدري:

  ```cmake
  include(cmake/CPM.cmake)

  CPMAddPackage(
      NAME drogon
      VERSION 1.7.5
      GITHUB_REPOSITORY drogonframework/drogon
      GIT_TAG v1.7.5
  )

  target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
  ```

* #### تضمين كود drogon المصدري محلياً

  بالطبع يمكنك أيضاً تضمين كود drogon المصدري داخل مشروعك مباشرة. بافتراض أنك وضعت مجلد drogon داخل المجلد الفرعي `third_party` لمشروعك (لا تنسَ تحديث الموديول الفرعي submodule داخل مجلد drogon). بعدها، كل ما عليك فعله هو إضافة السطرين التاليين إلى ملف cmake الخاص بمشروعك:

  ```cmake
  add_subdirectory(third_party/drogon)
  target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
  ```

# التالي: [بداية سريعة](/ARA/ARA-03-Quick-Start)

</div>
