[English](ENG-09-Plugins) | [简体中文](CHN-09-插件)

插件可以帮助用户构建复杂的应用，在Drogon中，所有的插件都由框架根据配置文件统一构建并安装到应用程序中。Drogon中的插件都是单实例的，用户可以用插件实现任何他们想要的功能。

Drogon在运行run()接口的时候，会根据配置文件，逐一实例化每个插件，并调用其`initAndStart()`接口。

### 配置

插件配置都在配置文件中完成，例如：

```json
    "plugins": [
        {
        //name: The class name of the plugin
        "name": "DataDictionary",
        //dependencies: Plugins that the plugin depends on. It can be commented out
        "dependencies": [],
        //config: The configuration of the plugin. This json object is the parameter to initialize the plugin.
        //It can be commented out
        "config": {
        }
        }],
```

可见，每个插件的配置共有三项：

* name: 是该插件的类名称(包含命名空间)，框架会根据这个类名称创建插件的实例，这种反射机制在控制器、过滤器的实例创建中也用到过；注释掉该项，则该插件变成禁用状态；
* dependencies: 是该插件所依赖的其他插件的名称列表，框架会按照顺序创建和初始化它们，被依赖的插件首先被创建并初始化；程序结束时，按相反的顺序关闭并销毁它们；需要注意，不要在插件中产生循环依赖，Drogon会检查循环依赖，如果发现，会报错并退出程序；注释掉该项，则依赖列表为空；
* config: 用于初始化该插件的json对象，用户可以在其中自定义任何配置，该对象将被作为入参传入到该插件的`initAndStart()`接口中。注释掉该项，则传入initAndStart接口的json对象为空对象；

### 定义

用户定义的插件必须继承自drogon::Plugin类模板，模板参数就是该插件类型，比如下面的定义：

```c++
class DataDictionary : public drogon::Plugin<DataDictionary>
{
public:
    virtual void initAndStart(const Json::Value &config) override;
    virtual void shutdown() override;
    ...
};
```

当然，命令行程序drogon_ctl提供了创建插件的命令，如下：

```shell
drogon_ctl create plugin <[namespace::]class_name>
```

用户可以用上述命令创建插件的源文件，然后再进一步编辑它们；

### 获取实例

插件的实例由drogon创建，用户可以通过drogon的如下接口获取插件实例：

```c++
template <typename T> T *getPlugin();
```

或者

```c++
PluginBase *getPlugin(const std::string &name);
```

显然，第一个方法更方便，例如，前文提到的DataDictionary插件可以这样获取：

```c++
auto *pluginPtr=app().getPlugin<DataDictionary>();
```

注意，获取插件最好在框架的run()接口调用之后，否则会得到未初始化的插件实例(这并不一定必然导致错误，只要在使用它时初始化已经完成即可)。当然，由于插件是按照依赖顺序初始化的，所以，只要依赖关系设置正确，在`initAndStart()`接口中获取其他插件的实例是没有问题的。

### 生命周期

所有插件在run()接口内初始化完毕，在应用程序退出时才销毁，因此，插件的生命周期几乎和应用程序等同，这也是getPlugin()接口不需要返回智能指针的原因。

# 10 [配置文件](CHN-10-配置文件)
