# C++常见关键字

## Const

#### 1.常量与指针

```c++
//从右向左读法
const int *m1 = new int(10); = int const *m1 = new int(10); //常量指针
// m1 is a pointer to const int， *m1可以指向别的int，不能通过*m1 修改int内容，但是可以通过别的方式修改其内容，比如用别的指针：
int* m3 = (int*)m1;
*m3 = 1;
//或者如下面的形式
int i = 10;
const int *m1 = &i;
i = 1;

int* const m2 = new int(20); //指针常量
// m2 is a const pointer to int，*m2不能指向别的int，但是可以修改其内容

//我们可以使用在函数参数中使用const来防止函数调用者改变参数的值。
//在函数参数中指针常量时表示不允许将该指针指向其他内容。
//在函数参数中使用常量指针时表示在函数中不能改变指针所指向的内容。
```

#### 2.**常量与引用**

```c++
//因为引用就是另一个变量的别名，它本身就是一个常量。所以不存在引用常量，只存在常量引用：
int i=1;
const int & m=i; //正确
int& const m=i; //错误

//Google的C++编程规范对引用参数的使用建议是：“引用传递的参数必须加上const”，通过指针实现输出。
void fun(std::string &str);
//这样的函数在涉及多线程编程时，可能会引起内存泄漏。
//因此推荐这样写：（出于易读性考虑）
//1、输入参数
void fun(const std::string &str);
//2、输出参数
void fun(std::string *str);

在函数输入参数中常用，为了减少形参复制到实参过程的复制开销。同时在复制构造函数中使用，防止复制构造函数的形参复制到实参过程中调用复制构造函数陷入无限循环。
```

#### 3.常量函数

```c++
//在一个函数的签名后面加上关键字const后该函数就成了常量函数。对于常量函数，最关键的不同是编译器不允许其修改类的数据成员。
class Test
{
    public:
    void func() const;
    private:
    int intValue;
};

void Test::func() const
{
    intValue = 100;
}
//上面的代码中，常量函数func函数内试图去改变数据成员intValue的值，因此将在编译的时候引发异常。
//当然，对于非常量的成员函数，我们可以根据需要读取或修改数据成员的值。但是，这要依赖调用函数的对象是否是常量。通常，如果我们把一个类定义为常量，我们的本意是希望他的状态（数据成员）不会被改变。
class Fred{
    public:
    void f1() const;
    void f2();
};

void UserCode(Fred& changeable, const Fred& unChangeable)
{
    changeable.f1(); // 正确，非常量对象可以调用常量函数。
    changeable.f2(); // 正确，非常量对象也允许修改调用非常量成员函数修改数据成员。
    unChangeable.f1(); // 正确，常量对象只能调用常量函数。因为不希望修改对象状态。
    unChangeable.f2(); // 错误！常量对象的状态不能被修改，而非常量函数存在修改对象状态的可能
}


//const 修饰类的数据成员
//const数据成员只在某个对象生存期内是常量，而对于整个类而言却是可变的。因为类可以创建多个对象，不同的对象其const数据成员的值可以不同。

```



## volatile

 volatile 关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素更改，比如：操作系统，硬件或者其他线程等。当要求使用 volatile 声明的变量的值的时候，系统总是重新从它所在的内存读取数据，即使它前面的指令刚刚从该处读取过数据。

一般说来，volatile用在如下的几个地方：  1) 中断服务程序中修改的供其它程序检测的变量需要加volatile；  2) 多任务环境下各任务间共享的标志应该加volatile；  3) 存储器映射的硬件寄存器通常也要加volatile说明，因为每次对它的读写都可能由不同意义；



## 类型转换相关的关键字：static_cast, reinterpret_cast, dynamic_cast, const_cast

### static_cast-**运算符完成相关类型之间的转换**

【特点】：静态转换，在编译处理期间。
【应用场合】：主要用于C++中内置的基本数据类型之间的转换，但是没有运行时类型的检测来保证转换的安全性。

+ 用于基类和子类之间的指针或引用之间的转换，这种转换把子类的指针或引用转换为基类表示是安全的；
+ 把void类型的指针转换成目标类型的指针（不安全）。
+ 用于基本数据类型之间的转换，如把int转换成char, 不能用于两个不相关的类型转换。
+ 不能把const对象转换成非const对象。

```c++
int i=10;
float f=static_cast <float>(i);
long l = static_cast <long>(i);
char c = static_cast<char>(i);


int * q=static_cast < int* >(malloc(100));
```

### **reinterpret_cast-处理互不相关类型之间的转换**

【特点】： 重解释类型转换

【应用场合】: 它可以转化任何的内置数据类型为其他的类型，同时它也可以把任何类型的指针转化为其他的类型；它的机理是对二进制进行重新的解释，不会改变原来的格式。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，再把该整数转换成原类型的指针，还可以得到原先的指针值）。一种最有可能出问题的最不安全的类型转换。

```c++
//eg 1:
const int sz = 100; // 定义数组大小，标准C++提倡用常型变量（而不是常数或
// 符号常量宏）
struct X {int a[sz];}; // 只包含一个整数数组的结构
X x; // 定义结构变量，此时结构中的数组元素的值无意义（需要初始化）
int *px = reinterpret_cast<int *> (&x); // 为了初始化，先把结构转化为int数组
for (int *i = px; i < px + sz; i++) *i = 0; // 将每个数组元素的值初始化为0
print(reinterpret_cast<X *> (px)); // 重新转换成结构指针，以便使用
// 也可以直接使用原来的标识符x
// 此语句相当于print(&x);

//eg 2:
int a=10;
double* b=reinterpret_cast<double*>(a); 
//b的转换结果为0x0000000a
```



### **dynamic_cast-处理基类型到派生类型的转换**

【应用场景】：基类必须有虚函数，即为多态时，可以转换

```c++
class Base
{
public:
　　 virtual int test(){return 0;} //基类中存在虚函数，故在派生类中存在虚函数指针指向虚函数表。
};

class Derived:public Base
{
public:
　　 virtual int test(){return 1;}
};

int main()
{
    Base cbase;
    Derived cderived;
    Base *p1=new Base;
    Base *p2=new Derived;
    Derived* pD1=dynamic_cast<Derived*>(p1);//p1没有真正指向派生类,pD1置为0
    Derived* pD2=dynamic_cast<Derived*>(p2); //正确
    //Derived& pd1=dynamic_cast<Derived&>(*p1);//p1没有真正指向派生类，pd1抛出异常
    Derived& pd2=dynamic_cast<Derived&>(*p2);//正确
    return 0;
}
```

### **const_cast-移除变量的const或volatile限定符**

【特点】：去常转换，编译时执行。 【应用场合】：const_cast操作不能在不同的种类间转换。相反，它仅仅把它作用的表达式转换成常量。它可以使一个本来不是const类型的数据转换成const类型的，或者把const属性去掉。

```c++
const int i = 0;
int *pi;
pi = &i; // 错误
pi = (int *)&i; // 被反对
pi = const_cast<int *>(&i); // 完美
long *pl = const_cast<long *>(&i); // 错误，要求是同数据类型
volatile int k = 0;
int *pk = const_cast<int *>(&k); // 正确
```

## new，malloc区别

1）.malloc与free是C++/C语言的标准库函数，new/delete是C++的运算符。它们都可用于申请动态内存和释放内存。

2）.对于非内部数据类型的对象而言，光用maloc/free无法满足动态对象的要求。对象在创建的同时要自动执行构造函数，对象在消亡之前要自动执行析构函数。由于malloc/free是库函数而不是运算符，不在编译器控制权限之内，不能够把执行构造函数和析构函数的任务强加于malloc/free。

3）.因此C++语言需要一个能完成动态内存分配和初始化工作的运算符new，以一个能完成清理与释放内存工作的运算符delete。注意new/delete不是库函数。

4）.C++程序经常要调用C函数，而C程序只能用malloc/free管理动态内存

```c++
int** M = new int*[m];
for(int i=0; i<m; i++)
    M[i] = new int[n];
for(int i=0; i<n; i++)
    delete[] M[i];
delete[] M;

int** M = (int*)malloc(sizeof(int*)*m);
for(int i=0; i<m; i++)
    M[i] = (int)malloc(sizeof(int)*n);
for(int i=0; i<n; i++)
    free(M[i]);
free(M);

```



## C++中static关键字的作用

同时编译多个文件时,所有未加static前缀的全局变量和函数都具有全局可见性,所以加了static关键字的变量和函数可对其它源文件隐藏。

#####静态局部变量

还可以保持变量内容的持久性。通常，在函数体内定义了一个变量，每当程序运行到该语句时都会给该局部变量分配栈内存。但随着程序退出函数体，系统就会收回栈内存，局部变量也相应失效。但有时候我们需要在两次调用之间对变量的值进行保存。通常的想法是定义一个全局变量来实现。但这样一来，变量已经不再属于函数本身了，不再仅受函数的控制，给程序的维护带来不便。静态局部变量正好可以解决这个问题。静态局部变量保存在全局数据区，而不是保存在栈中，每次的值保持到下一次调用，直到下次赋新值。

用static前缀作为关键字的变量默认的初始值为0。

#####静态函数

在函数的返回类型前加上static关键字,函数即被定义为静态函数。静态函数与普通函数不同，它只能在声明它的文件当中可见，不能被其它文件使用。

定义静态函数的好处： 

• 静态函数不能被其它文件所用； 

• 其它文件中可以定义相同名字的函数，不会发生冲突；

##### 面向对象的static关键字（类中的static关键字）

**静态数据成员有以下特点：**

无论这个类的对象被定义了多少个，**静态数据成员在程序中也只有一份拷贝，由该类型的所有对象共享访问。**

静态数据成员存储在全局数据区。静态数据成员定义时要分配空间，所以不能在类声明中定义。

**关于静态成员函数，可以总结为以下几点：**

- 出现在类体外的函数定义不能指定关键字static；
- 静态成员之间可以相互访问，包括静态成员函数访问静态数据成员和访问静态成员函数；
- 非静态成员函数可以任意地访问静态成员函数和静态数据成员；
- 静态成员函数不能访问非静态成员函数和非静态数据成员；
- 由于没有this指针的额外开销，因此静态成员函数与类的全局函数相比速度上会有少许的增长；