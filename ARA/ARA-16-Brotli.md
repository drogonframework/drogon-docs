<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-16-Brotli压缩) - [English](/ENG/ENG-16-Brotli)

## معلومات حول ضغط Brotli (Brotli Info)

يدعم إطار العمل Drogon الملفات المضغوطة مسبقاً بتقنية Brotli (الضغط الثابت Static compression) بشكل افتراضي وبدون أي إعدادات إضافية، وذلك في حالة وجود الملف المضغوط المناظر بجانب المورد الأصلي.

على سبيل المثال، سيبحث Drogon عن المسار `/path/to/asset.js.br` عند ورود طلب للمسار `/path/to/asset.js`.

ويقوم Drogon بذلك عن طريق ضبط خيار `br_static` على `true` افتراضياً داخل ملف الإعدادات `config.json`.

أما إذا كنت ترغب في تطبيق الضغط الديناميكي (Dynamic compression) باستخدام Brotli أثناء التشغيل، فيجب عليك ضبط الخيار `use_brotli` على `true` داخل ملف `config.json`.

بالنسبة للمطورين الذين لا ينوون استخدام الضغط الثابت بـ Brotli، يمكنهم الاستغناء عن عملية التحقق الإضافية للملف المجاوز (Sibling check) عبر ضبط الخيار `br_static` على `false` داخل ملف `config.json` لرفع الأداء.

# التالي: [الروتينات الفرعية المشتركة (Coroutines)](/ARA/ARA-17-Coroutines)

</div>
