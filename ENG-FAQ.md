# FAQ

This is a list of commonly asked questions and the answers, with some extended explnation.

## What's drogon's threading model and best prectices?

Drogon runs on a thread pool where the HTTP server threads as well as DB threads are created when `app().run()` is called. It is a sequential task based system. Thus it is recommended to always use the asynchronous APIs or coroutines when possible. See [Understanding drogon's threading model](ENG-FAQ-1-Understanding-drogon-threading-model.md) for detail.
