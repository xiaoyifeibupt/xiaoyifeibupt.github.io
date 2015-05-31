---
layout: post
title:  用C实现C++中的多态性
categories: [C++]
---

###前言：关于多态，关于 C

多态 (polymorphism) 一词最初来源于希腊语 polumorphos，含义是具有多种形式或形态的情形。在程序设计领域，一个广泛认可的定义是**“一种将不同的特殊行为和单个泛化记号相关联的能力”**。 然而在人们的直观感觉中，多态的含义大约等同于**“同一个方法对于不同类型的输入参数均能做出正确的处理过程，并给出人们所期望获得的结果”**，也许这正体现 了人们对于多态性所能达到的效果所寄予的期望：使程序能够做到越来越智能化，越来越易于使用，越来越能够使设计者透过形形色色的表象看到代码所要触及到的 问题本质。

作为读者的你或许对于面向对象编程已有着精深的见解，或许对于多态的方便与神奇你也有了深入的认识。这时候你讶异的开始质疑了：“多态，那是面向对象编程才有的技术，C语言是面向过程的啊！”而我想说的是，**C 语言作为一种编程语言，也许并不是为了面向对象编程而设计，但这并不意味着它不能实现面向对象编程所能实现的功能**，就比如说，多态性。

在本文中我们使用一个简单的单链表作为例子，展示 C 语言是如何体现多态性的。

###结构体：不得不说的故事

许多从写 C 代码开始，逐渐走向 C++ 的程序员都知道，其实 C++ 里面的 class，其前身正是 C 语言中的 structure。很多基于C语言背景介绍 C++ 的书籍，在介绍到 class 这一章的时候都会向读者清晰地展示，一个 C 语言里的 structure 是怎样逐渐变成一个典型的 C++ class 的，甚至最后得出结论：“structure 就是一个所有成员都公有的类”。当然了，class 还是 class，不能简单的把它看做一个复杂化了的 structure 而已。

下面我们来看看在 C 语言中定义一个简单的存储整型数据的单链表节点是怎么做的，当然是用结构体。大部分人会像我一样，在 linkList.h 文件里定义：

```c
 typedef struct Node* linkList; 
 struct Node                                       // 链表节点
 { 
    int data;                                 // 存储的整型数据
    linkList next;                            // 指向下一个链表节点
 };
 ```

链表有了，下面就是你想要实现的一些链表的功能，当然是定义成函数。我们只举几个常用功能：

```c
 linkList initialLinklist();                         // 初始化链表
 link newLinkList (int data);                        // 建立新节点
 void insertFirst(linkList h,int data);              // 在已有链表的表头进行插入节点操作
 void linkListOutput(linkList h);                   // 输出链表中数据到控制台
```

这些都是再自然不过的 C 语言的编程过程，然后我们就可以在 linkList.c 文件中实现上述两个函数，继而在 main.c 中调用它们了。

然而上面我们定义的链表还只能对整型数据进行操作，如果下次你要用到一个存储字符串类型的链表，就只好把上面的过程重新来过。也许你觉得这个在原有代码基础上做略微修改的过程并不复杂，可是也许我们会不断的增加对于链表这个数据结构的操作，而需要用链表来存储的数据类型也越来越多，这些都意味着海量的代码和繁琐的后期维护工作。当你有了上百个存储不同数据类型的链表结构，每当你要增加一个操作，或者修改某个操作的传入参数，工作量会变大到像一 场灾难。

但是我们可以改造上述代码，让它能够处理你所想要让它处理的任何数据类型：实行，字符型，乃至任何你自己定义的 structure 类型。

####void*：万能的指针“挂钩”

几乎所有讲授 C 语言课程的老师都会告诉你：“指针是整个 C 语言的精髓所在。”而你也一直敬畏着指针，又爱又恨地使用着它。许多教材都告诉你，int * 叫做指向整型的指针，而 char * 是指向字符型的指针，等等不一而足。然而这里有一个另类的指针家族成员—— void *。不要按照通常的命名方式叫它做指向 void 类型的指针，它的正式的名字叫做：可以指向任意类型的指针。你一定注意到了“任意类型”这四个字，没错，实现多态，我们靠的就是它。

下面来改造我们的链表代码，在 linkList.h 里，如下：

```c
 typedef struct Node* linkList; 
 struct Node                                       // 链表节点
 { 
    void *data;                               // 存储的数据指针
    linkList next;                            // 指向下一个链表节点
 }; 

 linkList initialLinklist();                             // 初始化链表
 link newLinkList (void *data);                    // 建立新节点
 void insertFirst(linkList h, void *data);         // 在已有链表的表头进行插入节点操作
 void linkListOutput(linkList h);                         // 输出链表中数据到控制台
```

我们来看看现在这个链表和刚才那个只能存储整型数据的链表的区别。

当你把 Node 结构体里面的成员定义为一个整型数据，就好像把这个链表节点打造成了一个大小形状固定的盒子，你定义一个链表节点，程序进行编译的时候编译器就为你打造一 个这样的盒子：装一个 int 类型的数据，然后装一个 linkList 类型的指针。如果你想强行在这个盒子里装别的东西，编译器会告诉你，对不起，盒子的大小形状并不合适。所以你必须为了装各种各样类型的数据打造出不同的生 产盒子的流水线，想要装哪种类型数据的盒子，就开启对应的流水线来生产。

但是当你把结构体成员定义为 void *，一切都变得不同了。这时的链表节点不再像个大小形状固定的盒子，而更像一个挂钩，它可以挂上一个任意类型的数据。不管你需要存储什么类型的数据，你只要传递一个指针，把它存储到 Node 节点中去，就相当于把这个数据“挂”了上去，无论何时你都可以根据指针找到它。这时的链表仿佛变成了一排粘贴在墙上的衣帽钩，你可以挂一排大衣，可以挂一 排帽子，可以挂一排围巾，甚至，你可以并排挂一件大衣一顶帽子一条围巾在墙上。void * 初露狰狞，多态离 C 语言并不遥远。

###实现：你的多态你做主

当你真正开始着手做这个工作的时候，你会发现把数据放入链表中的操作和普通的存放 int 类型的链表的实现并没有什么大的区别，很方便。但是当你要把已经存进去的数据读取出来的时候，就有一点麻烦了。对于 void * 类型的指针，编译器只知道它里面存储了一个地址，但是关于这个地址里的数据类型，编译器是没有任何概念的。毕竟我们不能指望编译器什么都知道，什么都能替你做好，所以存进去的数据的类型，作为程序员的我们必须清楚的知道，并且在取出这个数据的时候，用这一类型的指针来对 void * 做强制类型转换。

为了方便的做到这一点，我采取的方法是在 Node 结构体中增加一个标识数据类型的域，并用一个枚举类型来存放这些数据类型。这时的 linkList.h 如下所示：

```c
 #ifndef LINKLIST_H 
 #define LINKLIST_H 

 typedef struct Node* linkList; 

 enum dataType 
 { 
    INT, 
    DOUBLE, 
    CHAR, 
    STRING 
 }; 

 struct Node                                               // 链表节点
 { 
    void *data;                                       // 存储的数据指针
    int dataType;                                     // 存储数据类型
    linkList next;                                    // 指向下一个链表节点
 }; 

 linkList initialLinklist();                               // 初始化链表
 linkList newLinkList (void *data, int dataType);          // 建立新节点
 void insertFirst(linkList h, void *data, int dataType);   // 在已有链表的表头进行插入节点操作
 void linkListOutput(linkList h);                          // 输出链表中数据到控制台

 #endif 
```

初始化链表，代码如下：

```c
 linkList initialLinklist() 
 { 
 linkList link = (linkList*)malloc(sizeof(*link)); 
    link->data = NULL; 
    link->dataType = -1; 
    link->next = NULL; 

    return link; 
 }
 ```
 
建立新节点，代码如下：

```c
linkList newLinkList (void *data, int dataType) 
{ 
    linkList link = (linkList*)malloc(sizeof(*link)); 
    link->data = data; 
    link->dataType = dataType; 
    link->next = NULL; 

    return link; 
}
```

在已有链表的表头进行插入节点，代码如下：

```c
void insertFirst(linkList h, void *data, int dataType) 
{ 
    linkList l = newLinkList(data, dataType); 
    l->next = h->next; 
    h->next = l; 
}
```

输出链表中数据到控制台，代码如下：

```c
void linkListOutput(linkList h) { 
    linkList p = h; 

    p = p->next; 
    while(p != NULL) { 
        switch(p->dataType) { 
            case 0: { 
                printf("%4d", *(int*)(p->data)); 
                break; 
            } 
           case 1: { 
                printf("%4f", *(double*)(p->data)); 
                break; 
           } 
           case 2: { 
                printf("%4c", *(char*)(p->data)); 
                break; 
            } 
            case 3: { 
                printf("%s ", (char*)(p->data)); 
                break; 
            } 
        } 
        p = p->next; 
    } 
    printf("\n"); 
} 
```

###小结

通过上面这个链表的小例子，大家可能已经看到了 C 语言的灵活性。这段代码虽然短并且功能简单，但是已经实现了多态性。这篇文章的本意并不在于想要告诉大家用 C 实现多态的方法，而多态的含义也无疑是更加广泛的。这篇文章的初衷其实是基于这样一点认识：面向对象是一种程序设计思想，而 C 语言则是一种编程语言。也许它并不是专门为了面向对象编程而设计，但是这绝不意味着它不能实现面向对象的程序设计。当然以上所展示的这几个操作，如果是用 别的编程语言，可能只要寥寥几行就可以完成，但是 C 语言想告诉我们的是：也许我不擅长，但是并不意味着我做不到。

--------------------------------------------------------------------------------------------------------------------

###C语言实现C++继承

    注：用结构加指针实现

```c
#include "stdio.h"
#include "stdlib.h"

//定义函数指针类型DISPLAYINTEGER，指向返回值为void，参数列表为(const int)的函数
typedef  void( *DISPLAYINTEGER)(const int);

//定义函数，将数字以十进制形式输出，该函数类型与DISPLAYINTEGER匹配
 void DisplayDecimal(const int Number) {
    printf("The decimal value is %d\n",Number);
 }
 //定义函数，将数字以八进制形式输出，该函数类型与DISPLAYINTEGER匹配
 void DisplayOctal(const int Number) {
    printf("The octal value is %o\n",Number);
 }
 //定义函数，将数字以十六进制形式输出，该函数类型与DISPLAYINTEGER匹配
 void DisplayHexadecimal (const int Number) {
    printf("The hexadecimal value is %x\n",Number);
 }
 /********************************************************************************************
 定义通用的显示数字函数DisplayNumber：
 （1）参数DisplayFormat是DISPLAYINTEGER函数指针类型，实参可以是以上定义的三个函数之一，通过传递不同的实参，
      将数字以各种格式输出；
 （2）Number是准备输出的数字
*********************************************************************************************/
void DisplayNumber(DISPLAYINTEGER  DisplayFormat,const int Number) {
    //调用以实参传入的函数，以某种格式输出整型数字
    DisplayFormat(Number);
}

int main(int argc, char* argv[]) {
    int Number=0;

    // 如果有数字形式的命令行参数，将其输出，否则输出0
    if(argc>1)  
        Number=atoi(argv[1]);
    //分别以三种格式将数字输出
    DisplayNumber(DisplayDecimal,Number);
    DisplayNumber(DisplayOctal,Number);
    DisplayNumber(DisplayHexadecimal,Number);

    return 0;
}
```

###C语言实现C++多态性

用C语言实现一些类似C＋＋的功能还是有好处的，一方面多重继承开销大，而且在底层编程，C用得比较多。

1.parent.h

```c
#ifndef PARENT_H
#define PARENT_H

#include "child1.h"
#include "child2.h"
struct parent_type {
    unsigned int id;
    //char classname[20];
    char * name;
    struct{
        struct parent_handle * (*init)(void * pvoid);
        int (*fp1)(struct parent_handle *ph); 
        //...
    }pfn;
    struct parent_type *next;
};


struct parent_handle{
    int i;
    union{
        struct child1_handle child_hdl1;
        struct child2_handle child_hdl2;
        //...
    }priv;
    struct parent_type *ptp;
};
```

2.parent.c

```c
#include "stdafx.h"

#include "parent.h"

static struct parent_type *class_list;

int register_child(struct parent_type *p) {
    p->next = class_list;
    class_list = p;
    return 0;
}

struct parent_handle * parent_init(void * pchild_priv, unsigned int id) {
    struct parent_type *p;
    
    for (p = class_list; p; p = p->next)
        if (p->id == id)
            return p->pfn.init(pchild_priv);
    
    printf("unable to find matching child class\n");
    return NULL;
}

int parent_fp1(struct parent_handle *ph) {
    if (!ph->ptp->pfn.fp1)
    return 0;
    
    return ph->ptp->pfn.fp1(ph);
}


int register_child(struct parent_type *p);

struct parent_handle * parent_init(void * pchild_priv, unsigned int id);

int parent_fp1(struct parent_handle *ph);
#endif 
```

3. child1.h

```c
#ifndef CHILD1_H
#define CHILD1_H

#define CHILD_1_ID 101

struct child1_handle {
    char child1_decr[20];
    int  child1_data;
    //...
};

extern struct parent_type child_tp1;

struct parent_handle * child1_init(void * pvoid);
int child1_fp1(struct parent_handle *ph);

#endif
```

4.child1.c

```c
#include "stdafx.h"

#include "parent.h"
#include "child1.h"
struct parent_type child_tp1= {CHILD_1_ID,"class child1",{&child1_init,&child1_fp1},0};
/*
struct parent_type  child1= {
    .id = CHILD_1_ID,
    .name  = "class child1",
    .fn = {
        .init   = &child1_init,
        .fp1   = &child1_fp1,
     },
};
*/
struct parent_handle * child1_init(void * pvoid) {
    //int ret;
    struct child1_handle *h;
    struct parent_handle *ph = (struct parent_handle *)malloc(sizeof(struct parent_handle/* *h */));
    if (!ph)
        return NULL;
     ph->ptp = &child_tp1;
    
     //todo
    h = &ph->priv.child_hdl1;
    strcpy(h->child1_decr,"hello child1!");
    h->child1_data=*((int*)(pvoid));
    //...
    return ph;
}


int child1_fp1(struct parent_handle *ph) {
    struct child1_handle *h = &ph->priv.child_hdl1;
    //todo
    printf("%s,data=%d\r\n",h->child1_decr,h->child1_data);
    //...
    
    return 0;
}
```

5.child2.h

```c
#ifndef CHILD2_H
#define CHILD2_H

#define CHILD_2_ID 102

struct child2_handle {
    int  child2_data;
    char child2_decr[20];
    //...
};

extern struct parent_type child_tp2;

int child2_fp1(struct parent_handle *ph);
struct parent_handle * child2_init(void * pvoid);

#endif
```

6.child2.c

```c
#include "stdafx.h"

#include "parent.h"
#include "child2.h"

struct parent_type child_tp2= {CHILD_2_ID,"class child2",{&child2_init,&child2_fp1},0};
/*
struct parent_type  child1= {
    .id = CHILD_1_ID,
    .name  = "class child1",
    .fn = {
        .init   = &child1_init,
        .fp1   = &child1_fp1,
    },
};
*/
struct parent_handle * child2_init(void * pvoid) {
     //int ret;
     struct child2_handle *h;
     struct parent_handle *ph = (struct parent_handle *)malloc(sizeof(struct parent_handle/* *h */));  
     //别忘了自己加deinit 把它释放
     if (!ph)
         return NULL;
     ph->ptp = &child_tp2;
    
     //todo
     h = &ph->priv.child_hdl2;
     strcpy(h->child2_decr,"hello child2!");
     h->child2_data=*((int*)(pvoid));
     //...
     return ph;
}


int child2_fp1(struct parent_handle *ph) {
    struct child2_handle *h = &ph->priv.child_hdl2;

    //todo
    printf("%s,data=%d\r\n",h->child2_decr,h->child2_data);
    //...

    return 0;
}
```

7. main.c

```c
// testClass.cpp : Defines the entry point for the console application.

#include "stdafx.h"

#include "parent.h"
#include "child1.h"
#include "child2.h"

static struct parent_handle *pchild_1_h,*pchild_2_h;

int main(int argc, char* argv[]) {
    //printf("Hello World!\n");
    int child1_priv=123;
    int child2_priv=456;
    register_child(&child_tp1);                 //注册子类1
    pchild_1_h = parent_init((void *)(&child1_priv),CHILD_1_ID);//初始化子类1
    if(pchild_1_h)
         parent_fp1(pchild_1_h);                 //执行子类1的fp1函数
    
    register_child(&child_tp2);                 //注册子类2
    pchild_2_h = parent_init((void *)(&child2_priv),CHILD_2_ID);//初始化子类2
    parent_fp1(pchild_2_h);                     //执行子类2的fp1函数
    return 0;
}
```

用C++实现很简单

```cpp
class parent {
    virturl int fp1 (); 
}

class child1 :public parent {
    char child1_decr[20];
    int  child1_data;
    int fp1 ();
}

class child2 :public parent {
    char child2_decr[20];
    int  child2_data;
    int fp1 ();
}

parent * pa;
child1 ch1b;
pa= &ch1b;
pa->fp1;

```
