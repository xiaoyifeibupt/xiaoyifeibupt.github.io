---
layout: post
title: Effective C++ —— 资源管理
categories: [C++]
---

所谓资源就是，一旦用了它，将来必须还给系统。C++程序中最常使用的资源就好似动态分配内存（如果你`new`了，却忘了`delete`，会导致内存泄露），但内存只是你必须管理的众多资源之一。其它常见的有文件描述符（file descriptors）、互斥器（mutex）、图形界面中的字形和画刷。数据库连接以及网络sockets。当你不使用它们时，记得还给系统。

当你考虑到异常、函数内多重回传路径、程序维护员改动软件却没能充分理解随之而来的冲击，那么资源管理就显得复杂的多。

##条款13：以对象管理资源
例：

	void f(){ 
		Investment *pInv = createInvestment(); 
		...  //这里存在诸多“不定因素”，可能造成delete pInv；得不到执行，这可能就存在潜在的内存泄露。
		delete pInv;
	} 
    
解决方法：把资源放进对象内，我们便可依赖C++的“析构函数自动调用机制”确保资源被释放。

许多资源被动态分配于堆内而后被用于单一区块或函数内。它们应该在控制流离开那个区块或函数时被释放。标准程序库提供的`auto_ptr`正是针对这种形势而设计的特制产品。`auto_ptr`是个“类指针对象”，也就是所谓的“智能指针”，其析构函数自动对其所指对象调用`delete`。

	void f() { 
		std::auto_ptr<Investment> pInv(createInvestment()); 
		... 
	} //函数退出，auto_ptr调用析构函数自动调用delete，删除pInv；无需显示调用delete。

“以对象管理资源”的两个关键想法：

-	获得资源后立刻放进管理对象内（如`auto_ptr`）。每一笔资源都在获得的同时立刻被放进管理对象中。“资源取得时机便是初始化时机”（Resource Acquisition Is Initialization；RAII）。
-	管理对象运用析构函数确保资源被释放。即一旦对象被销毁，其析构函数被自动调用来释放资源。

由于`auto_ptr`被销毁时会自动删除它所指之物，所以不能让多个`auto_ptr`同时指向同一对象。所以`auto_ptr`若通过copying函数复制它们，它们会变成NULL，而复制所得的指针将取得资源的唯一拥有权！看下面例子：

	std::auto_ptr<Investment> pInv1(createInvestment()); //pInv1指向createInvestment()返回物；
	std::auto_ptr<Investment> pInv2(pInv1); //现在pInv2指向对象，而pInv1被设为NULL;
	pInv1 = pInv2; //现在pInv1指向对象，而pIn2被设为NULL;
    
受`auto_ptr`管理的资源必须绝对没有一个以上的auto_ptr同时指向它。即“有你没我，有我没你”。

`auto_ptr`的替代方案是引用计数型智能指针（reference-counting smart pointer；SCSP）、它可以持续跟踪共有多少对象指向某笔资源，并在无人指向它时自动删除该资源。
TR1的`tr1::shared_ptr`就是一个"引用计数型智能指针"。

	void f() { 
	... 
	std::tr1::shared_ptr<Investment>  pInv1(createInvestment()); //pInv1指向createInvestment()返回物；
	std::tr1::shared_ptr<Investment>  pInv2(pInv1);  //pInv1，pInv2指向同一个对象；
	pInv1 = pInv2;  //同上，无变化
	... 
	}  //函数退出，pInv1，pInv2被销毁，它们所指的对象也竟被自动释放。

`auto_ptr`和`tr1::shared_ptr`都在其析构函数内做`delete`而不是`delete[]`，也就意味着在动态分配而得的数组身上使用`auto_ptr`或`tr1::shared_ptr`是个潜在危险，资源得不到释放。也许`boost::scoped_array`和`boost::shared_array`能提供帮助。还有，`vector`和`string`几乎总是可以取代动态分配而得的数组。

请记住：

-	为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。
-	两个常被使用的RAII类分别是auto_ptr和tr1::shared_ptr。后者通常是较佳选择，因为其拷贝行为比较直观。若选择auto_ptr，复制动作会使他（被复制物）指向NULL。   

##条款14：在资源管理类中小心拷贝行为
我们在条款13中讨论的资源表现在堆上申请的资源，而有些资源并不适合被auto_ptr和tr1::shared_ptr所管理。可能我们需要建立自己的资源管理类。例：

	void lock(Mutex *pm);     //锁定pm所指的互斥量
	unlock(Mutex *pm);        //将pm解除锁定

我们建立的资源管理类可能会是这样：

	class Lock {
	public: 
		explicit Lock(Mutex *pm) : mutexPtr(pm) {
		lock(mutexPtr); 
	} 
	~Lock() { 
		unlock(mutexPtr); 
	} 
	
	private: 
		Mutex *mutexPtr; 
	};
 

但是，如果Lock对象被复制，会发生什么事？？？

“当一个RAII对象被复制，会发生什么事？”大多数时候你会选择一下两种可能：

-	禁止复制。如果复制动作对RAII类并不合理，你便应该禁止之。禁止类的`copying`函数参见条款6。
-	对底层资源使用”引用计数法“。有时候我们又希望保有资源，直到它的最后一个使用者被销毁。这种情况下复制RAII对象时，应该将资源的”被引用计数“递增。`tr1::shared_ptr`便是如此。

通常只要内含一个`tr1::shared_ptr`成员变量，RAII类便可实现”引用计数“行为。

	class Lock {
	public: 
	explicit Lock(Mutex *pm) : mutexPtr(pm, unlock) {
	//由于tr1::shared_ptr缺省行为是”当引用计数为0时删除其所指物“，幸运的是 
	//我们可以指定”引用计数“为9时被调用的所谓”删除器“，即第二个参数unlock
		lock(mutexPtr.get()); 
	} 
	private:
		std::tr1::shared_ptr<Mutex> mutexPtr; 
	}; 

本例中，并没说明析构函数，因为没有必要。编译器为我们生成的析构函数会自动调用其non-static成员变量（mutexPtr）的析构函数。而`mutexPtr`的析构函数会在互斥量”引用计数“为0时自动调用`tr1::shared_ptr`的删除器（unlock）。

`Copying`函有可能被编译器自动创建出来，因此除非编译器所生成版本做了你想要做的事，否则你得自己编写它们。

请记住：

-	复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
-	普遍而常见的RAII类拷贝行为是：抑制拷贝，施行引用计数法。不过其它行为也可能被实现。   

##条款15：在资源管理类中提供对原始资源的访问
前几个条款提到的资源管理类很棒。它们是你对抗资源泄漏的堡垒。但这个世界并不完美，许多APIs直接指涉资源，这时候我们需要直接访问原始资源。

这时候需要一个函数可将RAII对象（如`tr1::shared_ptr`）转换为其所内含之原始资源。有两种做法可以达成目标：显示转换和隐式转换。

`tr1::shared_ptr`和`auto_ptr`都提供一个`get`成员函数，用来执行显示转换，也就是返回智能指针内部的原始指针（的复件）。就像所有智能指针一样， `tr1::shared_ptr`和`auto_ptr`也重载了指针取值操作符（`operator->`和`operator*`），它们允许隐式转换至底部原始指针。（即在对智能指针对象实施`->`和`*`操作时，实际被转换为被封装的资源的指针。）

	class Font{
	public: 
		... 
		FontHandle get() const {//FontHandle 是资源；显示转换函数		 
			return f; 
		}
		operator FontHandle() const { //隐式转换  这个值得注意，可能引起“非故意之类型转换	
			return f; 
		} 
		... 
	}; 

是否该提供一个显示转换函数（例如`get`成员函数）将RAII类转换为其底部资源，或是应该提供隐式转换，答案主要取决于RAII类被设计执行的特定工作，以及它被使用的情况。

显示转换可能是比较受欢迎的路子，但是需要不停的get，get；而隐式转换又可能引起“非故意之类型转换”。

请记住：

-	APIs往往要求访问原始资源，所以每一个RAII类应该提供一个“取得其所管理之资源”的方法。
-	对原始资源的访问可能经由显示转换或隐式转换。一般而言显示转换比较安全，但隐式转换对客户比较方便。   

##条款16：成对使用new和delete时要采取相同形式
先看下一下代码：

	std::string *stringArray = new std::string[100];
	... 
	delete stringArray;

使用了`new`动态申请了资源，也调用了`delete`释放了资源。但这代码存在“不明确行为”。`stringArray`对象中的99个不太可能被适当删除，因为它们的析构函数很可能没被调用。

当我们使用`new`，有两件事情发生：第一，内存被分配出来；第二，针对此内存会有一个（或更多）构造函数被调用。当你使用`delete`，也有两件事发生：针对此内存会有一个（或多个）析构函数被调用，然后内存才被释放。`delete`的最大问题在于：即将被删除的内存之内究竟有多少对象？这个问题的答案决定了有多少个析构函数必须被调用起来。

解决以上问题事实上很简单：如果你调用`new`时使用`[]`，你必须在对应调用`delete`时也使用`[]`。如果你调用`new`时没有使用`[]`，那么也不该在对应调用`delete`时使用`[]`。

最好尽量不要对数组形式作`typedefs`动作。因为这样容易引起`delete`操作的“疑惑”（需不需要`[]`呢？？？）。

**请记住：**

-	如果你在`new`表达式中使用`[]`，必须在相应的`delete`表达式中也使用`[]`。如果你在`new`表达式中不使用`[]`，一定不要在相应的`delete`表达式中使用`[]`。    

##条款17：以独立语句将newed对象置入智能指针
为了避免资源泄漏的危险，最好在单独语句内以智能指针存储`newed`所得对象。即：

	int priority();
	void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);
	std::tr1::shared_ptr<Widget> pw(new Widget); //即在传入函数之前对智能指针初始化，而不是在传入参数中                                                                               
	//对其初始化，因为那样可能引起操作序列的问题。
	processWidget(pw, priority()); 

**请记住：**

-	以独立语句将`newed`对象存储于（置入）智能指针内。如果不这样做，一旦异常抛出，有可能导致难以察觉的资源泄漏。 


