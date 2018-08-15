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

今天我们体验下Jetpack中的Navigation库，Navigation库是官方为单Activity开发提供的官方支持，并且官方专门为Navigation开发了Navigation的图形化界面、专门定制了一套安全的数据传输方案，但是因为整个Navigation库还没有正式发布，所以这个安全的数据传输方案目前还有点问题。

> 需要注意的是，因为Navigation功能包含在Android Studio 3.2版本中，而Android Studio 3.2版本目前还处于Beta阶段，根据我亲身体验来说，Navigation图形化界面以及数据传输方案