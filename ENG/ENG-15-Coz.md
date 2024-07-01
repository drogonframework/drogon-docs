##### Other languages: [简体中文](/CHN/CHN-14-Coz分析)

## Causal profiling with coz

With coz you can profile two things:

* throughput
* latency

If you want to profile throughput of your application, you should switch on the `COZ_PROFILING` cmake option and include debug information in your executable with `Debug` or `RelWithDebInfo` release modes in cmake. Doing so will include coz progress points when serving a request. Profiling latency is currently not supported in whole application scope, but can still be done in user code.

When you're done compiling you application with progress points included. You need to run the executable with the coz profiler, for example `coz run --- [path to your executable]`.

Lastly, the application needs to be stressed, for best results you need to stress all code paths and run the profile for a good amount of time, 15+ min.

The final profile will be a `profile.coz` file created in the current working directory. To view results, open the profile in the official [viewer](https://plasma-umass.org/coz/), or you could run a local copy from the official [git repo](https://github.com/plasma-umass/coz).

Coz also supports scoping source files included for the profile with `--source-scope <pattern>` or `-s <pattern>` among other things, that should prove useful.

For more information checkout:

* `coz run --help`
* [Git repo](https://github.com/plasma-umass/coz)
* [Coz whitepaper](https://arxiv.org/pdf/1608.03676v1.pdf)


# 15 [Brotli compression](/ENG/ENG-15-Brotli)