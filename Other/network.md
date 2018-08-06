# Performing network operations

Fragment 不用来展示界面，而仅仅是参与activity的生命周期回调(类似一个Presenter)，但是需要注意的是，在activity生命周期中Fragment 有可能会多次创建，所以需要在Activity的onCreate()中设置setRetainInstance(true),并且Fragment getInstance()中要先通过TAG去找Fragment 找到的话就不用再new了。