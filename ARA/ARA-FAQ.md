<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-FAQ) - [English](/ENG/ENG-FAQ)

# الأسئلة الشائعة (FAQ)

هذه قائمة بالأسئلة الشائعة والإجابات الخاصة بها، مع بعض الشرح والتوضيح الموسع.

## ما هو نموذج خيوط التنفيذ (Threading model) في Drogon وما هي أفضل الممارسات؟

يعمل Drogon على مجمع خيوط تنفيذ (Thread pool)، حيث تُنشأ خيوط تنفيذ خادم HTTP بالإضافة إلى خيوط تنفيذ قواعد البيانات عند استدعاء الدالة `app().run()`. وهو نظام قائم على المهام المتسلسلة (Sequential task-based system). لذلك يُوصى دائماً باستخدام واجهات البرمجيات اللامتزامنة (Asynchronous APIs) أو الروتينات الفرعية المشتركة (Coroutines) كلما أمكن ذلك. راجع دليل [فهم نموذج خيوط التنفيذ في Drogon](/ARA/ARA-FAQ-1-Understanding-drogon-threading-model) لمزيد من التفاصيل.

</div>
