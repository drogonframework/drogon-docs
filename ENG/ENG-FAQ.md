<<<<<<< HEAD
##### Other languages: [简体中文](/CHN/CHN-FAQ)
=======
##### Other languages: [简体中文](/CHN/CHN-FAQ)
>>>>>>> 557192c59e37288b16a0d6dd1d9116101f5a08e1

# FAQ

This is a list of commonly asked questions and the answers, with some extended explanation.

## What's drogon's threading model and best practices?

<<<<<<< HEAD
Drogon runs on a thread pool where the HTTP server threads as well as DB threads are created when `app().run()` is called. It is a sequential task based system. Thus it is recommended to always use the asynchronous APIs or coroutines when possible. See [Understanding drogon's threading model](/ENG/ENG-FAQ-1-Understanding-drogon-threading-model) for detail.
=======
Drogon runs on a thread pool where the HTTP server threads as well as DB threads are created when `app().run()` is called. It is a sequential task based system. Thus it is recommended to always use the asynchronous APIs or coroutines when possible. See [Understanding drogon's threading model](/ENG/ENG-FAQ-1-Understanding-drogon-threading-model) for detail.
>>>>>>> 557192c59e37288b16a0d6dd1d9116101f5a08e1
