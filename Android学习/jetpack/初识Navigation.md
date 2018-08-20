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

> destinations 指被Navigation操作的目标组件，目前可以是Activity、Fragment或者我们自定义的其他东西(navgation中给我们提供了几个核心类可以用于实现自己的目标组件)。

#### 创建目标组件

上面提到目标组件可以是Activity、Fragment或者自定义组件。目标组件的创建就是创建这些对象，但是创建完成后需要将其加入到Navigation组件中去通过Navigation控制。加目标组件到Navigation中如下所示

![添加导航](./img/navigation_identify.png)

点击图中被小框框出来的按钮即会打开一个列表框，这个列表框中会列出所有尚未添加到nav_graph文件中(即尚未接入到Navigation中)的目标组件。另外还可以通过列表上部的'Create blank destination'按钮创建一个新的目标组件出来。

我们现在通过'Create blank destination'按钮来创建一个目标组件BlankFragment，通过Create blank destination按钮创建的目标组件将自动加入到Navigation视图中，点击视图中的该组件在属性框列表将有如下属性：

* Type  该目标组件的类型，可以是Fragment、Activity、自定义类型
* Label 该目标组件在Navigation视图中的name
* ID    ID最为重要，在代码中使用Navigation时需要通过该ID去定位目标组件
* Class 该属性指向目标组件的具体类

点击Navigation视图中的Text标签，看Navigation视图的XML定义。现在Navigation视图文件中多了一个`<fragment>`标签，该标签即用于表示我们刚刚添加的BlankFragment，可以看到除了四个属性都在该标签中有所表示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/blankFragment">

    <!--type属性-->
    <fragment
        <!-- id属性-->
        android:id="@+id/blankFragment"
        <!-- class属性-->
        android:name="com.olair.architecture.BlankFragment"
        <!-- label属性-->
        android:label="fragment_blank"
        <!-- tool:layout 用于预览 -->
        tools:layout="@layout/fragment_blank" />
</navigation>
```

> 在这个XML中navigation标签中有一个app:startDestination 属性，该属性用于表示Navigation的最开始的目标组件，即起始目标组件。
> 要设置起始目标组件可以在Navigation视图窗格中点击想要设置为起始目标组件的组件，在属性列表的下侧会有一个'Set Start Destination'按钮，点击即可设置，当然也可以以XML的形式去编辑。
> 设置为起始目标组件之后，可以看到在目标组件上方会出现一个小屋子的图标。

涉及到的Navigatin视图中的按钮如下：

![视图中的点击选项](Blog/Android学习/jetpack/img/navigation-clicked.png)

#### 连接到目标组件

当我们有多个目标组件的时候，就需要产生连接，可以从一个目标组件跳转到另外一个目标组件，这一个Navigation的核心。首先我们在创建一个Navigation目标组件，依然以Fragment为例。创建后XML文件内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@id/blankFragment">
    <fragment
        android:id="@+id/blankFragment"
        android:name="com.olair.architecture.BlankFragment"
        android:label="fragment_blank"
        tools:layout="@layout/fragment_blank" />
    <fragment
        android:id="@+id/blankFragment2"
        android:name="com.olair.architecture.BlankFragment2"
        android:label="fragment_blank_fragment2"
        tools:layout="@layout/fragment_blank_fragment2" />
</navigation>
```

其中一个BlankFragment一个BlankFragment2。现在我们希望可以从BlackFragment跳转到BlankFragment2。

1. 打开Navigation的视图窗口，将鼠标悬停在源组件BlackFragment上，将会看到在源组件右边缘出现一个小圆点。
2. 拖动在源组件右边缘出现的圆点，将其拖动到目标组件之上。现在可以看到一条连接在源组件和目标组件之间的连线，该连线用于表达两个组件之间的导航关系。
3. 点击这条线，属性面板中的内容会有所变化，如下：
   * Type Action 建立连接既是创建一个Action标签，Action标签用于表示一个导航时事件
   * ID 该Action的ID
   * Destination 目标组件