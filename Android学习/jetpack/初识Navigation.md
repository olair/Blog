# 初识Navigation(Google为 单Activity+多Fragment 开发方式提供的官方支持)

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

以后Android将不再提供v4/v7/v13/v14等包的更新，统一到Jetpack库中，并以androidx.作为新的包开头。

今天我们体验下Jetpack中的Navigation库，Navigation库是官方为单Activity开发提供的官方支持，并且官方专门为Navigation开发了Navigation的图形化界面、专门定制了一套安全的数据传输方案，但是因为整个Navigation库还没有正式发布，所以这个安全的数据传输方案目前还有点问题。