# [高并发网络编程之epoll详解](https://blog.csdn.net/shenya1314/article/details/73691088)

而所有添加到epoll中的事件都会与设备(网卡)驱动程序建立回调关系，也就是说，当相应的事件发生时会调用这个回调方法。这个回调方法在内核中叫ep_poll_callback,它会将发生的事件添加到rdlist双链表中。

Linux的方便之处是，大家遇到性能瓶颈时，可以快速的在现有基础上快速解决。

strcpy用于copy字符串 strlen和sizeof的区别

mmecpy用于内存copy