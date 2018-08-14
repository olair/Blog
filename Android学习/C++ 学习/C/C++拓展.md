# C/C++要点

C/C++ 标准中规定 int长度必须不小于short，long必须不小于int。所以尽量采用 int_32，int_64这种定义(C++ 中提议使用具体类型来表达意思，而不是模糊的类型，比如 size_t)。

float、double、long double

bool 在C++中才有，C99中已经有了布尔类型，在stdbool.h头文件中，bool类型就是一个int，非0既是true

sprintf(char *,"") 将后面字符串中的值复制到char* 中

malloc 申请的内存并不会初始化，可以使用memset初始化。也可以直接使用calloc。realloc重新调整malloc申请的大小。还有alloca向栈中申请内存。

sizeof 可以计算数组大小