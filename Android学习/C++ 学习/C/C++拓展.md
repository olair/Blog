# C/C++要点

C/C++ 标准中规定 int长度必须不小于short，long必须不小于int。所以尽量采用 int_32，int_64这种定义(C++ 中提议使用具体类型来表达意思，而不是模糊的类型，比如 size_t)。

float、double、long double

bool 在C++中才有，C99中已经有了布尔类型，在stdbool.h头文件中，bool类型就是一个int，非0既是true

sprintf(char *,"") 将后面字符串中的值复制到char* 中

malloc 申请的内存并不会初始化，可以使用memset初始化。也可以直接使用calloc。realloc重新调整malloc申请的大小。还有alloca向栈中申请内存。

sizeof 可以计算数组大小

[Linux下system()函数的深度理解(整理)](https://www.cnblogs.com/tdyizhen1314/p/4902560.html)

## 指针

char temp[] = "hello";
const char *p2 = temp; = char const *p3 = temp;(p2/p3中的char不能再修改，而const 是不能修改指针的)
char* const p4 = temp;(p4不可在被赋值)
const char* const p5 = temp;
chat const* const p6 = temp;(p5/p6都不能变)

### 可变参数

引入头文件 #include<stdarg.h>
va_list list;
va_start();// 标记可变参数开始位置
va_arg();// 根据参数类型获取参数
va_end();// 标记可变参数结束

### 函数指针

void(*p)(char*)声明函数指针 void返回值 \*p函数指针 char\*函数参数

void say(void (*p)(char*),char* msg)

使用typedef给函数声明创建一个别名

### 预处理器

| 预处理器 | 说明         |
| -------- | ------------ |
| #include | 导入头文件   |
| #if      | if           |
| #elif    | else if      |
| #else    | else         |
| #endif   | 结束 if      |
| #define  | 宏定义       |
| #ifdef   | 如果定义了宏 |
| #ifndef  | 如果未定义宏 |
| #undef   | 取消宏定义   |
| #pragma  | 设定编译器的状态 |

\#pragma once   该头文件只会被调用一次(但其实很多编译器不支持)

### 宏函数 内联函数(内联函数不能写复杂语句，不然会被自动降级为普通函数，inline)

## C结构体

### 对齐规则

### 预编译改变对齐规则

pragma pack()  __attribute__((aligned(2),packed))

## 共用体

## C和C++混用要注意的问题

### 查看符号表

nm -A main.o

注意 extern "C" 的使用

__cplusplus 编译器预定义的宏，用于判断是C还是C++

### C++引用类型 [Ｃ++ 引用（别名）](https://www.cnblogs.com/chuijingjing/p/9009293.html)

int i = 10;
int& j = i;

### 命名空间

可以嵌套定义。C++可以重载但是C不可以。操作同名的全局变量的时候可以用域作用符。

## C++ 类

```C++
#ifndef 文件名
#define 文件名

#enfdif
```

栈中定义的对象，从有效区域出去时会自动调用析构函数(猜测编译器有添加代码)

成员函数后面加const ,如 void setJ(int j) const;(常量函数，表示不会也不允许去修改类中的成员，一般用于get类方法)

### 友元函数 友元类

friend void test();可以访问类中的私有成员

### 单例

C++11 中对于静态变量，编译器会自动加上多线程同步的处理。class中的私有静态变量可以在cpp文件中赋初值?

### 重载 函数重载、操作符重载(成员运算符重载、非成员运算符重载)

### C++ 中的多态 静态多态、动态多态(虚函数)

析构函数都定义为虚函数，构造方法不允许被声明为虚函数

### 纯虚函数

不实现，类似于Java抽象方法。

## Cmake

D:\AndroidSDK\cmake\3.6.4111459\android.toolchain.cmake中定义了一些全局变量