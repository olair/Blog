# View

View里面有三个对象：

* CheckForLongPress
* CheckForTap
* PerformClick

分别用来表示长按事件(500ms)、轻触事件(100ms)、点击事件。

View中的setPressed(true, x, y); 只是设置View以及其子View的状态为Pressed,没有调用什么方法。

1. View的构造方法中有通过TypedArray.getDimensionPixelSize其中可以发现，转换时采用四舍五入的方式。