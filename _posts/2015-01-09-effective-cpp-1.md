---
layout: post
title: Effective C++ —— 让自己习惯C++
categories: [C++]
---

##条款01：视C++为一个语言联邦
为了更好的理解C++，我们将C++分解为四个主要次语言：

-  C：说到底C++仍是以C为基础。区块，语句，预处理器，内置数据类型，数组，指针统统来自C。
-  Object-Oreinted C++：这一部分是面向对象设计之古典守则在C++上的最直接实施。类，封装，继承，多态，virtual函数等等...
-	Template C++：这是C++泛型编程部分。
-	STL：STL是个template程序库。容器（containers），迭代器（iterators），算法（algorithms）以及函数对象（function objects）...

	**请记住：**
	这四个次语言，当你从某个次语言切换到另一个，导致高效编程守则要求你改变策略。C++高效编程守则视状况而变化，取决于你使用C++的哪一部分。

##条款02：尽量以const，enum，inline替换#define     

这个条款或许可以改为“宁可 以编译器替换预处理器”。**即尽量少用预处理。**

编译过程：.c文件－－预处理－－>.i文件－－编译-－>.o文件－-链接－－>bin文件

预处理过程扫描源代码，对其进行初步的转换，产生新的源代码提供给编译器。检查包含预处理指令的语句和宏定义，并对源代码进行相应的转换。预处理过程还会删除程序中的注释和多余的空白字符。可见预处理过程先于编译器对源代码进行处理。预处理指令是以#号开头的代码行。例

	#define ASPECT_RATIO 1.653

记号名称`ASPECT_RATIO`也许从未被编译器看见，也许在编译器开始处理源代码之前它就被预处理器移走了。即编译源代码时`ASPECT_RATIO`已被1.653取代。`ASPECT_RATIO`可能并未进入记号表（symbol table）。替换：

	const double AspectRatio = 1.653；

好处应该有：多了类型检查，因为`#define `只是单纯的替换，而这种替换在目标码中可能出现多份1.653；改用常量绝不会出现相同情况。

常量替换#define两点**注意：**

-	定义常量指针：

		const char *authorName = "Shenzi";
		cosnt std::string authorName("Shenzi");	


-	类专属常量：`static const int NumTurns = 5；`//static 静态常量 所有的对象只有一份拷贝。
    万一你编译器不允许“static整数型class常量”完成“in calss初值设定”（即在类的声明中设定静态整形的初值），我们可以通过枚举类型予以补偿：`enum { NumTurns = 5 }；`
-	取一个const的地址是合法的，但取一个enum的地址就不合法，而取一个#define的地址通常也不合法.如果你不想让别人获取一个pointer或reference指向你的某个整数常量，enum可以帮助你实现这个约束。

例：`#define CALL_WITH_MAX(a,b)    f((a) > (b)) ? (a) : (b))`
宏看起来像函数，但不会招致函数调用带来的额外开销，而是一种简单的替换。替换：

	template<typename T>
	inline void callWithMax(cosnt T &a, cosnt T &b) {
		f(a > b ? a : b);
	}

`callWithMax`是个真正的函数，它遵循作用于和访问规则。

**请记住：**

-	对于单纯常量，最好以const对象或enums替换#defines；
-	对于形似函数的宏，最好改用inline函数替换#defines。        

##条款03：尽可能使用const
const允许你告诉编译器和其他程序员某值应保持不变，只要“某值”确实是不该被改变的，那就该确实说出来。
关键字const多才多艺：

例：

	char greeting[] = "Hello";
	char *p = greeting;    //指针p及所指的字符串都可改变；
	const char *p = greeting; //指针p本身可以改变，如p = &Anyother；p所指的字符串不可改变；
	char * cosnt p = greeting;  //指针p不可改变，所指对象可改变；
	const char * const p = greeting;    //指针p及所致对象都不可改变；

说明：

-	如果关键字`const`出现在星号左边，表示被指物事常量。`const char *p`和`char const *p`两种写法意义一样，都说明所致对象为常量；
-	如果关键字`const`出现在星号右边，表示指针自身是常量。STL例子：

		const std::vector<int>::interator iter = vec.begin();
			//作用像T *const, ++iter 错误：iter是const
		
		std::vector<int>::const_iterator cIter = vec.begin();
			//作用像const T*，*cIter = 10 错误：*cIter是const


**以下几点注意：**

-	令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而不至于放弃安全性和高效性。
    例：`const Rational operator* （const Rational &lhs, cosnt Rational &rhs）;`
-	const成员函数使class接口比较容易被理解，它们使“操作const对象”称为可能；
    说明：声明为const的成员函数，不可改变non-static成员变量，在成员变量声明之前添加mutable可让其在const成员函数中可被改变。

		const_cast<char &>(static_cast<const TextBlock &>(*this))[position];
		//static_cast 将TextBlock &转为const TextBlock &；
		//const_cast将返回值去掉const约束；


**请记住：**

-	将某些东西声明为const可帮助编译器侦测出错误用法。const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体；
-	编译器强制实施bitwise constness，但你编写程序时应该使用“概念上的车辆”（conceptual constness）；
-	当cosnt和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复。

##条款04：确定对象被使用前已先被初始化
永远在使用对象之前先将它初始化。对于无任何成员的内置类型，你必须手工完成此事。至于内置类型以外的任何其它东西，初始化责任落在构造函数身上，确保每一个构造函数都将对象的每一个成员初始化。

赋值和初始化：
C++规定，对象的成员变量的初始化动作发生在进入构造函数本体之前。所以应将成员变量的初始化置于构造函数的初始化列表中。

	ABEntry::ABEntry(const std::string& name, const std::string& address,const std::list<PhoneNumber>& phones) { 
		theName = name;                //这些都是赋值，而非初始化
		theAddress = address;          //这些成员变量在进入函数体之前已调用默认构造函数，接着又调用赋值函数，
		thePhones = phones;            //即要经过两次的函数调用。            
		numTimesConsulted = 0;
	} 
	
	ABEntry::ABEntry(const std::string& name, const std::string& address,const std::list<PhoneNumber>& phones) 
		: theName(name),                    //这些才是初始化 
		theAddress(address),                //这些成员变量只用相应的值进行拷贝构造函数，所以通常效率更高
		thePhones(phones),
		numTimesConsulted(0)
		{    }
	 
所以，对于非内置类型变量的初始化应在初始化列表中完成，以提高效率。而对于内置类型对象，如numTimesConsulted（int），其初始化和赋值的成本相同，但为了一致性最好也通过成员初始化表来初始化。如果成员变量时const或reference，它们就一定需要初值，不能被赋值。

C++有着十分固定的“成员初始化次序”。基类总是在派生类之前被初始化，而类的成员变量总是以其说明次序被初始化。所以：当在成员初始化列表中列各成员时，最好总是以其声明次序为次序。
请记住：

-	为内置对象进行手工初始化，因为C++不保证初始化它们；
-	构造函数最好使用成员初始化列表，而不要在构造函数本体内使用赋值操作。初始化列表列出的成员变量，其排列次序应该和它们在类中的声明次序相同；
-	为免除“跨编译单元之初始化次序”问题，请以local static对象替换non-local static对象。   

