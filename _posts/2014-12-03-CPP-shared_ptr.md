---
layout: post
title:  C++中的动态内存与智能指针
categories: [c++]
---

在C++中，我们通过`new`（在动态内存中为对象分配空间并初始化对象）和`delete`（销毁该对象，并释放内存）直接分配和释放动态内存。

如下代码：

```cpp
int *pi = new int;//pi 指向一个未初始化的int
```

有些人有这样的疑问，指针一定要`new`吗？其实指针和`new`没有什么关系。这里的`new`在动态内存里为对象分配了内存空间，并返回了一个指向该对象的指针。`new`是申请堆空间，不是在栈上分配内存，指针只要指向有效的内存空间就可以。比如：

```cpp
int i;
int *p = &i;
//p可以直接使用了
```

new直接初始化对象：

```cpp
int *pi = new int(128);//pi指向值为128的对象
string *ps = new string("christian");//*ps 指向“christian”的字符串
```
`new`分配`const`对象必须进行初始化，并且返回的是一个指向`const`对象的指针：

```cpp
const int *p = new const int(1024);//分配并初始化一个const int；
```

当然`new`申请内存分配的时候也不是都会成功的，一旦一个程序用光了他的所有可用内存（虽然这种情况一般很少发生），`new`表达式就会失败。这时候会抛出`bad_alloc`异常。所以我们需要通过`delete`来释放占用的内存。在这里注意`delete`并不是要删除指针，而是释放指针所指的内存。

```cpp
int i;
int *pi = &i;
string str = "dwarves";
double *pd = new double(33);
delete str; // 错误：str不是一个指针
delete pi;  // 错误：pi指向一个局部变量
delete pd;  // 正确
```

使用new和delete来管理动态内存常出的一些错误：

1.忘记delete，即导致了“内存泄漏”，

2.野指针。在对象已经被释放掉之后，（这里注意，此时的指针成为了悬垂指针，即指向曾经存在的对象，但该对象已经不再存在。结果未定义，而且难以检测。）这时候我们再次使用，会产生使用非法内存的指针。

不过如果我们需要保留指针，可以在delete以后将nullptr赋予指针，这样指针就不指向任何对象了，如下代码：

```cpp
auto p(new auto 42);
auto q = p;
delete p;
p = nullptr;
``` 

题外话：在测试这个问题的时候，我输出了下q的值发现还是42，并且没有报错，后来在delete p之后，我又给*p = 19;这个时候 p ,q的值在输出的时候都是19，也没有报错。这个代码其实根本就是错误的了，因为p,q已经没有有效的内存空间了。这里是释放了内存，但指针的值不变，指向的内存不会清0，指向的这片内存区域是待分配的，如果没有被其他数据覆盖的话，你就能幸运得输出这主要原因是你分配的内存小，没有继续分配，被占用的概率小所致。我用的xcode，换到VS下就正常报错了，是因为VS为了从编译器的角度上解决缓冲区溢出等问题，加上的这种功能，C++标准里面没有这么要求，所有xcode和gcc是不会检查的。所以在这里 建议大家写纯C++代码的时候用vs。

3.重复delete，就会使自由空间遭到破坏如：

```cpp
string *ps1 = new string ("one"),*ps2 = ps1;
delete ps1;
delete ps2;//ps2的内存已经被释放了
```

虽然显示的管理内存在性能上有一定的优势，但是随着多线程程序的出现和广泛使用，内存管理不佳的的情况变得更严重。所以C++标准库中的智能指针很好的解决了这些问题。

`auto_ptr`以对象的方式管理堆分配的内存，并在适当的时间（比如析构），释放内存。我们只需要将`new`操作返回的指针作为`auto_ptr`的初始值，而不需要调用`delete`：

```cpp
auto_ptr (new int);
```

但是`auto_ptr`在拷贝时会返回一个左值并且不能调用`delete[]`;所以在C++11中改用`shared_ptr`(允许多个指针指向一个对象),`unique_ptr`（“独占”所指向的对象）还有`weak_ptr`它是一种不控制所指对象生存期的智能指针，指向`shared_ptr`所管理的对像，在`memory`头文件中。

```cpp
shared_ptr<int> pi;//指向int
```

当然我们也可以shared_ptr和new来结合使用，但是必须使用直接初始化的形式来初始化一个智能指针，

```cpp
shared_ptr<int> p1 = new int(1024);//error：必须使用直接初始化的形式
shared_ptr<int> p2(new int(1024));
```

但是最好不要混合使用普通指针和智能指针，最安全的分配和使用动态内存的方法是调用`make_shared`的标准库函数。在使用它的时候，必须指定想要创建的对象类型。

```cpp
shared_ptr<int >pi = make_shared<int>(1);//指向一个值为1的int的shared_ptr
shared_ptr<string>ps =  make_shared<string>(10,'a');//ps为指向“aaaaaaaaaa”的string
```

如果我们不传递任何参数，对象会进行值初始化

```cpp
shared_ptr<int>pi = make_shared<int>();//初始化默认值为0；
```

`shared_ptr` 实现了引用计数型的智能指针，当进行拷贝的时候，计数器都会递增。而对一个对象进行赋值时，赋值操作符减少左操作数所指对象的引用计数（减1，如果引用计数为减至0，则删除对象），并增加右操作数所指对象的引用计数（加1），如下代码：

```cpp
auto p = make_shared<int>(13);//p 指向的对象只有p一个引用者
auto q(p);//此时对象有两个引用者
auto r = make_shared<int>(10);
r = q;
```

此时r的引用技术为0，r原指对象被自动释放。q的引用计数增加。

`weak_ptr`的使用和析构都不会改变`shared_ptr`的引用计数。`weak_ptr`可以使用一个非常重要的成员函数`lock()`从被观测的`shared_ptr`获得一个可用的`shared_ptr`对象， 从而操作资源。但当`expired()==true`的时候，`lock()`函数将返回一个存储空指针的`shared_ptr`.如下：

```cpp
#include <boost/smart_ptr.hpp>
#include <boost/make_shared.hpp>
using namespace boost;
using namespace std;
int _tmain(int argc, _TCHAR* argv[]) {
    shared_ptr<int> sp(new int(10));
    assert(sp.use_count() == 1);
    weak_ptr<int> wp(sp); //从shared_ptr创建weak_ptr
    assert(wp.use_count() == 1);
    if (!wp.expired())//判断weak_ptr观察的对象是否失效
    {
        shared_ptr<int> sp2 = wp.lock();//获得一个shared_ptr
        *sp2 = 100;
        assert(wp.use_count() == 2);
    }
    assert(wp.use_count() == 1);
return 0;
}
```

当我们定义一个`unique_ptr`的时候，需要将其绑定到一个`new`返回的指针。

只能有一个`uniqu_ptr`指向对象，也就是说它不能被拷贝，也不支持赋值。但是我们可以通过`move`来移动

```cpp
std::unique_ptr<int> p1(new int(5));
std::unique_ptr<int> p2 = p1; // 编译会出错
std::unique_ptr<int> p3 = std::move(p1); // 转移所有权, 现在那块内存归p3所有, p1成为无效的指针.
 
p3.reset(); //释放内存.
p1.reset(); //实际上什么都没做.
```
