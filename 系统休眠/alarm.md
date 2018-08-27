# AlarmManager 详解

## 方法列表

方法还是蛮多的

* cancel(PendingIntent operation)
* cancel(AlarmManager.OnAlarmListener listener)
* getNextAlarmClock()
* set(int type, long triggerAtMillis, PendingIntent operation)
* set(int type, long triggerAtMillis, String tag, AlarmManager.OnAlarmListener listener, Handler targetHandler)
* setAlarmClock(AlarmManager.AlarmClockInfo info, PendingIntent operation)
* setAndAllowWhileIdle(int type, long triggerAtMillis, PendingIntent operation)
* setExact(int type, long triggerAtMillis, PendingIntent operation)
* setExact(int type, long triggerAtMillis, String tag, AlarmManager.OnAlarmListener listener, Handler targetHandler)
* setExactAndAllowWhileIdle(int type, long triggerAtMillis, PendingIntent operation)
* setInexactRepeating(int type, long triggerAtMillis, long intervalMillis, PendingIntent operation)
* setRepeating(int type, long triggerAtMillis, long intervalMillis, PendingIntent operation)
* setTime(long millis)
* setTimeZone(String timeZone)
* setWindow(int type, long windowStartMillis, long windowLengthMillis, PendingIntent operation)
* setWindow(int type, long windowStartMillis, long windowLengthMillis, String tag, AlarmManager.OnAlarmListener listener, Handler targetHandler)

其中有PendingIntent以及OnAlarmListener 两种回调方式，一种是通过广播，一种是通过listener，看OnAlarmManager文档要求必须处于运行状态。
Exact 准时的准确的，set本身并不一定准确

对于setRepeating方法有一个地方需要注意，If an alarm is delayed (by system sleep, for example, for non _WAKEUP alarm types), a skipped repeat will be delivered as soon as possible. After that, future alarms will be delivered according to the original schedule; they do not drift over time. For example, if you have set a recurring alarm for the top of every hour but the phone was asleep from 7:45 until 8:45, an alarm will be sent as soon as the phone awakens, then the next alarm will be sent at 9:00.

android 6.0 API23 新增了Doze(low-power idle) 以及APP Standby模式

对于android 4.4(API 19),添加了AlarmManager的批处理,并且添加了setExact()方法，可以用来设置准时的Alarm，如果你 targetSdkVersion to "18" or lower 的话即使使用set或者setRepeating也是准时的(在4.4运行，行为将和4.4以前版本保持一致)

电量优化有三个步骤：
程序员角度

1. 需要遵循 Lazy First
2. 利用Android提供的平台特性
3. 使用工具找到电量消耗的元凶

Lazy First模式核心
Reduce Defer Coalesce

平台特性
提供了APIs 帮助实现你的需求 Job S车都灵
提供 Doze 以及 App Standby

工具
Profile GPU Rendering 和 Battery Historian

Doze 模式中 系统忽略wake locks AlarmManager的setExacty()以及setWindow()设置的警报都会被延期到下一个窗口期 但是setAlarmClock() 依然可以正确的触发(在系统触发setAlarmClock方法之前系统退出Doze) 虽然提供了AndAllowWhileIdle()相关方法，但是该方法已依然有限制，看optimizing for battery life 中说的不能超过九分钟一次，而方法注释中说的 1分钟一次/15分钟一次

APP Standby相当于一个细粒度的Doze(针对APP的，从这句话看出 App Standby allows the system to determine that an app is idle when the user is not actively using it. )

要想不进入APP Standby 状态
除非APP是被用户主动打开
APP处于前台(用户交互Activity 或者有一个前台服务，或者被上面两个使用)
APP生成了一个用户可见的内容(在锁屏或者notification tray中)
APP是一个设备管理员

应用可以被添加进入白名单，这样在Doze以及APP Standby模式下也可以访问网络或者持有 [Partial wake locks](https://developer.android.google.cn/reference/android/os/PowerManager#PARTIAL_WAKE_LOCK),

但是我没有测试加入白名单以后是否setAndAllowWhileIdle依然受到限制

监控电池状态 电池状态广播是一个粘性广播，不要去频繁获取电池状态
BatteryManager 电池基座有桌面类型还有汽车类型等。

android 5.0 提供JobScheduler 在 android 7.0中移除了三个常用广播 CONNECTIVITY_ACTION, ACTION_NEW_PICTURE, and ACTION_NEW_VIDEO(since those can wake the background processes of multiple apps at once and strain memory and battery)