# Ionic/Cordova的笔记

## 1. Android原生集成Ionic——以打包成aar的方式

集成Ionic应用到Android原生应用中，可以采用Cordova提供的内嵌一个CordovaWebView的方式，但是实际上，Ionic应用在Cordova的基础上加入了一些自己特有的打包方式，工程的目录结构与标准的Android Studio工程的目录结构存在一定的差异，配置上需要做较多的修改。这里采用了一种较为简单的方式，以aar的形式，将Ionic工程的资源文件全部打包，然后在Android原生应用引入aar包即可。这种方式较为简单，只需要在Ionic工程中做一些简单的修改即可，具体的操作如下：

* ### 修改Ionic工程：由application修改为libray

打开`build.gradle`文件，













































