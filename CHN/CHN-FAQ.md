[English](/ENG/ENG-FAQ) | [简体中文](/CHN/CHN-FAQ)

# FAQ

这是常见问题和答案的列表，与一展说明。

## What's drogon's threading model and best practices?

Drgon在线程池上运行，当调用`app().run()`时，会在该线程池中创建HTTP服务器线程和数据库线程。 它是一个基于顺序任务的系统。 因此，建议在可能的情况下始终使用异步API或协程。 详见[理解drogon的线程模型](/CHN/CHN-FAQ-1-线程模型)。
