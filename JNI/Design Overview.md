# JNI 设计概述 (第二章)
* 原文连接 [Design Overview](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/design.html)

## 翻译约定 
1. 源文档中多次提到JNI functions,我们将其理解我JNI方法或者JNI功能又或者JNI接口函数，注意区分这几个名字和Native方法，不要搞混(已经混掉了可以看评论)。
2. Native方法定义指在Java文件中定义的 `native int compute();` 这种方法。
3. 本机方法指JVM所运行的系统支持的原生代码(比如Linux原生支持C/C++，Win支持C#/C/C++等，当然还支持汇编)。
4. Native方法实现指动态库中Native方法的实现。
5. 翻译完发现JNI文档也有好几个版本，虽然大体内容差不多，但是最新版本的比之前版本还是有所扩充的，不如下面的VM链接静态库的地方(上方链接已链接到最新版本)。

<!-- TOC -->

- [JNI 设计概述 (第二章)](#jni-设计概述-第二章)
    - [翻译约定](#翻译约定)
    - [第二章概要](#第二章概要)
    - [JNI接口函数 和 JNI接口指针](#jni接口函数-和-jni接口指针)
    - [编译，加载和链接Native方法](#编译加载和链接native方法)
        - [关于本地方法名字的定义格式](#关于本地方法名字的定义格式)
        - [Native方法参数](#native方法参数)
    - [Native方法中引用Java对象](#native方法中引用java对象)
        - [本地引用和全局引用](#本地引用和全局引用)
        - [本地引用的实现方式](#本地引用的实现方式)
    - [访问Java对象](#访问java对象)
        - [访问基本数据类型数组](#访问基本数据类型数组)
        - [访问Java中的字段和方法](#访问java中的字段和方法)
    - [报告编程错误](#报告编程错误)
        - [JNI开发中的异常](#jni开发中的异常)

<!-- /TOC -->

## 第二章概要
本章主要聚焦JNI的设计原则。本章中的大部分设计原则都和Native 方法有关。关于 Invocation API将在第五章进行介绍(Invacation API是在本地代码中对JVM进行利用的一些编程接口，比如创建JVM、attach 原生线程到JVM等)。

## JNI接口函数 和 JNI接口指针
Native代码通过调用JNI接口函数和JVM进行通信(使用JVM平台中的一些特性，比如使用其中的很多类库、功能方法等)。JNI接口函数可以通过JNI接口指针获得。JNI接口指针是一个二级指针。这个二级指针指向的是一个指针列表，指针列表中的每一个指针都指向一个JNI接口函数。每一个JNI接口函数的入口都通过该指针列表给出。图2-1 展示了JNI接口指针的组织形式。
![图2-1 JNI接口指针](https://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/images/designa.gif)

JNI接口采用类似C++中虚函数表或者COM接口的组织形式(之所以说是像，因为定义JNI接口是通过struct定义的，即C中的结构体，并不是纯粹的C++虚函数)。只想目标机器(使用JNI接口函数的机器)提供接口函数表而不是直接提供硬编码的代码去和JVM通信，是为了更好的将JNI接口函数功能从本机代码中分离，而且VM可以轻松的提供多个版本的JNI功能列表。例如，VM可能支持如下两个JNI函数表：

> 总结起来就是说，这样只定义了JNI接口出来，接口的实现交于各个版本的JVM去实现，随便JVM实现多少个版本怎么实现，在目标机器上(调用JNI接口函数的机器)并不关心，我只要能使用JNI接口中的方法就可以。怎么使用呢？需要通过一种方式让JVM提供出来我们上面提到的JNI接口指针，通过指针去访问。JVM怎么将JNI接口指针提供出来呢？有两种方式，一种是从Java调用本地方法时会有JNI接口指针传入本地方法中，另一种是本地代码需要用到JVM的特性，这个时候需要从主动创建或者其他方式得到的JVM中获取到JNI接口指针。

* 实现一个严格的参数检查的JNI接口版本，主要用于从测试。
* 实现一个JNI规范所需的尽量少的检查，可以提高运行效率。
一个JNI接口指针只会在当前线程有效。所以，一个native方法不可以将JNI接口指针从一个线程传递到另一个线程。实现JNI编程接口的VM很可能会在JNI接口指针指向的区域中去分配以及存储当前线程数据(包括Java、Native数据)。

Native方法中肯定会JNI接口指针这个参数。VM会保证在同一个Java线程中多次调用Native方法时传入的JNI接口指针都是同一个。但是同一个Native方法可以被多个Java线程调用，所以可能会收到不同的JNI接口指针。

## 编译，加载和链接Native方法
因为Java VM是支持多线程的，所以Native方法的编译器编译的时候也需要添加多线程支持。例如，使用[Sun Studio](https://baike.baidu.com/item/Sun%20Studio/8463165?fr=aladdin)编译器编译C++代码的时候需要加上 -mt 标志(编译时候的配置参数，类似于java -g -d C:/Test.java 这样)。对于使用GNU gcc编译器的代码，需要加上这个配置参数  -D_REENTRANT or -D_POSIX_C_SOURCE。关于本机原生代码编译的更多信息请参考本机原生代码的编译器文档。

Native方法需要通过Java中的System.loadLibrary()方法加在进系统(Native方法将会被编译成库文件，例如Linux的本机代码C++，用C++实现好的Native方法将会被编译成搜库通过loadLibrary加载)。在下面的例子中，在Java类中的静态代码库中加载了Java虚拟机所处的平台上的库文件(比如在Win上加载的将是dll文件，在Linux加载的将是搜文件)，在这个库文件中实现了Native方法 f(int i, Stirng s):


    package pkg;

    class Cls { 
        native double f(int i, String s); 

        static { 
            System.loadLibrary(“pkg_Cls”); 
        } 
    }


System.loadLibrary方法中的参数是要加载的库文件名字，这个库文件名字程序员可以自由定义。JVM系统会自动的遵循相应的平台标准将这个作为参数的库名字转为本地库名字。例如，Solaris系统中会将 pkg_Cls 转换为pkg_Cls.so,如果是在Win32系统上将会转换为pkg_Cls.dll。

> 总结下，loadLibrary中的参数填写的时候并不是程序员可以随便填写，而是需要根据你所编译出来的本机代码库的名字进行填写，但是本机代码库的名字程序员是可以随便命名的(当然也受到系统平台的限制，一般文件后缀会有要求，还要些不能包含*、&、￥等符号)，比如编译的Linux平台的动态代码库 media.so ，通过loadLibrary方法进行加载的时候参数必须这样 `System.loadLibrary(media)`，也就是说，最为loadLibrary的参数传入的时候只需要把后缀去掉就可以，VM会根据它所处的平台自动去添加后缀 (有动态库和静态库之分，动态库后缀 .so 静态库后缀 .a ，JVM只能加载动态库。Win中也有静态库动态库 分别是 .lib .dll)(还有一点需要提到就是还有一个System.load方法，用于在绝对路径加载动态库)。

程序员可以用一个单个的动态库文件来实现在任意数量的Class定义的Native方法。但是有一个前提条件，要想这些Native方法可以使用，就必须使用同一个类加载器(Class Loader)加载这些Class。VM内部维护者每个类加载器加载的动态库列表。动态库提供商应该对动态库设计一个良好的命名方式用以尽量减少命名冲突的可能性。

> 跟着`System.loadLibrary();`方法进去看一看会发现里面`Runtime.getRuntime().loadLibrary0(VMStack.getCallingClassLoader(), libname);`执行了这样一句。会将执行loadLibrary方法的Class所属的ClassLoader作为参数传进去。之所以将动态库跟类加载器绑定在一起和之所以会有类加载器这个东西道理是一样的，就是为了避免出现冲突，大家各用个的库版本。

如果底层操作系统不支持动态链接(不支动态库)，所有Native方法实现必须预先硬链接到VM，在这种情况下，VM可以完成loadLibrary调用但是没有什么实际意义。程序员还可以通过调用JNI接口函数中的RegisterNatives()方法将Native方法与本机代码关联起来(会有专门的章节介绍)。

> 前文中我提到JNI是只支持动态链接的，这么快就被打脸，其实JNI也可以满足一些系统在不支持动态库的情况下又需要使用JNI接口函数功能的需求。但是这种需求过于小众，而且实现也有点复杂，根据静态库的原理来看甚至需要重新对VM进行编译，一般没有这种需求，所以不进行讨论，具体的可以参看最新官方文档，内容不是很长。

### 关于本地方法名字的定义格式
动态链接过程中，动态连接器将根据Native名字以及本机方法名字在这两种方法之间产生关联信息。Native方法的实现方法名字是通过一下几个部分组合而成：

* 前缀Java_。
* Native方法所处Java类的完全限定类名(以下划线作为分隔符)。
* 下划线分隔符。
* 一个全限定方法名。
* 对于重载的本机方法，两个下划线后面跟完全的参数签名(参数签名后面会有讲解)。

VM将会负责检查在动态库中的方法的名字，跟Native方法进行匹配。VM会首先去匹配短一点的名字，也就是说Native方法中参数较少的那一个。然后才回去匹配带有参数的长名称。程序员仅仅在覆盖了Native方法的定义的时候才需要使用长名称(也就是加了参数的名称)，但是如果该Native方法没有被另一个Native方法覆盖的话就不需要使用长名称。如果Native方法和Java方法具有一样的名字依然不用使用长名称(因为在动态库中只有一个版本的Native方法实现)。
> 大家用IDE直接生产Native对应方法时需要注意以上问题。

在下面示例中，Native方法g可以不适用长名称进行链接，因为两位一个名称也为个g的方法并不是Native方法，并不在动态库中，不会应该本地代码库中Native方法的实现和Java中Native方法的定义的一一对应。

    class Cls1 { 

        int g(int i); 

        native int g(double d); 

    } 

我们采用一种简单的命名修改方案，以确保所有的Unicode字符都可以转为有效的C函数名称(应该不仅仅限于C)。我们使用下划线字符("_")代替完全限定名中的反斜杠("/")。由于在Java中名称和类型的描述不能以数字打头，所以我们可以使用_0，....，_9作为转义字符，像表2-1这样：

```
表2-1 Unicode字符转意
转义字符           意义
_0XXXX             一个Unicode字符XXXX，注意要使用小写而不是使用大小标示Unicode的字符编码
_1                  字符"_"
_2                  字符";"，一般用于类型签名中
_3                  字符";"，也是用于类型签名
```
> Both the native methods and the interface APIs follow the standard library-calling convention on a given platform. For example, UNIX systems use the C calling convention, while Win32 systems use __stdcall.(没有实质性价值的一句话。)

### Native方法参数
JNI接口指针是Native方法(这里指Native方法在动态库中的实现)的第一个参数。JNI接口指针是一个JNIEnv类型指针。第二个参数跟这个Native方法是否是静态方法有关，当Native方法时一个static方法的时候第二个参数是这个Java class的引用，当这个Native方法不是static方法时指向的是该Native方法所属的对象。
其余的参数一一对应于Java中Native方法定义时的参数。Native方法通过返回值将其结果传递给调用者。第三章中将会讲到JNI中的数据类型以及数据结构，通过这些数据类型用来帮助程序开发者在Java和C类型之间相互映射。

以下代码示例说明了如何使用C函数实现Native方法 `f(int i,String s);`。Native方法声明如下：


    package pkg; 

    class Cls {
        native double f(int i, String s);
        // ...
    }

使用采用长名称命名的C函数实现本机方法：

    jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (
        JNIEnv *env,        /* interface pointer */
        jobject obj,        /* "this" pointer */
        jint i,             /* argument #1 */
        jstring s)          /* argument #2 */
    {
        /* Obtain a C-copy of the Java string */
        const char *str = (*env)->GetStringUTFChars(env, s, 0);

        /* process the string */
        ...

        /* Now we are done with str */
        (*env)->ReleaseStringUTFChars(env, s, str);

        return ...
    }

注意：如上所示我们必须也只能使用JNI接口指针env操作Java对象。
使用C++，你可以写出稍微简洁一些的代码版本，如下代码所示：

    extern "C" /* specify the C calling convention */ 

    jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (
        JNIEnv *env,        /* interface pointer */
        jobject obj,        /* "this" pointer */
        jint i,             /* argument #1 */
        jstring s)          /* argument #2 */
    {
        const char *str = env->GetStringUTFChars(s, 0);

        // ...

        env->ReleaseStringUTFChars(s, str);

        // return ...
    }

采用C++编写后，你会发现*号以及额外的参数都不需要了。但是，其实本质上和C还是一样的只是在C++中JNI接口函数被定义成内联成员函数，编译时它们将被扩展成C对应的函数。

## Native方法中引用Java对象
基本类型数据，像int,char等等这些数据，会在Java和Native实现代码之间直接进行复制，而Java对象是通过引用传递。VM必须跟踪传递到Native方法实现中的所有Java对象，以防止该对象会被垃圾收集器释放掉。反过来，Native方法中也必须有一种机制用来通知JVM它不在需要这些对象。此外，垃圾收集器也必须能够移动本机代码所引用的对象(不是很清楚，应该是在说需要可以将Java对象从一个区域移动到另一个区域)。

### 本地引用和全局引用
JNI将Native代码中对Java对象的引用分为两类：本地引用和全局引用。本地引用只在该Native方法调用期间有用，本机方法返回的时候将会自动解除掉该引用(一旦执行了return对象引用就将失效，其实说是失效仅仅是说有可能会被JVM垃圾收集器回收，但是也不一定会被回收因为在Java对象中可能会有其引用)。而全局引用将一直有效直到其被显式的释放。
Java对象以本地应用的形式传递给Native代码。所有的JNI接口函数返回的Java对象都是本地引用。JNI允许开发者将本地引用转化为一个全局引用。JNI接口函数可以接受本地或全局引用作为参数，而一个Native方法也可以将本地或者全局引用返回给VM。

大部分时候，程序开发者依赖VM在Native方法返回后自动释放掉本地引用即可，但是，有时候开发者也需要主动的去释放本地引用。考虑下面这些情况：
* Native代码需要操作一个大型的Java对象，于是创建了对该Java对象的本地引用。此时，Native代码在该调用方法结束之前需要执行很多其他操作，这样，即使该大型Java对象在剩余的操作中已经不再被使用，可是因为此时本地引用的存在，该大型Java对象依然不会被垃圾收集器回收。
* Native代码中需要创建大量的本地引用，而VM是需要一定的空间去跟踪这些本地引用，所以创建太多的本地引用很可能会导致内存不足，而且这些引用可能并不需要同时使用，这个时候就有主动去释放的必要。例如，需要在Native代码中去循环遍历一个数组，将数组中每一个元素作为本地引用在每一次迭代中去操作。在每次迭代之后，开发者将不再需要上一次迭代所使用元素的本地引用，这个时候就需要主动去删除本地引用。
  
JNI允许开发者在Native代码中的任何地方去主动的删除本地引用。 To ensure that programmers can manually free local references, JNI functions are not allowed to create extra local references, except for references they return as the result.(没有很明白这句，可以肯定的是JNI functions是可以创建本地引用的，并且 **所有的JNI接口函数返回的Java对象都是本地引用**)。

本地引用只在创建他们的线程中有效，所以Native代码中不能将本地引用从一个线程传递到另一个线程。

### 本地引用的实现方式
为了实现本地引用(根据上面说的是否明白了什么是本地引用)，每次从Java代码转换到Native代码中时，JVM都将创建一个注册表。这个注册表将本地引用映射到相应的Java对象，并防止Java对象被垃圾收集器回收。所有传递给Native方法的Java对象还有所有JNI接口函数返回的对象都会自动添加到注册表。当Native方法执行结束之后，注册表将会被删除，注册表也不再会影响到JVM的垃圾收集器释放对象(当然只是注册表的影响没有了，其他地方依然引用了该对象就依然不能被回收)。

该注册表的实现多种多言，例如使用表、链表或者哈希表。需要注意的是注册表中允许存在重复条目，即注册表中可能有两个引用指向同一个Java对象，JNI并没有义务去检测和合并重复的引用。哪怕通过引用计数去避免在注册表出现重复条目是如此简单，但JNI没有义务一定要这么做。

需要注意的是，仅仅通过本地方法栈没有办法很好的实现本地引用。所以JNI实现本地引用的时候应该尽量将本地引用存储到方法区或者堆中(Note that local references cannot be faithfully implemented by conservatively scanning the native stack. The native code may store local references into global or heap data structures.)。

## 访问Java对象
JNI接口函数提供了基于全局引用以及本地引用的丰富的访问函数。这意味着Native代码不需要关心VM内部是怎么表示Java对象的，在任何VM平台上Native方法都可以顺利编译执行。

通过不透明的方式使用JNI接口函数去操作Java对象的确会比直接去操作C语言结构数据要多一些性能损耗。但是，我们相信，在大多数情况下，Java开发者是为了通过Native方法去执行很重要的任务代码，而这个重要性将忽略掉这点性能开销。

### 访问基本数据类型数组
对于包含许多基本数据类型的大型Java对象而言(一般数组具有这个特征)，因为JNI接口产生的性能损耗往往是不能接受的(比如执行向量或者矩阵运算的Native方法)。使用JNI接口函数去迭代每一个元素将是非常低效的。

下面一种解决方案引入了"pinning(固定)"的概念，Native代码可以要求VM固定住数组内容。Native代码直接访问数组在VM中的直接指针。但是这种方法有两个要求：

* 垃圾收集器必须支持固定。
* VM必须在内存中连续布局原生数据类型数组。虽然这是大多数原始数组最自然的实现，但是boolean类型数组却不一般。Although this is the most natural implementation for most primitive arrays, boolean arrays can be implemented as packed or unpacked.Therefore, native code that relies on the exact layout of boolean arrays will not be portable.(可以不理解。。。。)

为了更好的实现，我们采用了一种妥协的方案达到上面的要求。
首先，我们提供了一组函数可以复制Java原始数据类型数组中的一部分到宿主机内存中。当Native代码中只需要访问大型数组中的少量元素时可以使用。
第二，开发者可以使用另一组函数来检索数组对象的固定版本(pinning将数组固定之后再访问)。但是请注意，这些功能可能依然需要JVM执行存储空间分配和复制操作，究竟是否复制取决于虚拟机自己的实现，具体情况如下：

* 如果垃圾收集器主持固定(pinning)，而且Java中的数据布局方式跟宿主机中使用的布局方式相同，那就不需要复制。
* 负责，将数组复制到不可以移动的内存块(例如C堆中)，并且复制的时候需要根据宿主机以及JVM具体情况执行必要的格式转换。JNI接口函数将返回对应的指针。


最后，JNI接口函数中提供方法用于通知VM，Native方法中不再需要访问数组元素。通过调用这些方法，系统会取消该数组的固定(pinning)，或者将原始数组与其不可移动的副本进行协调并释放副本(协调一般指将副本更新到Java数据中去)。
这些方法将给VM很大的灵活性。垃圾收集器算法可以针对每个给定的数组对象做单独的不同的处理。比如，对于较小的对象可以进行复制，对于较大的对象可以采用固定。

JNI接口函数的实现，必须确保在多个线程中运行的Native代码可以同时访问一个数组。例如，JNI可以为每一个固定数组保留一个内部计数器，这样可以保证多个线程固定的时候不会出现提前解除固定的情况。但是请注意，JNI不需要去主动的给原始数组加锁去避免Native方法的独占访问。所以，开发者必须去保证数据同步。

### 访问Java中的字段和方法
JNI允许Native代码访问Java对象中的字段以及方法，JNI通过它们的名称以及类型签名来识别方法以及字段。往往只需要两步就可以从字段或者方法签名获取到该字段值或者使该方法执行。例如，为了执行cls类中的f(int i,String s);方法，采用如下步骤

    jmethodID mid = env->GetMethodID(cls, “f”, “(ILjava/lang/String;)D”);
    jdouble result = env->CallDoubleMethod(obj, mid, 10, str);
    
上面的jmethodID 可以重复使用，而无需每次去Get一次(看jmethodID定义也可以看出来)。

需要注意的是，字段或者方法ID并不会阻止VM卸载其所属的class对象。卸载对应class对象后，这些字段ID、方法ID将变得无效。因此如果打算长时间使用该ID，Native方法必须保证直接或间接持有该class的引用，或者每次都重新去获取字段或者方法ID。
并且JNI不会对VM内部如何实现字段和方法ID施加任何限制。

## 报告编程错误
JNI接口函数不负责检查编程错误，包括NULL指针以及非法参数类型错误。非法参数类型错误包括使用普通Java对象代替Java Class对象(应该也包含反过来)等。之所以JNI接口函数不负责检查这些编程错误是基于以下原因：
* 强制JNI接口函数检查所有的可能错误的条件将会降低Native代码的性能。
* 在许多时候并没有足够的运行时类型信息供JNI接口函数去执行此类检查。
大多数C库函数都不能有效的防止编程错误。例如，printf()函数接收到无效地址时通常会导致程序异常崩溃而不是返回错误码。而且强制C库函数检查所有的错误条件，很可能会导致重复的检查(一次在用户代码中，一次在库中)。

程序员不得将非法指针或者错误类型参数传递给JNI接口函数，因为这样做很可能会导致难以预料的结果，包括系统状态损坏或者VM崩溃。

### JNI开发中的异常
异常在JNI开发中用起来并不困难，但是要明白文档中所说的还是需要点内容的，后面会专门针对JNI异常讲解(该段暂时没有翻译)。
