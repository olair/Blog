# JNI中的数据类型以及数据结构

<!-- TOC -->

- [JNI中的数据类型以及数据结构](#jni中的数据类型以及数据结构)
    - [一、概述](#一概述)
    - [二、Java基本类型](#二java基本类型)
    - [三、Java引用类型](#三java引用类型)
    - [四、Java字段和方法(Field/Method)](#四java字段和方法fieldmethod)
    - [五、JNI Value类型([联合体类型](https://www.cnblogs.com/fengty90/p/3768840.html))](#五jni-value类型联合体类型httpswwwcnblogscomfengty90p3768840html)
    - [六、JNI中的类型签名](#六jni中的类型签名)
    - [七、JNI中UTF-8字符串说明](#七jni中utf-8字符串说明)
    - [八、字符编码说明](#八字符编码说明)
    - [九、JNI为什么定义这些信息](#九jni为什么定义这些信息)
        - [1、基于方法调用](#1基于方法调用)
        - [2、相互保存对方变量以及持有对方引用](#2相互保存对方变量以及持有对方引用)

<!-- /TOC -->

## 一、概述

因为JNI是一套Native层和JVM通信的机制，通信就是信息交流，所以就必须要提供在两种语言间的数据交换的机制。基于这个原因，JNI中通过typedef定义了一套专用的数据类型以及数据结构。

## 二、Java基本类型

|Java Type|Native Type|Description|
|---------|-----------|-----------|
|boolean| jboolean|unsigned 8 bits|
|byte|jbyte|signed 8 bits|
|char|jchar|unsigned|
|short|jshort|signed 16 bits|
|int|jint|signed 32 bits|
|long|jlong|signed 64 bits|
|float|jfloat|32 bits|
|double|jdouble|64 bits|
|void|void|not applicable|

还添加了一下定义方便使用

* `#define JNI_FALSE 0`
* `#define JNI_TRUE 1`
* `typedef jint jsize` 用于标识索引和大小(C++标准建议用尽量准确的类型符来标识一个变量。比如，表示大小就用size_t等，而不是使用在个平台都有可能产生不一致的int、short等)

## 三、Java引用类型

JNI 中还包含了一组特殊的类型，用来对应Java中的引用类型，我们暂且称之为JNI引用类型。为了尽量和Java引用类型像照应，JNI引用类型遵照如下层次结构：

* jobject(用于引用所有Object对象)
  * jclass(用于引用Classs对象)
  * jstring(用于引用String对象)
  * jarray(数组对象)
    * jobjectArray(对象数组)
    * jbooleanArray(boolean数组，不是Boolean数组)
    * jbyteArray(byte数组)
    * jcharArray(char数组)
    * jshortArray(short数组)
    * jintArray(int数组)
    * jlongArray()
    * jfloatArray()
    * jdoubleArray()

JNI在定义这一组JNI引用类型的时候，正对C和C++分别采取了不同的定义方式。C中采用`typedef jobject jclass;`这种定义。C++因为是面向对象的，所以直接采用了类继承的方式。详见下图

![JNI引用类型](./img/jni-reference.png)

> 可以看到在C中，不管是什么类型本质上都是void* 类型，而C++则是空类并且有了继承关系。可本质上并没有什么区别，使用的时候具体是什么类型仍需要用户自己去区别。

## 四、Java字段和方法(Field/Method)

Java中的字段和方法定义为C中结构体的指针，如下：

```C++
struct _jfieldID;              /* opaque structure */
typedef struct _jfieldID *jfieldID;   /* field IDs */

struct _jmethodID;              /* opaque structure */
typedef struct _jmethodID *jmethodID; /* method IDs */
```

> jfieldID以及jmethodID后面注释了一句 "opaque structure(不透明结构)"，应该是说该结构交给JVM自己去定义。jni.h文件中还定义了JNINativeMethod类型，该类型暂时不用关注,它用于标识一个Java方法(一般在Native方法与Java方法的动态绑定时使用)。

## 五、JNI Value类型([联合体类型](https://www.cnblogs.com/fengty90/p/3768840.html))

```C++
typedef union jvalue {
    jboolean z;
    jbyte    b;
    jchar    c;
    jshort   s;
    jint     i;
    jlong    j;
    jfloat   f;
    jdouble  d;
    jobject  l;
} jvalue;
```

## 六、JNI中的类型签名

在JNI中个JVM通信时很多时候不使用具体类型(比如boolean、int、class等)而是使用类型签名，并且是直接使用JVM所使用的类型签名(应该是处于效率之类的考虑吧)。JVM中的类型签名如下：

|类型签名|具体Java类型|
|----|----|
|Z|boolean|
|B|byte|
|C|char|
|S|short|
|I|int|
|J|long|
|F|float|
|D|double|
|L 接上具体Class的全限定名 ;|具体的Class类型|
|[ 接上类型签名|数组类型|
|(参数签名)方法返回值类型签名| 方法类型签名|

举使用类型签名的一个例子：

一个Java方法:
`long f(int n,String s,int[] arr);`
其类型签名如下：
(ILjava/lang/String;[I)J

> 1. 注意其中的信息：*I* 是 *int* 的类型签名，*Ljava/lang/String;* 是 *String* 的类型签名，*[I* 是 *int[]* 的类型签名，*J*是long的类型签名。** 其合在一起是*`long f(int n,String s,int[] arr);`*这个方法的类型签名。
>
> 2. Java中有方法签名来标识方法的唯一性，那么就有类型签名来标识类型的唯一性(类型签名就是给boolean、int、class等这些找了一个简称或者别称)。
>
> 3. 其中最后一个方法类型签名，在java.lang.invoke包中有一个MethodType类型，该类型属于Java方法签名的一部分，包含方法的参数信息以及返回值信息。我们可以理解为，方法包含类型信息以及方法名信息(类似参数包含参数名以及类型信息)。

## 七、JNI中UTF-8字符串说明

JNI中使用的字符编码方案是在UTF-8的基础上稍加修改而来的，具体的查看JNI文档(JVM中采用的也是修改过的UTF-8)。

其中涉及到两点不同：

1. 对空字符`(char)0`编码时不是像标准UTF-8那样使用一个字节，而是使用两个字节，这就避免了在字符串中嵌入了空值(应该是避免了无意间引入的空字符对JVM的运行产生影响)。
2. Java VM 无法识别UTF-8的四字节格式，它使用两倍或三倍三字节来代替UTF-8中超过三字节的表示。

## 八、字符编码说明

因为早前搞OpenVPN的时候仔细去熟悉了下Unicode才算明白Unicode以及UTF-8/UTF-16之间的渊源，简单的提一下。

Unicode（统一码、万国码、单一码），是计算机行业定义的一套字符标准。一般我们提到的时候会说Unicode字符集，讲的是一套字符集，而不是一套编码方案，并且它引入了平面的概念。不讲复杂码位平面什么的，简单点说Unicode字符集用四个字节来指向一个字符，这是字符标准，在你电脑的字体文件中就是通过Unicode的码位来一一照应上文字(不是通过UTF-8或者其他编码方案的值)。而UTF-8/UTF-16等都编码方案，将四个字节的标准Unicode码编码成UTF-8等所定义的编码格式，但是真正用的时候还需要解码回Unicode。之所以这样做，是因为Unicode每个字符都占用4个字节，但常用字符往往没那么多，重新实现编码方案将大大的缩小空间浪费。

## 九、JNI为什么定义这些信息

JNI是JVM和Native层沟通的桥梁，要达到JVM和Native可以相互沟通的目的，就必须实现下面两点

1. Java中可以调用Native层的方法(不可避免的需要传递参数)，持有Native层数据结构或者对象的引用，保存Native层的变量。
2. Native同样需要可以调用Java层的方法，持有Java层对象的引用，保存Java层变量。

### 1、基于方法调用

1. 在Java层提供了native方法属性，被该属性标记的方法可以和Native层方法产生关联用来代用Native层方法。关联方式有：
   * 静态绑定(通过命名方式绑定)
   * 动态绑定(通过手动写绑定方法绑定，后面会有)
2. 在Native层呢，就是通过上面的jclass、jmethod等以及方法类型签名来调用。

### 2、相互保存对方变量以及持有对方引用

1. Java层持有Native层对象一般很好解决，真毒C/C++来说，直接保存其内存地址或者内存数据即可。
2. Native层只有Java层对象只能通过jclass、jstring、jobjectArray等来持有对象的引用。数据的保存，比如byte[]等数组类型除了持有引用外还提供一些额外的操作，像copy等，后面会有介绍。
