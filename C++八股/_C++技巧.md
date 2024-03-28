
## inline

- inline函数像是宏，但却多了类型检查
- 在类声明中定义的函数，除了虚函数都会自动隐式被当作内联函数
- **定义在类声明之中的成员函数将自动地成为内联函数**
- 编译器会为所用 inline 函数中的局部变量分配内存空间
- 会将 inline 函数的的输入参数和返回值映射到调用方法的局部变量空间中
- 虚函数是运行时概念，内联是编译时发生的，所以虚函数不能内联
- **只有在编译器具有实际对象而不是对象的指针或引用时才可以内联**
## explicit

- 仅含有一个参数或除了第一个参数都有默认值的构造函数隐含了**隐式转换**功能
- explicit禁止了隐式转换
```C++[]
int i=3;
double j = 3.1;
i+j;//i会被转换成double类型，然后才做加法运算。

class A{};
class B: public A
{};//B是子类
void Fun(A& a);
B b;
Fun(b);//使用子类对象代替父类对象是可以的，也是因为隐式类型转换。

class Test {
public:
	Test(int i);
};
Test t1 = 1;//正确，由于强制类型转换，1先被Test构造函数构造成Test对象，然后才被赋值给t1
Test t2(1);//正确
```

## 友元

- **友元函数**能够使函数直接访问类的保护数据和私有数据成员
- **友元类**的所有成员函数都是另一个类的**友元函数**
- 想要在结构体中重载运算符可以这样写
```C++[]
struct Person {
    std::string name;
    int age;
    // 构造函数
    Person(std::string _name, int _age) : name(_name), age(_age) {}
    // 重载 < 运算符
    // 这样重载时必须只有一个参数
    bool operator<(const Person& a) const {
        // 在这里定义你的比较规则，例如按年龄升序排序
        return age > other.age;
    }
    // 或者
    friend bool operator<(Person a, Person b) {
        return a.age < b.age;
    }
};
```


# 四种强制类型转换


## static_cast

- 用于基本数据类型之间的转换
- void\*和其他类型指针的转换
- 子类对象的指针转换成父类对象的指针
- **上面三个都是隐式转换，虽然安全但最好都用static_cast代替**
- **static_cast不能转换掉expression的const、volitale、或者__unaligned属性**

## dynamic_cast

- 只用于对象的指针和引用，主要用于执行**安全的向下转型**
- **在运行时处理**
- 唯一可能有重大运行时代价的强制转型
- 对于NULL指针还返回NULL
- 需要父类拥有虚函数，否则无法获取运行时类型信息RTTI，会报错
- 如果类型不兼容，则指针类型会返回NULL，引用类型会抛出std::bad_cast异常

## const_cast

- 用来移除const或volatile属性，C语言并没有此类机制
- 一般用于消除对象的常量性质，但不能消除变量的常量性质
- 消除对象必须是指针或引用
- 常量指针、引用被转换为非常量指针、引用，并且仍然指向原来的对象
- 常量对象被转换成非常量对象

## reinterpret_cast

- 能够在非相关的类型之间转换
- 只是简单的从一个指针到别的指针的值的二进制拷贝


# C++11特性

## auto

- 自动类型推导
- 当变量**不是指针或者引用类型**时，推导的结果中**不会保留**const、volatile关键字
- 当变量**是指针或者引用类型**时，推导的结果中会**保留**const、volatile关键字
- **不能**推导函数参数，因为调用才会赋值，不赋值就不能推导
- **不能**用于类的非静态成员初始化
- **不能**定义数组
- **不能**推导模板参数
- auto使用的是**模板实参推断**的机制
```C++[]
template <typename T>
struct Test{};
int func()
{
    Test<double> t;
    Test<auto> t1 = t;           // error, 无法推导出模板类型
    return 0;
}
```
```C++[]
template <class A>
void func(void)
{
    auto val = A::get(); // 使用auto推断模板类的函数返回值
    cout << "val: " << val << endl;
}
int main()
{
    func<T1>();
    func<T2>();
    return 0;
}


// 否则就要
template <class A, typename B>        
void func(void)
{
    B val = A::get();
    cout << "val: " << val << endl;
}
int main()
{
    func<T1, int>();                  // 手动指定返回值类型 -> int
    func<T2, string>();               // 手动指定返回值类型 -> string
    return 0;
}
```

## decltype

- 用于在编译时推导一个表达式的值
```C++[]
int a = 10;
decltype(a) b = 99;                 // b -> int
decltype(a+3.14) c = 52.13;         // c -> double
decltype(a+b*c) d = 520.1314;       // d -> double
```
- 表达式可简单可复杂，auto做不到
- 表达式为**普通变量、普通表达式或者类表达式**，推导出的类型和表达式的类型一致
- 表达式是**函数调用**，推导出的类型和函数返回值一致
- **对于纯右值，只有类类型可以携带const、volatile限定符，除此之外需要忽略掉这两个限定符**
- 表达式是**左值**，或者**被括号( )包围**，推导出的是表达式类型的引用（不会忽略const、volatile）

## 返回类型后置

```C++[]
int& test(int &i) { return i; }
double test(double &d)
{
    d = d + 100;
    return d;
}
template <typename T>
// 返回类型后置语法
auto myFunc(T& t) -> decltype(test(t))
{
    return test(t);
}
int main()
{
    int x = 520;
    double y = 13.14;
    // auto z = myFunc<int>(x);
    auto z = myFunc(x);             // 简化之后的写法
    cout << "z: " << z << endl;
    // auto z = myFunc<double>(y);
    auto z1 = myFunc(y);            // 简化之后的写法
    cout << "z1: " << z1 << endl;
    return 0;
}
```
- 怪东西

## 智能指针

- 智能指针能解决内存泄漏、多线程下析构问题

## auto_ptr

- 不好用

## shared_ptr

- 内部有一个共享引用计数器
- 每当复制一个指向相同对象的shared_ptr时便会令引用计数+1
- 一个shared_ptr离开作用域会令引用计数-1
- 引用计数应该是一块堆内存，而不是static变量
- 由资源指针和一个控制块指针组成
- 控制块存deleter、allocator、引用计数、同一指向的weak_ptr等
- **同一个shared_ptr被多线程读——安全**
- **同一个shared_ptr被多线程写——不安全**
- **共享计数的多个shared_ptr被多线程写——安全，因为引用计数的改变是原子的**
- 多线程中如果修改shared_ptr指向，则有被提前析构而出错的风险
- get()直接返回所管理的指针，不要用
- reset函数，释放并销毁原生指针。如果参数为一个新指针，将管理这个新指针（旧的引用-1）
- make_shared函数，可一次性分配存储管理对象和控制块的堆内存
- new创建会分两次分配对象内存和控制块内存，比make_shared慢
- swap交换两个shared_ptr所管理的对象
- shared_from_this使得类可以获得管理自己的shared_ptr
```C++[]
#include <iostream>
#include <memory>
class MyClass : public std::enable_shared_from_this<MyClass> {
public:
    std::shared_ptr<MyClass> getShared() {
        return shared_from_this();
    }
};
int main() {
    MyClass *obj = new MyClass;
    // std::shared_ptr<MyClass> ptr = std::make_shared<MyClass>(42);
    std::shared_ptr<MyClass> ptr(obj);
    std::shared_ptr<MyClass> ptr2 = ptr->getShared();
    std::shared_ptr<MyClass> ptr3(obj); // 不要这样使用
    std::cout << "ptr.use_count(): " << ptr.use_count() << std::endl;
    std::cout << "ptr2.use_count(): " << ptr2.use_count() << std::endl;
    std::cout << "ptr3.use_count(): " << ptr3.use_count() << std::endl;
    return 0;
}
// 输出2 2 1
```

## weak_ptr

- 不能直接访问所管理的资源指针内部的东西
- 若要访问则需要先用lock()函数将其转换为shared_ptr
- 为了解决shared_ptr的循环引用问题问题
```C++[]
struct A{
    shared_ptr<B> b;
};
struct B{
    weak_ptr<A> a;
};
shared_ptr<A> pa = make_shared<A>();
shared_ptr<B> pb = make_shared<B>();
pa->b = pb;
pb->a = pa;
```
- weak_ptr不增加引用计数，只是观测
- expired()函数判断所管理的指针是否被释放
- use_count()函数返回原生指针的引用计数
- lock()返回shared_ptr，若原生指针被释放则返回一个空的shared_ptr
- reset()将本身置空

## unique_ptr

- 拥有对持有对象的唯一所有权
- 两个unique_ptr不能指向同一对象
	- 也即不能被复制，只能转移所有权
```C++[]
std::unique_ptr<A> a1(new A());
std::unique_ptr<A> a2 = a1;//编译报错，不允许复制
std::unique_ptr<A> a3 = std::move(a1);//可以转移所有权，所有权转义后a1不再拥有任何指针
```
- get()获取其原生指针，尽量不要使用
- release()释放所管理指针的所有权，返回原生指针，但并不销毁原生指针
- reset()释放并销毁原生指针，如果参数为一个新指针，将管理这个新指针

## lambda表达式

- 具体语法体现为
```
[capture](params) opt -> ret {body;};
```

- 捕获列表有以下几种捕获方式
1. \[] - 表示不捕捉任何变量
2. \[\&] - 捕获外部作用域中所有变量，并作为引用在函数体内使用 (按引用捕获)
3. \[=] - 捕获外部作用域中所有变量，并作为副本在函数体内使用 (按值捕获)
4. \[=, &foo] - 按值捕获外部作用域中所有变量，并按照引用捕获外部变量 foo
5. \[this] - 捕获当前类中的 this 指针，让 lambda 表达式拥有和当前类成员函数同样的访问权限
```C++[]
void output(int x, int y)
{
    auto x1 = [] {return m_number; };                      // error
    auto x2 = [=] {return m_number + x + y; };             // ok
    auto x3 = [&] {return m_number + x + y; };             // ok
    auto x4 = [this] {return m_number; };                  // ok
    auto x5 = [this] {return m_number + x + y; };          // error
    auto x6 = [this, x, y] {return m_number + x + y; };    // ok
    auto x7 = [this] {return m_number++; };                // ok
}
```

- params参数列表和普通函数的参数列表一样
- opt选项，可以省略
1. mutable表达式内的代码可以修改被捕获的变量，并且可以访问被捕获的对象的non-const方法
2. exception说明lambda表达式是否抛出异常以及何种异常
3. attribute用来声明属性
- C++11 中允许省略 lambda 表达式的返回值

## NULL和nullptr

- NULL无类型，nullptr是nullptr_t类型，代表空指针常量
```C++[]
#ifdef __cplusplus  
#define NULL    0  
#else  
#define NULL    ((void *)0)  
#endif  
```
- 避免了以void\*为参数和int为参数的函数重载时的调用问题
```C++[]
void f(void*){}
void f(int){}
int main()
{
    f(NULL); // what function will be called?
}
```
- 用nullptr做参数会正确调用void\*版本

- **nullptr，可以保证在任何情况下都代表空指针**

## 成员初始化列表

- A类成员有另一个B类对象时，初始化A类对象会先调用B类的默认构造函数
```C++[]
class temp {
public:
    int x;
    temp() {
        std::cout << "Create temp" << std::endl;
    }
};
class A {
    temp x;
};
int main() { A y;} // 会输出一次Create temp
```
- 必须使用成员初始化列表的情况
1. const类型的成员变量
2. 引用类型的成员变量
3. 带有引用的类类型的成员变量
4. 存在继承关系，子类必须在初始化列表中调用父类构造函数
5. 没有默认构造函数的累类型的成员变量

- 成员初始化顺序与成员的声明顺序相同，与在初始化列表中的先后顺序无关
- 初始化列表先于构造函数体执行
- 对于类类型的成员变量，由于初始化列表会直接调用拷贝构造函数，少创建了一个对象


## 右值引用
## 范围for循环
## 可变参数模板

```C++[]
template <class... T>
void f(T... args);
```
- 参数包args可以包含任意数量的模板参数
- 可通过递归或者逗号分隔的方式将模板参数一个一个拆出来

## 原子操作

- 比互斥对象的实现更高效
- atomic_bool、atomic_int等原子操作数据类型

## std::function和std::bind

- 几种可调用对象
1. 函数
2. 函数指针
3. lambda表达式
4. bind对象
5. 函数对象：重载了()也即调用运算符的类，其对象可以直接作为函数名使用

- std::function是一个可调用对象包装器，是一个类模板
- 可以用统一的方式处理函数、函数对象、函数指针，lambda表达式并允许保存和延迟它们的执行
- std::bind可以将可调用对象和参数一起绑定
```C++[]
//表示绑定函数 fun 的第一，二，三个参数值为： 1 2 3
std::function<void(int, int, int)> f1 = std::bind(fun_1, 1, 2, 3); 				
f1(); 	//print: x=1,y=2,z=3
//表示绑定函数 fun 的第三个参数为 3，而fun 的第一，二个参数分别由调用 f2 的第一，二个参数指定
std::function<void(int, int, int)> f2 = std::bind(fun_1, std::placeholders::_1, std::placeholders::_2, 3);
f2(1, 2);		//print: x=1,y=2,z=3
//表示绑定函数 fun 的第三个参数为 3，而fun 的第一，二个参数分别由调用 f3 的第二，一个参数指定
std::function<void(int, int, int)> f3 = std::bind(fun_1, std::placeholders::_2, std::placeholders::_1, 3);
//注意： f2  和  f3 的区别。
f3(1, 2);		//print: x=2,y=1,z=3
```

## C++11关键字

- **noexcept**告诉编译器这个函数不会抛出异常
- C++11之前要写throw()，括号中如果没有写某个类型则不能抛出对应类型的异常
- **override**重写父类方法，参数、返回类型必须相同
- **final**写在类名后面，表示类不允许被继承
- **=default** 默认生成构造、析构、拷贝、=几类函数的实现
- **=delete**禁止编译器自动生成这些函数
- **using**可作为升级版的typedef
```C++[]
//声明命名空间
using namespace std;
//using 新的类型 = 旧的类型; 可读性与typedef相差无几
using ll = long long;
//using 定义函数指针func_ptr，凸显using的可读性
using func_ptr = int(*)(int, double);//返回值是int，两个参数int和double
//模板的别名
template<typename T>
using mymap = map<int, T>;
```

## sizeof

```C++[]
//sizeof = 8
struct str1{
    char p;
    int a;
    int b[0];
};
//sizeof = 0 ?
struct str2{
    int b[0];
};
//sizeof = 1
struct str3{
};
//sizeof = 1
class cls{
};
```
- 在编译器就计算完成，所以不能用于计算动态分配的内存空间
```C++[]
int *a = new int[10];
cout << sizeof(a) << endl; // 输出8而不是40
delete[] a;
```
- 空的类或结构体占1字节，往里添加一个函数（不论是构造析构还是普通函数）不会改变大小
- 如果添加虚函数，则由于虚表指针的存在会变成8

## volatile

- 用来修饰一些因为程序不可控因素导致变化的变量
- 比如访问底层硬件设备的变量，以提醒编译器不要对该变量的访问擅自进行优化

## extern

- extern "C"会指示编译器这部分代码按C语言的标准进行编译
- 主要是因为C++和C的代码编译过后在目标代码中的命名规则不同
- 标有extern所声明的变量或函数是在其他文件中定义的，而不是在当前文件中定义的

## C++程序优化的方法

- 经常读取的资源缓存在内存中
- 减少大内存对象的构造与析构
- 使用右值语义以减少临时对象的构造
- 可以内联，减少继承层数
- 优化循环条件
- 能用原子操作就不用锁，能应用层同步的就不用内核对象同步
- 如果有内存频繁的申请与释放，可以考虑**内存池**
- 优化线程的使用，节省系统资源与切换造成的性能损耗，线程使用频繁的可以考虑**线程池**
- 尽量使用事件通知，谨慎使用轮循或者sleep函数

## C++11 RVO/NRVO机制

- 编译器优化返回值是一个类对象的情况
- 优化为传入一个引用，将值直接放进这个引用里
- 不优化的版本会有两次拷贝构造函数，两次析构函数的成本
- 可以少一次拷贝构造函数和一次析构函数

## C++代码编译到可执行文件过程

- 预处理
- 编译
	- 词法分析
	- 语法分析
	- 语义分析
	- 符号汇总
- 汇编
- 链接

## 静态库和动态库

- 静态库在**链接阶段，会将汇编生成的目标文件\*\*\*.o与引用到的库一起链接打包到可执行文件中**
- 静态链接在编译时期完成，移植方便，浪费空间资源

- 静态库变一点，引用者全要重新编译，全量更新
- 动态库可以实现增量更新
- 动态库可以把一些库函数的链接推迟到运行时进行
- **可以实现进程间资源共享**

## 模板类和模板函数的区别

- 模板函数的实例化是由编译程序在处理函数调用时自动完成的
- 类模板的实例化必须由程序员在程序中显式地指定
- 使用时类模板必须加`<T>`

## mutable

- lambda表达式中表示按值传递的值可以修改，只不过改了外面也不变
- 可以修饰类中非静态成员变量
- 表示一个const实例中有可以修改的部分

## STL

- **包含6大部件：容器、迭代器、算法、仿函数、适配器和空间配置器**

