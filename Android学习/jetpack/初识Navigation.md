# 初识Navigation(Google为 单Activity+多Fragment 开发方式提供的官方支持)

## 简单介绍下新的android支持包Jetpack

Android Developer上面挂着Jetpack已经很久了。Jetpack是一组基于Android平台的库。Jetpack中包含四个层面的东西

* Foundation
  * AppCompat
  * Android KTX
  * Multidex
  * Test
* Architecture
  * Data Binding
  * Lifecycles
  * LiveData
  * Navigation
  * Paging
  * Room
  * ViewModel
  * WorkManager
* Behavior
  * Download manager
  * Media & playback
  * Notifications
  * Permissions
  * Sharing
  * Slices
* UI
  * Animation & transitions
  * Auto
  * Emoji
  * Fragment
  * Layout
  * Palette
  * TV
  * Wear Os by Google

Android Jetpack的具体介绍可以参看官方文档：
[Android Jetpack](https://developer.android.google.cn/jetpack/)

总结来说，以后Android将不再提供v4/v7/v13/v14等包的更新，统一到Jetpack库中，并以androidx.作为新的包开头。官方将通过Jetpack提供对Android各平台的兼容，并且Jetpack独立于Android各平台API进行更新，所以以后一般情况下我们只需要用最新的androidx.包就可以了，全部使用一套接口来兼容各平台。

## Jetpack中的Navigation(位于Architecture中)

今天我们体验下Jetpack中的Navigation库，Navigation库是官方为单Activity开发提供的官方支持。官方为Navigation开发了IDEA的插件可以使用图像化手段处理页面跳转关系；除了采用Bundle进行数据传输以外还提供了gradle插件帮助开发者使用图形化界面来传递参数；另外还提供了各个跳转之间的切换动画(Navigation中也提供了跳转到Activity的支持，但是暂时并没有提供从Activity跳转到其他页面的支持)。

## 开始构建一个使用Navigation的项目

### 安装Android Studio 3.2版本

要想在Android Studio中使用Navigation架构组件就必须使用Android Studio3.2或以上版本。而目前Android Studio3.2版本还处于Beta状态，所以为了提前体验新功能我们只能下载Beta版尝尝鲜 [Android Studio 3.2下载链接1](https://dl.google.com/dl/android/studio/ide-zips/3.2.0.22/android-studio-ide-181.4913314-windows.zip)。[Android Studio 3.2下载链接2](https://developer.android.google.cn/studio/preview/)。

> 下载安装后，导入配置文件时如果导入了老版本的Android Studio的配置文件，那么老版本中的项目列表将同样会显示在Android Studio3.2中。

### 在项目中使用Navigation

#### 添加依赖

安装好3.2版本的Android Studio之后，可以单独创建一个项目，也可以打开之前的项目。总之打开一个项目，并在其build.gradle文件中配置Navigation支持

```gradle
dependencies {
    implementation 'android.arch.navigation:navigation-fragment:1.0.0-alpha05'
    implementation 'android.arch.navigation:navigation-ui:1.0.0-alpha05'

    androidTestImplementation 'android.arch.navigation:navigation-testing:1.0.0-alpha05'
}
```

Navigation中的包命名方式已经改为以androidx.为包首，但是当前版本依然依赖有android support相关包。

#### 创建Navigation资源文件

在资源目录(res目录)中创建navigation文件夹，并在其中创建xml文件，并在其中创建文件nav_graph.xml。文件内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android">
</navigation>
```

因为目前Android Studio3.2以及Navigation都还处于Beta阶段，所以支持还不是很完善，创建该xml文件时并不像layout、anim这些有直接的选项(最新版本3.3Canary或许已经有了，可以下载试下)。

#### 开启Android Studio对Navigation的图形界面支持

在Android Studio 3.2版本中对Navigation的图像化支持默认没有开启，需要我们手动打开。将Settings->Experimental界面中最下面*Enable Navigation Editor*勾上即可。

#### 操作Navigation的图形化界面、熟悉Navigation使用

![navigation](./img/navigation-editor.png)

如上所示即为Navigation的图形化界面。其中分为三个部分：

1. Destinations列表，包含当前界面展示的所有的destination
2. Navigation图形化编辑界面
3. 属性编辑器，用于编辑图形化界面选中的元素属性

> destinations 指被Navigation操作的部分，可能是Activity、Fragment或者我们自定义的其他东西(我们可以自己实现一套导航方案)。