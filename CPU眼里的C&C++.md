# C++基础语法知识点

## C++的可见性

​	可见性关键字，对于程序的性能和运行没有任何的影响的，其只是语法上的功能，有利于程序员组织和划分语法组织，增强可读性。关键字可分为以下三种：

* Public：公开，任何该类或子类的实例都可以访问。
* Private：私有，意味着只有定义该成员的当前类可以访问，即使为当前类的实例也无法访问。
* Protected：保护， 任何该类或子类都可以访问，定义类的代码中有效。

>注：类成员默认为Private私有，结构体成员默认为Public公开。





# CPU眼里的C/C++

​	这是一本在通过**Compiler Explorer**的网页程序查看代码的汇编语言，对比高级语言的语法，查看语法对于CPU底层而言的异同，加深对于C/C++语法编程的深刻认识的编程书籍。学习作者的分析方法，在日后也可借用该网页程序分析自身编写程序的优缺点。



## C++的引用与指针的区别

​	C++的引用本质上是变量的别名，在语法格式上等价于指针常量```void* const p```，引用在声明的同时就需要进行赋值，确定其引用的对象。一旦赋值后就不可再次赋值。

>```
>int var = 1;
>int& a = &var;//等价于 int* const p = &var
>a = 2;
>```

​	对于引用a的直接赋值操作 都会对变量var做出改变，引用作为指针使用的语法糖，可以在一些场景下使得代码的可读性增加。



## C++的构造函数

​	C++的类中有一些类成员变量，以及一些函数，为了在定义类的同时，初始化类成员变量（否则类成员变量的值为内存中的随机值无法确认），所以需要编写构造函数（函数名称与类名称一致的函数），构造函数在编译后由编译器确定在合适的位置调用，所以在代码运行的时候，建立类实例的同时会自动的调用类的构造函数。

 >```
 >Class A:
 >{
 >	public:
 >	int a;
 >	A()
 >	{
 >		a = 1;
 >	}
 >}
 >```



## C++的析构函数

​	C++析构函数，字如其名，是解构函数，对构造函数初始化的变量进行内存释放，在类实例被释放的同时自动调用（机制与构造函数一直，由编译器决定自动调用的）。

> ```
> Class A:
> {
> 	public:
> 	int a:
> 	~A()
> 	{
> 	}
> }
> ```



## C++的虚函数



## C++的多态