<div id="rtl-content" lang="ar">
<style>
  @import url('./ARA/style.css');
</style>

##### لغات أخرى: [简体中文](/CHN/CHN-04-控制器-简介) - [English](/ENG/ENG-04-0-Controller-Introduction)

# المتحكم (Controller) - مقدمة (Introduction)

يُعد المتحكم (Controller) مفهوماً مهماً جداً في تطوير تطبيقات الويب. ففيه نقوم بتحديد عناوين URL، وطرق HTTP المسموح بها، والفلاتر [filters](/ARA/ARA-05-Middleware-and-Filter) التي سيتم تطبيقها، وكيفية معالجة الطلبات والاستجابة لها. لقد قام إطار عمل Drogon بالتعامل مع عمليات النقل الشبكي وتحليل بروتوكول HTTP وغيرها نيابة عنا، وكل ما نحتاجه هو التركيز على المنطق الخاص بالمتحكم فقط؛ يمكن لكائن المتحكم أن يحتوي على دالة معالجة واحدة أو أكثر (تُسمى عادة المعالجات Handlers)، وتُعرف واجهة الدالة عادة كالتالي:

```c++
void handlerName(const HttpRequestPtr &req,
                  std::function<void (const HttpResponsePtr &)> &&callback,
                 ...);
```

حيث يُمثل `req` كائن طلب HTTP (مغلف بواسطة مؤشر ذكي Smart pointer)، و`callback` هو كائن دالة الاستدعاء اللاحق الذي يمرره إطار العمل إلى المتحكم. يقوم المتحكم بتوليد كائن الاستجابة (المغلف أيضاً بمؤشر ذكي) ثم يمرر هذا الكائن إلى Drogon عبر الـ callback. يتولى إطار العمل بعد ذلك إرسال محتوى الاستجابة إلى المتصفح نيابة عنك. أما الجزء الأخير `...` فيُمثل قائمة من المعاملات. يقوم Drogon بتعيين المعاملات الواردة في طلب HTTP إلى المعاملات المناظرة لها في الدالة وفقاً لقواعد التعيين (Mapping rules). يُعد هذا الأمر مريحاً للغاية أثناء تطوير التطبيقات.

من الواضح أن هذه واجهة لامتزامنة (Asynchronous interface)؛ حيث يمكن إجراء العمليات التي تستغرق وقتاً طويلاً في خيوط تنفيذ (Threads) أخرى، ثم استدعاء الـ callback بعد الانتهاء.

يوفر Drogon ثلاثة أنواع من المتحكمات: HttpSimpleController، و HttpController، و WebSocketController. عند استخدام أي منها، يلزم الوراثة من قالب الفئة (Class template) المناظر. على سبيل المثال، يكون الإعلان عن فئة مخصصة باسم "MyClass" ترث من HttpSimpleController كالتالي:

```c++
class MyClass:public drogon::HttpSimpleController<MyClass>
{
public:
    //TestController(){}
    virtual void asyncHandleHttpRequest(const HttpRequestPtr &req,
                                        std::function<void (const HttpResponsePtr &)> &&callback) override;

    PATH_LIST_BEGIN
    PATH_ADD("/json");
    PATH_LIST_END
};
```

### دورة حياة المتحكم (Controller life cycle)

كائن المتحكم المسجل في إطار عمل Drogon يحتوي على مثيل واحد فقط (Single instance) كحد أقصى ولا يتم تدميره طوال فترة تشغيل التطبيق. لذلك يمكن للمستخدمين الإعلان عن متغيرات الأعضاء (Member variables) واستخدامها داخل فئة المتحكم. يُرجى الانتباه إلى أنه عند استدعاء معالج المتحكم (Handler)، فإن ذلك يتم في بيئة متعددة الخيوط (Multi-threaded environment) (عند إعداد عدد خيوط الإدخال/الإخراج IO threads لإطار العمل ليكون أكثر من 1). إذا كنت بحاجة للوصول إلى متغيرات غير مؤقتة، يُرجى التأكد من تنفيذ آليات الحماية من التزامن (Concurrent protection).

# التالي: [HttpSimpleController](/ARA/ARA-04-1-Controller-HttpSimpleController)

</div>
