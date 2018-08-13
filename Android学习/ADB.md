# ADB基本命令

设备的Backkey键
adb shell input keyevent 4

解锁屏幕
adb shell input keyevent 82

滑动
adb shell input swipe 50 250 250 250 500

点击
adb shell input tap 50 250

输入
adb shell input text abc

adb -s deviceCode shell input text adb