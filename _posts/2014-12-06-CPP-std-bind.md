---
layout: post
title: C++11中的std::bind
categories: [c++]
---

###`std::bind1st`和`std::bind2nd`

bind是这样一种机制，它可以预先把指定可调用实体的某些参数绑定到已有的变量，产生一个新的可调 用实体，这种机制在回调函数的使用过程中也颇为有用。C++98中，有两个函数bind1st和bind2nd，它们分别可以用来绑定functor的第一个和第二个参数，它们都是只可以绑定一个参数。各种限制，使得bind1st和bind2nd的可用性大大降低。

如果通过上面的内容，你还没有明白std::bind1st和std::bind2nd到底是何方圣神到底是什么东西，那就看这段代码示例：

```cpp
#include <iostream>
#include <functional>
#include <algorithm>
#include <vector>
using namespace std;

int main()
{
    vector<int> coll;
    for (int i = 1; i <= 10; ++i)
    {
        coll.push_back(i);
    }

    // 查找元素值大于10的元素的个数
    // 也就是使得10 < elem成立的元素个数 
    int res = count_if(coll.begin(), coll.end(), bind1st(less<int>(), 10));
    cout << res << endl;

    // 查找元素值小于10的元素的个数
    // 也就是使得elem < 10成立的元素个数 
    res = count_if(coll.begin(), coll.end(), bind2nd(less<int>(), 10));
    cout << res << endl;

    return 0;
}
```

对于上面的代码，less<int>()其实是一个仿函数，如果没有std::bind1st和std::bind2nd，那么我们可以这样使用less<int>()，代码如下：

```cpp
less<int> functor = less<int>();
bool bRet = functor(10, 20); // 返回true
```

`less<int>()`这个仿函数对象是需要两个参数的，比如10<20进行比较，那么10叫做left参数，20叫做right参数。

当使用`std::bind1st`的时候，就表示绑定了left参数，也就是left参数不变了，而right参数就是对应容器中的element；

当使用`std::bind2nd`的时候，就表示绑定了right参数，也就是right参数不变了，而left参数就是对应容器中的element。


###`std::bind`介绍

C++11中提供了`std::bind。bind()`函数的意义就像它的函数名一样，是用来绑定函数调用的某些参数的。

bind的思想实际上是一种延迟计算的思想，将可调用对象保存起来，然后在需要的时候再调用。而且这种绑定是非常灵活的，不论是普通函数、函数对象、还是成员函数都可以绑定，而且其参数可以支持占位符，比如你可以这样绑定一个二元函数`auto f = bind(&func, _1, _2);`，调用的时候通过f(1,2)实现调用。


###`std::bind`使用方法

先看看   [《C++11中的std::function》](http://xiaoyifeibupt.github.io/blog/CPP-std-function.html) 中那段代码，std::function可以绑定全局函数，静态函数，但是绑定类的成员函数时，必须要借助`std::bind`的帮忙。但是话又说回来，不借助`std::bind`也是可以完成的，只需要传一个`*this`变量进去就好了，比如：

```cpp
#include <iostream>
#include <functional>
using namespace std;

class View
{
public:
    void onClick(int x, int y)
    {
        cout << "X : " << x << ", Y : " << y << endl;
    }
};

// 定义function类型, 三个参数
function<void(View, int, int)> clickCallback;

int main(int argc, const char * argv[])
{
    View button;

    // 指向成员函数
    clickCallback = &View::onClick;

    // 进行调用
    clickCallback(button, 10, 123);
    return 0;
}
再来一段示例谈谈怎么使用std::bind代码：

#include <iostream>
#include <functional>
using namespace std;

int TestFunc(int a, char c, float f)
{
    cout << a << endl;
    cout << c << endl;
    cout << f << endl;

    return a;
}

int main()
{
    auto bindFunc1 = bind(TestFunc, std::placeholders::_1, 'A', 100.1);
    bindFunc1(10);

    cout << "=================================\n";

    auto bindFunc2 = bind(TestFunc, std::placeholders::_2, std::placeholders::_1, 100.1);
    bindFunc2('B', 10);

    cout << "=================================\n";

    auto bindFunc3 = bind(TestFunc, std::placeholders::_2, std::placeholders::_3, std::placeholders::_1);
    bindFunc3(100.1, 30, 'C');

    return 0;
}
```

上面这段代码主要说的是bind中`std::placeholders`的使用。 `std::placeholders`是一个占位符。当使用bind生成一个新的可调用对象时，`std::placeholders`表示新的可调用对象的第 几个参数和原函数的第几个参数进行匹配，这么说有点绕。比如：

```cpp
auto bindFunc3 = bind(TestFunc, std::placeholders::_2, std::placeholders::_3, std::placeholders::_1);

bindFunc3(100.1, 30, 'C');
```

可以看到，在bind的时候，第一个位置是TestFunc，除了这个，参数的第一个位置为占位符`std::placeholders::_2`，这就表示，调用bindFunc3的时候，它的第二个参数和TestFunc的第一个参数匹配，以此类推。

**注意**

-   bind预先绑定的参数需要传具体的变量或值进去，对于预先绑定的参数，是pass-by-value的；
-   对于不事先绑定的参数，需要传`std::placeholders`进去，从_1开始，依次递增。placeholder是pass-by-reference的；
-   bind的返回值是可调用实体，可以直接赋给`std::function`对象；
-   对于绑定的指针、引用类型的参数，使用者需要保证在可调用实体调用之前，这些参数是可用的；
-   类的this可以通过对象或者指针来绑定。

###使用`std::bind`的原因

当我们厌倦了使用`std::bind1st`和`std::bind2nd`的时候，现在有了`std::bind`，你完全可以放弃使用`std::bind1st`和`std::bind2nd`了。`std::bind`绑定的参数的个数不受限制，绑定的具体哪些参数也不受限制，由用户指定，这个bind才是真正意义上的绑定。

使用`std::bind`生成一个可调用对象，这个对象可以直接赋值给`std::function`对象；在类中有一个`std::function`的变量，这个`std::function`由`std::bind`来赋值，而`std::bind`绑定的可调用对象可以是Lambda表达式或者类成员函数等可调用对象。
