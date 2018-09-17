# View参数

* mTransientStateCount View有一个名为暂态的状态(即动画执行过程中为暂态，动画开始需要执行setHasTransientState(true)，结束执行setHasTransientState(false))。ViewGroup中有一个childHasTransientStateChanged的实现，子View改变瞬时状态时会通过它通知ViewParent，

* 感觉mPrivateFlags 是View的一些状态，mPrivateFlags2是有关View中的内容状态

* 该段代码感觉是设置随键盘滚动的View的相关信息

```java
case R.styleable.View_isScrollContainer:
                    setScrollContainer = true;
                    if (a.getBoolean(attr, false)) {
                        setScrollContainer(true);
                    }
                    break;

```

* View中有一个DeclaredOnClickListener用于实现在xml中配置监听事件，但需要注意这里面有一个异常用于标示是不是Activity的context。
* View_transitionName Viw中的过渡动画有关吗？
* DUPLICATE_PARENT_STATE 属性，用于标示当前View状态(绘制相关状态)是否直接使用父容器的状态(onCreateDrawableState())。
* ?View 中有ENABLED_MASK相关属性，具体作用未知
* ？ThreadedRenderer 硬件加速相关，可以去具体看一看
* ？StateSet View中使用的一个
* onCreateDrawableState 和 mergeDrawableStates可以用来增加View状态

* ScrollabilityCache中包含了支持Scroll的很多相关字段，避免了每个View中都会出现这些字段。
* [StateListAnimator](https://www.cnblogs.com/android-blogs/p/5816965.html) 发现了一个动画选择器
* drawableStateChanged 中观察得来，导致View重新进行绘制的状态改变有四个大的方向
  1.mBackground 2.mForegroundInfo 3.mScrollCache 4.mStateListAnimator
* android 5.0以后增加了Tint [Android Tint使用](https://blog.csdn.net/zhuoxiuwu/article/details/50976337)
* android 5.0以后还新增了[阴影和剪裁](https://www.cnblogs.com/McCa/p/4465597.html)
* android 5.0 [Material Design Reveal effect(揭示效果) 你可能见过但是叫不出名字的小效果](https://www.cnblogs.com/dd-dd/p/6195985.html)

View里面有三个对象：

* CheckForLongPress
* CheckForTap
* PerformClick

分别用来表示长按事件(500ms)、轻触事件(100ms)、点击事件。

View中的setPressed(true, x, y); 只是设置View以及其子View的状态为Pressed,没有调用什么方法。

1. View的构造方法中有通过TypedArray.getDimensionPixelSize其中可以发现，转换时采用四舍五入的方式。
