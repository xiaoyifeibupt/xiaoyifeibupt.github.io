---
layout: post
title: malloc()与calloc区别
categories: [c++]
---

###版本一

####分配内存空间函数malloc

-   调用形式 ：`(类型说明符*) malloc (size)`
-   功能：在内存的动态存储区中分配一块长度为"size" 字节的连续区域。函数的返回值为该区域的首地址。 “类型说明符”表示把该区域用于何种数据类型。(类型说明符*)表示把返回值强制转换为该类型指针。“size”是一个无符号数。
-   例如： `pc=(char *) malloc (100);` 表示分配100个字节的内存空间，并强制转换为字符数组类型， 函数的返回值为指向该字符数组的指针， 把该指针赋予指针变量pc。

####分配内存空间函数 calloc

　　calloc 也用于分配内存空间。
　　
-   调用形式： `(类型说明符*)calloc(n,size) `
-   功能：在内存动态存储区中分配n块长度为“size”字节的连续区域。函数的返回值为该区域的首地址。(类型说明符*)用于强制类型转换。calloc函数与malloc 函数的区别仅在于一次可以分配n块区域。
-   例如： `ps=(struet stu*) calloc(2,sizeof (struct stu));` 其中的`sizeof(struct stu)`是求stu的结构长度。因此该语句的意思是，按stu的长度分配2块连续区域，强制转换为`stu`类型，并把其首地址赋予指针变量`ps`。

 

简单的说是：

malloc它允许从空间内存池中分配内存，`malloc()`的参数是一个指定所需字节数的整数，例如:

```cpp
P=(int*)malloc(n*sizeof(int));
```

colloc与malloc类似，但是主要的区别是存储在已分配的内存空间中的值默认为0，使用`malloc`时，已分配的内存中可以是任意的值。

colloc需要两个参数，第一个是需要分配内存的变量的个数，第二个是每个变量的大小。例如:

```cpp
P=(int*)colloc(n,colloc(int));
```
 

###版本二

函数原型不同:

```cpp
void *malloc(unsigned size)
```

动态申请size个字节的内存空间。功能：在内存的动态存储区中分配一块长度为" size" 字节的连续区域，函数的返回值为该区域的首地址。(类型说明符*)表示把返回值强制转换为该类型指针。

```cpp
(void *)calloc(unsigned n,unsigned size)
```

用于向系统动态申请n个, 每个占size个字节的内存空间，并把分配的内存全都初始化为零值。函数的返回值为该区域的首地址

```cpp
(void *)realloc(void *p,unsigned size)
```

将指针p所指向的已分配内存区的大小改为size

**区别** ：两者都是动态分配内存。主要的不同是`malloc`不初始化分配的内存，已分配的内存中可以是任意的值。`calloc`初始化已分配的内存为0。次要的不同是`calloc`返回的是一个数组，而`malloc`返回的是一个对象。

malloc它允许从空间内存池中分配内存，`malloc()`的参数是一个指定所需字节数的整数。例如:

```cpp
P=(int*)malloc(n*sizeof(int));
```

colloc与malloc类似，colloc需要两个参数，第一个是需要分配内存的变量的个数，第二个是每个变量的大小。例如:

```cpp
P=(int*)colloc(n,sizeof(int)); 
```

例，申请一个字符大小的指针

```cpp
char *p=(char *)malloc(sizeof(char)); //当然单个是没有什么意义的申请动态数组或一个结构,如

char *str=(char *)malloc(sizeof(char)*100); //相当于静态字符数组str[100],但大小可以更改的

typedef struct pointer
{ int data; 
struct pointer *p; 
} pp; 

pp *p=(pp *)malloc(sizeof(struct pointer)); //动态申请结构体空间
```

其他几个函数是队申请空间的修改的操作根据定义自己可以试试


###版本三

一：它们都是动态分配内存，先看看它们的原型：

```cpp
void *malloc( size_t size ); //分配的大小

void *calloc( size_t numElements, size_t sizeOfElement ); // 分配元素的个数和每个元素的大小
```

-   共同点是：它们返回的是 void * 类型，也就是说如果我们要为int或者其他类型的数据分配空间必须显式强制转换；

-   不同点是：用malloc分配存储空间时，必须由我们计算需要的字节数。如果想要分配5个int型的空间，那就是说需要5*sizeof(int)的内存空间：

```cpp
int * ip_a;
ip_a = (int*)malloc( sizeof (int) * 5 );
```

而用calloc就不需要这么计算了，直接：

```cpp
ip_a = ( int* )calloc( 5, sizeof(int) );
```
这样，就分配了相应的空间，而他们之间最大的区别就是：用malloc只分配空间不初始化，也就是依然保留着这段内存里的数据，而calloc则进行了初始化，calloc分配的空间全部初始化为0，这样就避免了可能的一些数据错误。

先写段代码体验体验....

```cpp
#include <iostream>
using namespace std;

void main() {
    int * ip_a;
    int * ip_b;
    
    ip_a = (int*)malloc( sizeof (int) * 5 );
    for( int i = 0; i < 5; i++ ) {
       cin>>ip_a[i];
    }
    for( int j = 0; j < 5; j++ ) {
       cout<<ip_a[j]<<" ";
    }


    ip_b = ( int* )calloc( 5, sizeof(int) );
    for( int j = 0; j < 5; j++ ) {
        cout<<ip_b[j]<<" ";
    }

}

```

 

 

###终极版：

三个函数的申明分别是:

```cpp
void* realloc(void* ptr, unsigned newsize);
void* malloc(unsigned size);
void* calloc(size_t numElements, size_t sizeOfElement);
```

都在`stdlib.h`函数库内，它们的返回值都是请求系统分配的地址，如果请求失败就返回NULL

malloc用于申请一段新的地址，参数size为需要内存空间的长度，如:

```cpp
char* p;
p=(char*)malloc(20);
```

calloc与malloc相似,参数sizeOfElement为申请地址的单位元素长度,numElements为元素个数,如:

```cpp
char* p;
p=(char*)calloc(20,sizeof(char));
```

这个例子与上一个效果相同

realloc是给一个已经分配了地址的指针重新分配空间，参数ptr为原有的空间地址，newsize是重新申请的地址长度
如:

```cpp
char* p;
p=(char*)malloc(sizeof(char)*20);
p=(char*)realloc(p,sizeof(char)*40);
```

注意，这里的空间长度都是以字节为单位。

C语言的标准内存分配函数：malloc，calloc，realloc，free等。

malloc与calloc的区别为1块与n块的**区别**：

malloc调用形式为`(类型*)malloc(size)`：在内存的动态存储区中分配一块长度为`size`字节的连续区域，返回该区域的首地址。

calloc调用形式为`(类型*)calloc(n，size)`：在内存的动态存储区中分配n块长度为`size`字节的连续区域，返回首地址。

realloc调用形式为`(类型*)realloc(*ptr，size)`：将`ptr`内存大小增大到`size`。

free的调用形式为`free(void*ptr)`：释放ptr所指向的一块内存空间。

C++中为`new/delete`函数。
