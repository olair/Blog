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

\#pragma once   该头文件只会被调用一次

### 宏函数 内联函数(内联函数不能写复杂语句，不然会被自动降级为普通函数，inline)