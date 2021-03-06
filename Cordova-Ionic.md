# Ionic/Cordova的笔记

## 1. Android原生集成Ionic——以打包成aar的方式

集成Ionic应用到Android原生应用中，可以采用Cordova提供的内嵌一个CordovaWebView的方式，但是实际上，Ionic应用在Cordova的基础上加入了一些自己特有的打包方式，工程的目录结构与标准的Android Studio工程的目录结构存在一定的差异，配置上需要做较多的修改。这里采用了一种较为简单的方式，以aar的形式，将Ionic工程的资源文件全部打包，然后在Android原生应用引入aar包即可。这种方式较为简单，只需要在Ionic工程中做一些简单的修改即可，具体的操作如下：

### 修改Ionic工程：由application修改为library

从Ionic工程的platform/android文件夹，将Android平台的代码导入到Android Studio中，工程的目录结构如下：

```
--CordovaLib
--android
--Gradle Scripts
----build.gradle(Module: CordovaLib)
----cordova.gradle(Module: CordovaLib)
----build.gradle(Module: android)
----......(其他文件)
```
其中CordovaLib为Ionic依赖的Cordova代码，其本身就是一个library，因此可以直接打包成aar格式。而android则为Ionic自身封装的应用工程，需要做的就是修在该工程的`build.gradle`文件：

```
apply plugin: 'com.android.application'
修改为：
apply plugin: 'com.android.library'
```
然后同步一下gradle，会报错`Library projects cannot set applicationId`的错误，因为我们将application修改为了library，所以只要将`build.gradle`文件中的`applicationId`配置删除即可。

此时如果进行打包，由于AndroidManifest.xml文件为应用的配置，所以会在桌面上显示两个app的图标，因此需要将其中的

`<category android:name="android.intent.category.LAUNCHER"/>`

配置删除，避免两个app图标的情况发生。

### 使用Gradle打包工程为aar格式文件

打开Gradle的面板，其位于Android Studio界面的右侧，目录大概是这样的：

```
--android
----tasks
------android
------build
------help
------install
------other
------verification
```
在`build`下找到`assembleRelease`选项,右键选择`run`即可。

最后再项目目录的/build/output/aar文件夹中会看到生成的aar文件。

同时打包CordovaLib，aar文件在CordovaLib的对应文件中找到

### 添加aar到其他原生项目中

打开Android Studio的Project视图，将生成的两个aar文件复制到libs目录，如果没有，则在与src平级的目录小新建libs文件夹。修改build.gradle文件，添加如下信息：

```
allprojects {
    repositories {
        jcenter()
        flatDir {
            dirs 'libs'
        }
    }
}

dependencies {
    ...
    compile(name:'CordovaLib-release', ext:'aar')
    compile(name:'android-release', ext:'aar')
    ...
}
```
然后可以在原生项目中通过Intent的方式来启动Ionic项目的Activity，下面是一段简单的调用：

```java
Button btn = (Button) findViewById(R.id.start_ionic_btn);
btn.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        Intent intent = new Intent(MainActivity.this, com.nari.ipark.MainActivity.class);
        startActivity(intent);
    }
});
```
此外，还有可能遇到`icon error, Manifest Merger`的错误，即打包的aar中Manifest文件与原生应用Manifest文件的配置有冲突，解决的方式是使用原生应用的配置覆盖aar包的配置，在原生AndroidManifest.xml文件的`manifest`标签下加入`xmlns:tools="http://schemas.android.com/tools"`，再向`application`标签加入`tools:replace="android:icon,android:theme"`即可。

### 参考
  * [Convert existing project to library project in Android Studio](http://stackoverflow.com/questions/17614250/convert-existing-project-to-library-project-in-android-studio)
  * [How to manually include external aar package using new Gradle Android Build System](http://stackoverflow.com/questions/16682847/how-to-manually-include-external-aar-package-using-new-gradle-android-build-syst)
  * [Android Studio 1.0 and error “Library projects cannot set applicationId”](http://stackoverflow.com/questions/27374933/android-studio-1-0-and-error-library-projects-cannot-set-applicationid)
  * [Android studio Gradle icon error, Manifest Merger](http://stackoverflow.com/questions/24506800/android-studio-gradle-icon-error-manifest-merger)
  * [how to hide app's icon correctly?](http://stackoverflow.com/questions/35098439/how-to-hide-apps-icon-correctly)
  * [How to change android version and code version number in Android Studio?](http://stackoverflow.com/questions/22274657/how-to-change-android-version-and-code-version-number-in-android-studio)

## 2. 在Android原生项目中嵌入CordovaWebView以集成Ionic工程

这种方式是Cordova官方给出的方式，这里做了一些简化的处理，基本是对官方文档的一个翻译。

1. 下载最新的Cordova发布版本，从[官网](https://dist.apache.org/repos/dist/release/cordova/platforms/)找到Android平台对应的发布版本，解压备用。
2. 打开`/framework`目录，使用`ant jar`进行打包，得到`/framework/cordova-x.x.x.jar`文件。
3. 复制`.jar`文件到Android原生项目的`libs`中。
4. 这一步官方文档是采用自己重新实现类似于`CorodovaActivity`的Activity，这里我们不采用这种方式，因为Ionic工程已经通过`extends`的方式实现了一个`MainActivity`，我们可以直接使用这个`Activity`。
5. 将Ionic工程所用到的Cordova插件的源代码复制到原生工程的`src`目录下，同时，由于这些插件可能还引入了其他的`jar`包，需要将Ionic工程中`libs`下的所有`jar`包复制到原生应用的`libs`下，否则Cordova会初始化失败。
6. 将Ionic的`www`文件夹复制到原生工程的`assets`目录下，如果没有`assets`目录，请使用Android Studio自行新建一个。
7. 将Ionic中的`/res/xml/config.xml`文件复制到原生工程的对应目录下，如果没有`xml`目录，请自行创建。

至此，Android原生项目就可以通过`Intent`的方式来启动已有的Ionic工程。

实际上，Ionic通过`ionic run android`命令以后所生成的项目已经是一个Android原生的项目，代码级的集成相对来说也比较容易。






