[English](ENG-15-Coz) | [简体中文](CHN-14-Coz分析)

## 使用coz进行因果分析

使用coz，您可以分析两件事：

* 吞吐量
* 延迟

如果要分析应用程序的吞吐量，则应打开“COZ_PROFILING”cmake选项，并在可执行文件中使用cmake中的`Debug`或`RelWithDebInfo`发布模式包含调试信息。这样做将在处理请求时包括coz进度点。目前,分析延迟在整个应用程序范围内不受支持，但仍可以在用户代码中完成。

编译完包含进度点的应用程序后。您需要使用coz分析器运行可执行文件，例如“coz run --- [可执行文件的路径]”。

最后，应用程序需要进行压力测试，为了获得最佳结果，您需要压测所有代码路径并进行一个长时间的分析，15+分钟。

在当前工作路径下会生成一个最终的分析文件`profile.coz`。要查看结果，请在官方[viewer](https://plasma-umass.org/coz/) 中打开分析文件，或者您可以从官方[git repo](https://github.com/plasma-umass/coz) 下载副本到本地进行查看。

Coz还支持使用 `--source-scope <pattern>` 或 `-s <pattern>` 等其他方法对分析文件中包含的源文件进行范围限定，这应该是有用的。

更多信息请查看：

- `coz run --help`
- [Git repo](https://github.com/plasma-umass/coz)
- [Coz whitepaper](https://arxiv.org/pdf/1608.03676v1.pdf)

# 15 [Brotli 压缩](CHN-15-Brotil-压缩)
