##### Other languages: [简体中文](/CHN/CHN-FAQ)

# FAQ

This is a list of commonly asked questions and the answers, with some extended explanation.

## What's drogon's threading model and best practices?

Drogon runs on a thread pool where the HTTP server threads as well as DB threads are created when `app().run()` is called. It is a sequential task based system. Thus it is recommended to always use the asynchronous APIs or coroutines when possible. See [Understanding drogon's threading model](/ENG//ENG/ENG-FAQ-1-Understanding-drogon-threading-model) for detail.
