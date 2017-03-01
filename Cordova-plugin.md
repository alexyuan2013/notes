# Cordova插件相关
## 1. Cordova插件开发指南——简译官方文档

Cordova插件是Cordova WebView与平台本地代码交互的桥梁，其使得Cordova应用拥有了其他web应用所没有的与设备和平台功能交互的能力，如照相机、NFC、日历接口等。
### 创建一个插件
这里通过device插件的结构来说明cordova插件的代码结构。通过cli添加device插件：

`cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-device.git`

插件代码的根目录必须包含一个`plugin.xml`的配置文件，下面就是device插件的配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plugin xmlns="http://apache.org/cordova/ns/plugins/1.0"
        id="cordova-plugin-device" version="0.2.3">
    <name>Device</name>
    <description>Cordova Device Plugin</description>
    <license>Apache 2.0</license>
    <keywords>cordova,device</keywords>
    <js-module src="www/device.js" name="device">
        <clobbers target="device" />
    </js-module>
    <platform name="android">
        <config-file target="res/xml/config.xml" parent="/*">
            <feature name="Device" >
                <param name="android-package" value="org.apache.cordova.device.Device"/>
            </feature>
        </config-file>

        <source-file src="src/android/Device.java" rget-dir="src/org/apache/cordova/device" />
    </platform>
    <platform name="ios">
        <config-file target="config.xml" parent="/*">
            <feature name="Device">
                <param name="ios-package" value="CDVDevice"/>
            </feature>
        </config-file>
        <header-file src="src/ios/CDVDevice.h" />
        <source-file src="src/ios/CDVDevice.m" />
    </platform>
</plugin>
```
根部标签`<Plugin>`的`id`属性作为device插件的标识，在cli中也可以通过`id`来安装插件：

`cordova plugin add cordova-plugin-device`

`<js-module>`标签定义了插件javascript端的接口，其内部标签`<clobbers>`表示将插件的对象绑定到全局对象上，即在js代码中可以通过`window.device`或者直接使用`device`变量来访问插件对象的属性。详细的解释可以参考官方的[Reference](https://cordova.apache.org/docs/en/latest/plugin_ref/spec.html)。

`<platform>`标签定义了对应平台Android、iOS等的配置信息，包括本地代码的源文件路径和完整的包名等。

### 使用plugman查看插件的可用性

使用npm全局安装plugman：`npm install -g plugman`；具体使用的操作参考[这里](http://cordova.apache.org/docs/en/latest/plugin_ref/plugman.html),不再赘述。

### JavaScript接口
JS代码与Native代码交互的接口为cordova.exec()，用法如下：

```
cordova.exec(function(winParam) {},
             function(error) {},
             "service",
             "action",
             ["firstArgument", "secondArgument", 42, false]);
```

参数的意义如下：
- `function(winParam){}`，native代码执行成功后的回调函数
- `function(error){}`，native代码执行失败后的回调函数
- `service`，通常对应于native代码中的类。如在Android系统上，对应的就是Java的类名
- `action`，对应native代码中的方法
- `arguments`，action方法所需要的参数列表

### Native接口
这部分内容以目前能力主要是对Android平台来进行，这就主要针对Android平台介绍。

- 插件类的映射

Cordova的CLI会根据plugin.xml文件中的`<feature>`标签的配置，注入到platform/android下的/res/xml/config.xml文件中，原生代码根据config.xml中的配置找到对应的原生方法（应该是用反射实现的）。
```xml
<feature name="<service_name>">
    <param name="android-package" value="<full_name_including_namespace>" />
</feature>
```
其中`service_name`与JavaScript接口中的`exec`中`service`参数相匹配。Java类的引用一定要使用完整的包名。 

- 插件的初始化和生命周期

当Webview启动时，插件并不会生成实例对象，而是在插件JavaScript接口中第一次调用后完成实例化，除非在配置文件中加上启动参数，插件才会在Webview启动时立刻完成实例化。参数形式如下：

```xml
<feature name="service_name">
    <param name="android-package" value="<full_name_including_namespace>" />
    <param name="onload" value="true" />
</feature>
```
可以通过重载`initialize()`方法来完成插件的初始化工作：

```java
@Override
public void initialize(CordovaInterface cordova, CordovaWebView webView) {
    super.initialize(cordova, webView);
    // your init code here
}
```
插件也可以访问Android生命周期并通过重载`onResume/onDestroy`等方法，如果插件需要在后台模式下长时间的请求，那么需要重载`onReset`方法，该方法会在Webview跳转到新页面或者刷新等，需要重新加载js时执行。（`onReset`方法的用处这里不是很清楚，为什么后台模式下的长时间请求要调用该方法？而`onReset`的默认实现是什么都不做，不解）

- 编写Android插件
至少包含一个`CordovaPlugin`的子类并且至少重载了一个`execute`方法:
```java
@Override
public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
    if ("beep".equals(action)) {
        this.beep(args.getLong(0));
        callbackContext.success();
        return true;
    }
    return false;  // Returning false results in a "MethodNotFound" error.
}
```
其中，`action`与js端的cordova.exec()方法中的参数`action`相对应，通常在`execute`方法中更具`action`来调用相应的方法；`args`为对应的参数列表；`callbackContext`为回调时的上下文环境。

- 线程

插件的js代码一般运行在`Webcore`线程，而不是`Webview`的主线程。如果需要与用户界面交互，那么需要在`Webview`主线程上进行操作，这时候需要调用`Activity`的`runOnUiThread`方法。
```java
@Override
public boolean execute(String action, JSONArray args, final CallbackContext callbackContext) throws JSONException {
    if ("beep".equals(action)) {
        final long duration = args.getLong(0);
        cordova.getActivity().runOnUiThread(new Runnable() {
            public void run() {
                ...
                callbackContext.success(); // Thread-safe.
            }
        });
        return true;
    }
    return false;
}
```
如果不想运行在UI线程，也不想阻塞`Webcore`线程，如文件传输操作，那么可以使用Cordova `ExecutorService`的`execute`方法，该方法通过`cordova.getThreadPool()`获得:
```java
@Override
public boolean execute(String action, JSONArray args, final CallbackContext callbackContext) throws JSONException {
    if ("beep".equals(action)) {
        final long duration = args.getLong(0);
        cordova.getThreadPool().execute(new Runnable() {
            public void run() {
                ...
                callbackContext.success(); // Thread-safe.
            }
        });
        return true;
    }
    return false;
}
```

- 添加依赖库

在`plugin.xml`文件中通过`<framework>`标签添加`gradle`依赖：
```xml
<!-- Depend on latest version of GCM from play services -->
<framework src="com.google.android.gms:play-services-gcm:+" />
<!-- Depend on v21 of appcompat-v7 support library -->
<framework src="com.android.support:appcompat-v7:21+" />
<!-- Depend on library project included in plugin -->
<framework src="relative/path/FeedbackLib" custom="true" />
```
另一种方式是`<lib-file>`标签引入`jar`包来实现，这种方式最好不用，除非确定引入的`jar`包，别的插件不会引用，否则就会引入多个相同的`jar`包，导致编译错误。

- Android权限配置

通过在`plugin.xml`的`<platform name="android">`中引入如下配置,即可在`AndroidManifest.xml`文件中注入相应的权限配置。
```xml
<config-file target="AndroidManifest.xml" parent="/*">
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
</config-file>
```
- 运行时权限

Android系统从6.0开始，介绍一种全新的权限模型，允许用户在App运行时进行权限的申请；Cordova从5.0开始支持这一新的权限申请模型。可以通过下面的方法进行申请：
```
cordova.requestPermission(CordovaPlugin plugin, int requestCode, String permission);
```
其中，`plugin`一般就是类本身，即`this`；`requestCode`为一个权限的编码，用户可以自己定义，据我的理解是，当一个插件有多个权限需要申请时，用`requestCode`作不同权限的标识；`permission`对应于`Manifest.permission`中定义好的权限字符串。以下为一段示例代码：
```java
//定义一个READ权限
public static final String READ = Manifest.permission.READ_CONTACTS;
//定义requestCode
public static final int SEARCH_REQ_CODE = 0;
//检查权限
if(cordova.hasPermission(READ))
{
    //执行任务
    search(executeArgs);
}
else
{
    //申请权限
    getReadPermission(SEARCH_REQ_CODE);
}
//申请权限方法
protected void getReadPermission(int requestCode)
{
    cordova.requestPermission(this, requestCode, READ);
}
//申请权限后，onRequestPermissionResult()方法将响应用户的操作：允许或者不允许
public void onRequestPermissionResult(int requestCode, String[] permissions, int[] grantResults) throws JSONException
{
    for(int r:grantResults)
    {
        if(r == PackageManager.PERMISSION_DENIED)
        {
            this.callbackContext.sendPluginResult(new PluginResult(PluginResult.Status.ERROR, PERMISSION_DENIED_ERROR));
            return;
        }
    }
    switch(requestCode)
    {
        case SEARCH_REQ_CODE:
            search(executeArgs);
            break;
        case SAVE_REQ_CODE:
            save(executeArgs);
            break;
        case REMOVE_REQ_CODE:
            remove(executeArgs);
            break;
    }
}
```
- Reference
  - [Plugin Development Guide](http://cordova.apache.org/docs/en/latest/guide/hybrid/plugins/index.html)
  - [Android Plugin Development Guide](http://cordova.apache.org/docs/en/latest/guide/platforms/android/plugin.html)
  - 感谢[chenliyu](https://github.com/chenliyu)提供的文档参考
