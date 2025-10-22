# 前言
经典书籍 Effrctive C++ 学习笔记。
# 第一章：让自己习惯 C++
## 条款 1：视 C++ 为一个语言联邦
C++ 作为 C 语言的 plus 版本，兼容 C 的同时，引入了面对对象思想和泛型编程，由以下四部分组成：
1. 传统的面向过程的 C 语言：区块、语句、预处理器、内置数据类型、数组和指针。  
2. 面向对象的 C with Classes：类、封装、继承、多态和动态绑定。  
3. 模板编程 Template C++ 和模板元编程（TMP）。  
4. C++ 标准库 STL。  
   
注：不同模块，效率优化策略不同
## 条款 2：尽量以 const, enum, inline 替换 #define
> 宁可以编译器替换预处理器

在预处理时 #define 修饰的符合会被替代为某个具体数值及常量，遇到编译错误时难以纠正。
### 1. 应当用 constexpr 定义编译期常量来替代大部分的 #define 宏常量定义  

注：原文中使用 const 替代定义常量，
```
#define ASPECT_RATIO 1.653 // X

constexpr auto aspect_ratio = 1.653;

class Player{
    private:
        static const int numTurns = 5; // 静态成员变量，被类的对象访问
}
```
### 2. enum 可以用于替代整型的常量

存在编译器在 static 成员变量在声明时不允许赋值
```
class GamePlayer {
public:
    enum { numTurns = 5 }; // numTurns 实际成为 5 的一个记号
};
```
### 3. 大部分 #define 宏常量应当用内联模板函数替代
```
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))  // X

template<typename T>
inline void CallWithMax(const T& a, const T& b) {
    f(a > b ? a : b);
}
```
注：宏和函数的行为本身并不完全一致，宏只是简单的替换，并不涉及传参和复制。

## 条款 3：尽可能使用 const
设置常量属性为只读！ 
### 1. const 修饰 变量，指针，函数，函数返回值等，可以使程序减少错误，或者更容易检测错误  
   
- 指针常量：`int* const p;//指针地址不可变，指针指向值可变`
- 常量指针：`const int* p;//指针指向值不可变，指针地址可变`
- 常量指针常量：`const int* const p;//都不可变`
### 2. const 修饰迭代器
- iterator 相当于 `T* const //指针常量`
- const_iterator 相当于` const T* //常量指针`

### 3. const 修饰返回值： 面对函数声明时，避免函数的结果被无意义地当作左值 
### 4. const成员函数：操控const对象
- const 成员函数和普通成员函数可以发生重载。 
- 当const对象中的某些成员变量应当是允许被改变的，使用关键字mutable来标记这些成员变量。  
- 在重载 const 和 non-const 成员函数时，使用转型进行常量性转除，避免书写重复的内容：  
```
class TextBlock {
public:
    const char& operator[](std::size_t position) const {

        // 假设这里有非常多的代码

        return text[position];
    }

    char& operator[](std::size_t position) {
        return const_cast<char&>(static_cast<const TextBlock&>(*this)[position]);
    }

private:
    std::string text;
};
```
## 条款 4：确定对象在使用前已被初始化
1. 直接在定义处赋初值
2. 使用构造函数成员初始化列表  

注：
- 类中成员的初始化具有次序性。次序与成员变量的声明次序一致，与成员初始化列表的次序无关。    
- 类中成员的初始化是可选的，但是引用类型必须初始化。  
1. C++ 对于定义于不同编译单元内的全局静态对象的初始化相对次序并无明确定义  
 Meyers' singleton：利用一个函数，定义并初始化本 static 对象，并返回它的引用。
```
FileSystem& tfs() {
    static FileSystem fs;
    return fs;
}

Directory& tempDir() {
    static Directory td;
    return td;
}
```
注：C++ 保证，函数内的局部静态对象会在该函数被调用期间和首次遇上该对象之定义式时被初始化。  
# 第二章：构造/析构/赋值运算  
##条款 5：了解 C++ 默默编写并调用哪些函数  
编译器自动生成如下函数：
```
// 定义
class Empty {
public:
    Empty() { ... }                           // 默认构造函数（没有任何构造函数时）
    Empty(const Empty&) { ... }               // 拷贝构造函数
    Empty(Empty&&) { ... }                    // 移动构造函数 (since C++11)
    ~Empty() { ... }                          // 析构函数
    
    Empty& operator=(const Empty&) { ... }    // 拷贝赋值运算符
    Empty& operator=(Empty&&) { ... }         // 移动赋值运算符 (since C++11)
};
// 调用
Empty e1;                   // 默认构造函数 & 析构函数
Empty e2(e1);               // 拷贝构造函数
Empty e3 = std::move(e2);   // 移动构造函数 (since C++11)
e2 = e1;                    // 拷贝赋值运算符
e3 = std::move(e1);         // 移动赋值运算符 (since C++11)
```
注：拷贝赋值运算符只有在允许存在时才会自动创建
- 存在引用类型时，引用无法指向不同对象  
- 类中含有 const 成员。
- 基类中含有 private 的拷贝赋值运算符。
## 条款 6：若不想使用编译器自动生成的函数，就该明确拒绝
- 将不想使用的函数声明为 private
- delete 关键字
## 条款 7：为多态基类声明虚析构函数
当派生类对象经由一个基类指针被删除，而该基类指针带着一个非虚析构函数，其结果是未定义的，可能会无法完全销毁派生类的成员，造成内存泄漏。消除这个问题的方法就是对基类使用虚析构函数。
```
class Base {
public:
    Base();
    virtual ~Base();
};
```
- 只要基类的析构函数是虚函数，那么派生类的析构函数不论是否用virtual关键字声明，都自动成为虚析构函数。  
- 虚析构函数的运作方式是，最深层派生的那个类的析构函数最先被调用，然后是其上的基类的析构函数被依次调用。      
- 如果你想将基类作为抽象类使用，但手头上又没有别的虚函数，那么将它的析构函数设为纯虚函数是一个不错的想法。
注:即使是纯虚析构函数，也需要一个函数体
```
Base::~Base() {}

class Base {
public:
    virtual ~Base() = 0 {}
};
```
## 条款 8：别让异常逃离析构函数
1. 杀死程序
```
DBConn::~DBConn() {
    try { db.close(); }
    catch (...) {
        // 记录运行日志，以便调试
        std::abort();
    }
}
```
2. 直接吞下异常不做处理  
3. 重新设计接口，将异常的处理交给客户端完成
```
class DBConn {
public:
    ...
    void close() {
        db.close();
        closed = true;
    }

    ~DBConn() {
        if (!closed) {
            try {
                db.close();
            }
            catch(...) {
                // 处理异常
            }
        }
    }

private:
    DBConnection db;
    bool closed;
};
```  
## 条款 9：绝不在构造和析构过程中调用虚函数  
在创建派生类对象时，基类的构造函数永远会早于派生类的构造函数被调用，而基类的析构函数永远会晚于派生类的析构函数被调用。  
在派生类对象的基类构造和析构期间，对象的类型是基类而非派生类，因此此时调用虚函数会被编译器解析至基类的虚函数版本。  
```
class Transaction {
public:
    Transaction() { Init(); }
    virtual void LogTransaction() const = 0;

private:
    void Init(){
        ...
        LogTransaction();      // 此处间接调用了虚函数！
    }
};
```
如果想要基类在构造时就得知派生类的构造信息，推荐的做法是在派生类的构造函数中将必要的信息向上传递给基类的构造函数：  
```
class Transaction {
public:
    explicit Transaction(const std::string& logInfo);
    void LogTransaction(const std::string& logInfo) const;
    ...
};

Transaction::Transaction(const std::string& logInfo) {
    LogTransaction(logInfo);                           // 更改为了非虚函数调用
}

class BuyTransaction : public Transaction {
public:
    BuyTransaction(...)
        : Transaction(CreateLogString(...)) { ... }    // 将信息传递给基类构造函数
    ...

private:
    static std::string CreateLogString(...);
}
```
注: CreateLogString 是一个静态成员函数，静态成员函数可以确保不会使用未完成初始化的成员变量。    
## 条款 10：令 operator= 返回一个指向 *this 的引用  
实现连锁赋值
```
class Widget {
public:
    Widget& operator+=(const Widget& rhs) {    // 这个条款适用于
        ...                                    // +=, -=, *= 等等运算符
        return *this;
    }
    Widget& operator=(int rhs) {               // 即使参数类型不是 Widget& 也适用
        ...
        return *this;
    }
};
```
## 条款 11：在 operator= 中处理“自我赋值”
在一些情况下自我赋值可能会导致意外的错误，例：   
```
Widget& operator+=(const Widget& rhs) {
    delete pRes;                          // 删除当前持有的资源
    pRes = new Resource(*rhs.pRes);       // 复制传入的资源
    return *this;
}
```
1. 在执行后续语句前先进行证同测试（Identity test）
```
Widget& operator=(const Widget& rhs) {
    if (this == &rhs) return *this;        // 若是自我赋值，则不做任何事

    delete pRes;
    pRes = new Resource(*rhs.pRes);
    return *this;
}
```
2. 只关注异常安全性，而不关注是否自我赋值
```
Widget& operator=(const Widget& rhs) {
    Resource* pOrigin = pRes;             // 先记住原来的pRes指针
    pRes = new Resource(*rhs.pRes);       // 复制传入的资源
    delete pOrigin;                       // 删除原来的资源
    return *this;
}
```
3. 使用 copy and swap 技术，这种技术聪明地利用了栈空间会自动释放的特性，这样就可以通过析构函数来实现资源的释放 
```
Widget& operator=(const Widget& rhs) {
    Widget temp(rhs);
    std::swap(*this, temp);
    return *this;
}

Widget& operator=(Widget rhs) {
    std::swap(*this, rhs);
    return *this;
}
```
## 条款 12：复制对象时勿忘其每一个成分
字面意思，当你决定手动实现拷贝构造函数或拷贝赋值运算符时，忘记复制任何一个成员都可能会导致意外的错误。  
- 复制类中所有的局部变量；
- 派生类中要调用基类中相应的函数或操作符。  

注：不要尝试在拷贝构造函数中调用拷贝赋值运算符，或在拷贝赋值运算符的实现中调用拷贝构造函数，一个在初始化时，一个在初始化后，它们的功用是不同的。  
# 第三章：资源管理
## 条款 13：以对象管理资源  
申请资源 -> 使用资源 -> 释放资源
- 使用成对的 new 和 delete。  
- RAII，构造函数负责申请资源，析构函数负责资源的释放。  

例：在 C++11 中，通过专一所有权来管理RAII对象可以使用`std::unique_ptr`，通过引用计数来管理RAII对象可以使用`std::shared_ptr`。

## 条款 14：在资源管理类中小心拷贝行为
1. 禁止复制
  
许多时候允许 RAII 对象被复制并不合理，如果确是如此，那么就该明确禁止复制行为。  
2. “引用计数法”  

每一次复制对象就使引用计数 +1，每一个对象离开定义域就调用析构函数使引用计数 -1，直到引用计数为0就彻底销毁资源。  
3. 复制底层资源  
   
在复制对象的同时复制底层资源的行为又被称作深拷贝（Deep copying） 
4. 转移底层资源的所有权  
   
永远保持只有一个对象拥有对资源的管理权，当需要复制对象时转移资源的管理权。  
## 条款 15：在资源管理类中提供对原始资源的访问
管理类存放的是资源的指针，我们无法从管理类直接得到一个资源对象（只能得到一个指针，通过指针找到对象）。所以我们最好用显式转化或者隐式转换（自动类型转换）来得到一个资源对象。
STL 中的智能指针提供了对原始资源的隐式访问和显式访问：
```
Investment* pRaw = pSharedInv.get();    // 显式访问原始资源
Investment raw = *pSharedInv;           // 隐式访问原始资源
```
同理可得：
```
class Font {
public:
    FontHandle Get() const { return handle; }       // 显式转换函数
    operator FontHandle() const { return handle; }  // 隐式转换函数

private:
    FontHandle handle;
};
```
一般而言显式转换比较安全，但隐式转换对客户比较方便。  
## 条款 16：成对使用 new 和 delete 时要采用相同形式
- 使用 new 来分配单一对象，使用 new[] 来分配对象数组  
必须明确它们的行为并不一致，分配对象数组时会额外在内存中记录“数组大小”。
- 使用 delete[] 会根据记录的数组大小多次调用析构函数，使用 delete 则仅仅只会调用一次析构函数
```
int* array = new int[10];
int* object = new int;

delete[] array;
delete object;
```
## 条款 17：以独立语句将 newed 对象置入智能指针
使用std::make_unique和std::make_shared替代
```
auto pUniqueInv = std::make_unique<Investment>();    // since C++14
auto pSharedInv = std::make_shared<Investment>();    // since C++11
```
# 第四章：设计与声明
## 条款 18：让接口容易被正确使用，不易被误用 
1. 保证参数一致性  
2. 保持行为一致性及相同函数功能一致
3. 阻止误用  
   
“阻止误用”的办法包括建立新类型、限制类型上的操作、束缚对象值以及消除客户的资源管理责任。  
```
// 三个参数类型相同的函数容易造成误用
Data::Data(int month, int day, int year) { ... }

// 通过适当定义新的类型加以限制，降低误用的可能性
Data::Data(const Month& m, const Day& d, const Year& y) { ... }
```
尽量使用智能指针，避免跨 DLL 的 new 和 delete，使用智能指针自定义删除器来解除互斥锁（mutexes）。  
## 条款 19：设计 class 犹如设计 type 
1. 新 type 对象应该如何被创建和销毁？ 
     
这会影响到类中构造函数、析构函数、内存分配和释放函数（operator new，operator new[]，operator delete，operator delete[]）的设计。  

2. 对象的初始化和赋值该有什么样的差别？   

这会影响到构造函数和拷贝赋值运算之间行为的差异。  

3. 新 type 的对象如果被按值传递，意味着什么？  

这会影响到拷贝构造函数的实现。    

4. 什么是新 type 的合法值？  

你的类中的成员函数必须对类中成员变量的值进行检查，如果不合法就要尽快解决或明确地抛出异常。    

5. 你的新 type 需要配合某个继承图系吗？   

你的类是否受到基类设计地束缚，是否拥有该覆写地虚函数，是否允许被继承（若不想要被继承，应该声明为 final）。  

6. 什么样的运算符和函数对此新 type 而言是合理的？  

这会影响到你将为你的类声明哪些函数和重载哪些运算符。    

7. 什么样的标准函数应该被驳回？   

这会影响到你将哪些标准函数声明为= delete。   

8. 谁该取用新 type 的成员？  

这会影响到你将类中哪些成员设为 public，private 或 protected，也将影响到友元类和友元函数的设置。  

9. 什么是新 type 的“未声明接口”？  

为未声明接口提供效率、异常安全性以及资源运用上的保证，并在实现代码中加上相应的约束条件。  

10.  你的新 type 有多么一般化？  

如果你想要一系列新 type 家族，应该优先考虑模板类。   
## 条款 20：宁以 const 引用传参替换按值传参
按值传参时，程序会调用对象的拷贝构造函数构建一个在函数内作用的局部对象，开销较为昂贵.  
引用传递因为没有任何新对象被创建，不会调用任何构造函数或析构函数，所以效率比按值传参高得多。  
使用按引用传参也可以避免对象切片（Object slicing） 的问题。
```
class Window {
public:
    ...
    std::string GetName() const;
    virtual void Display() const;
};

class WindowWithScrollBars : public Window {
public:
    virtual void Display() const override;
};
此处一个WindowWithScrollBars类继承自Window基类。

void PrintNameAndDisplay(Window w) {    // 按值传参，会发生对象切片
    std::cout << w.GetName();
    w.Display();
}
```
对于内置类型、STL 的迭代器和函数对象，使用按值传参则更加高效。  
## 条款 21：必须返回对象时，别妄想返回其引用
返回引用存在指向一个不存在对象的风险。 
## 条款 22：将成员变量声明为 private
- 保证语法的一致性
- 精确控制对成员变量的处理
- 保证类的封装
## 条款 23：宁以非成员、非友元函数替换成员函数
虽然成员函数和非成员函数都可以完成我们的目标，使用非成员函数。
> 越少的代码可以访问数据，数据的封装性就越强

## 条款 24：若所有参数皆需类型转换，请为此采用非成员函数
```
class Rational {
public:
    Rational(int numerator = 0, int denominator = 1);
    const Rational operator*(const Rational& rhs) const;
};

Rational oneEight(1, 8);
Rational oneHalf(1, 2);
Rational result = oneHalf / oneEight;

result = oneHalf * 2;    // 正确
result = 2 * oneHalf;    // 报错

result = oneHalf.operator*(2);    // 正确
result = 2.operator*(oneHalf);    // 报错
```
在调用 operator* 时，int类型的变量会隐式转换为Rational对象，因此用 Rational 对象乘以 int 对象是合法的，但反过来则不是如此。  
所以，为了避免这个错误，我们应当将运算符重载放在类外，作为非成员函数：  
```
const Rational operator*(const Rational& lhs, const Rational& rhs);
```

## 条款 25：考虑写出一个不抛异常的swap函数
std::swap 函数在 C++11 后改为了用 std::move 实现
使用模板全特化为自定义类型实现自己的 swap 方法
```
class Widget {
public:
    void swap(Widget& other) {
        using std::swap;
        swap(pImpl, other.pImpl);
    }
    ...

private:
    WidgetImpl* pImpl;
};

namespace std {
    template<>
    void swap<Widget>(Widget& a, Widget& b) {
        a.swap(b);
    }
}
```
注意，由于外部函数并不能直接访问 Widget 的 private 成员变量，因此我们先是在类中定义了一个 public 成员函数，再由 std::swap 去调用这个成员函数。  
然而若 Widget 和 WidgetImpl 是类模板，情况就没有这么简单了，因为 C++ 不支持函数模板偏特化，所以只能使用重载的方式：
```
namespace std {
    template<typename T>
    void swap(Widget<T>& a, Widget<T>& b) {
        a.swap(b);
    }
}
```
但很抱歉，这种做法是被 STL 禁止的，因为这是在试图向 STL 中添加新的内容，所以我们只能退而求其次，在其它命名空间中定义新的 swap 函数：
```
namespace WidgetStuff {
    ...
    template<typename T>
    class Widget { ... };
    ...
    template<typename T>3
    void swap(Widget<T>& a, Widget<T>& b) {
        a.swap(b);
    }
}
```
我们希望在对自定义对象进行操作时找到正确的 swap 函数重载版本，这时候如果再写成std::swap，就会强制使用 STL 中的 swap 函数，无法满足我们的需求，因此需要改写成：
```
using std::swap;
swap(obj1, obj2);
```
这样，C++ 名称查找法则能保证我们优先使用的是自定义的 swap 函数而非 STL 中的 swap 函数。      
C++ 名称查找法则：编译器会从使用名字的地方开始向上查找，由内向外查找各级作用域（命名空间）直到全局作用域（命名空间），找到同名的声明即停止，若最终没找到则报错。   
函数匹配优先级：普通函数 > 特化函数 > 模板函数
# 第五章：实现
## 条款 26：尽可能延后变量定义式出现的时间
- 避免不必要的变量定义，以及延后变量定义式直到你确实需要它。  
- “默认构造+赋值”效率低于“直接构造”。
```
// 效率低
std::string encrypted;
encrypted = password;

// 效率高
std::string encrypted(password);
```
- 循环中变量定义 A. 定义于循环外，在循环中赋值：
```
Widget w;
for (int i = 0; i < n; ++i) {
    w = 取决于 i 的某个值;
    ...
}
```
开销：1 个构造函数 + 1 个析构函数 + n 个赋值操作  
- 循环中变量定义 B. 定义于循环内：
```
for (int i = 0; i < n; ++i) {
    Widget w(取决于 i 的某个值);
    ...
}
```
开销：n 个构造函数 + n 个析构函数  
注： 做法A会将变量的作用域扩大，除非知道该变量的赋值成本比“构造+析构”成本低，或者对这段程序的效率要求非常高，否则建议使用做法B。  
## 条款 27：少做转型动作
类型转换会破坏类型系统，带来未知风险
类型转换：
- C风格 / 函数风格  
仅小括号位置不同
- C++风格 

C 式转型：
```
(T)expression
T(expression)
```
C++ 式转型：
```
const_cast<T>(expression)
dynamic_cast<T>(expression)
reinterpret_cast<T>(expression)
static_cast<T>(expression)
```
- `const_cast`用于常量性转除，这也是唯一一个有这个能力的 C++ 式转型。
- `dynamic_cast`用于安全地向下转型，这也是唯一一个 C 式转型无法代替的转型操作，它会执行对继承体系的检查，因此会带来额外的开销。只有拥有虚函数的基类指针能进行dynamic_cast。
- `reinterpret_cast`用于在任意两个类型间进行低级转型，执行该转型可能会带来风险，也可能不具备移植性。
- `static_cast`用于进行强制隐式转换，也是最常用的转型操作，可以将内置数据类型互相转换，也可以将 void* 和 typed 指针，基类指针和派生类指针互相转换。  

尽量在 C++ 程序中使用 C++ 式转型，因为 C++ 式转型操作功能更明确，可以避免不必要的错误。  
唯一使用 C 式转型的时机可能是在调用 explicit 构造函数时：
```
class Widget {
public:
    explicit Widget(int size);
    ...
};
void DoSomeWork(const Widget& w);
DoSomeWork(Widget(15));
// 等价于 DoSomeWork(static_cast<Widget>(15));
```
需要注意的是，转型并非什么都没有做，而是可能会更改数据的底层表述，或者为指针附加偏移值，这和具体平台有关，因此不要妄图去揣测转型后对象的具体布局方式。  
避免对 *this 进行转型：
```
class Window {
public:
    virtual void OnResize() { ... }
    ...
};

class SpecialWindow : public Window {
public:
    virtual void OnResize() {
        static_cast<Window>(*this).OnResize();
        ...
    }
    ...
};
```
这段代码试图通过转型 *this 来调用基类的虚函数，然而这是严重错误的，这样做会得到一个新的 Window 副本并在该副本上调用函数，而非在原本的对象上调用函数。  
正确的做法如下：
```
class SpecialWindow : public Window {
public:
    virtual void OnResize() {
        Window::OnResize();
        ...
    }
    ...
};
```
当你想知道一个基类指针是否指向一个派生类对象时，你需要用到 dynamic_cast，如果不满足，则会产生报错。但是对于继承体系的检查可能是非常慢的，所以在注重效率的程序中应当避免使用 dynamic_cast，改用 static_cast 或别的代替方法。  
## 条款 28：避免返回 handles 指向对象的内部成分
返回一个成员变量的副本：
```
Point UpperLeft() const { return pData->ulhc; }
Point LowerRight() const { return pData->lrhc; }
```
避免返回 handles（包括引用、指针、迭代器）指向对象内部。  
增加封装性，使得 const 成员函数的行为符合常量性，并将发生 “空悬句柄” 的可能性降到最低。  
## 条款 29：为“异常安全”而努力是值得的
异常安全函数提供以下三个保证之一：  
基本承诺： 如果异常被抛出，程序内的任何事物仍然保持在有效状态下，没有任何对象或数据结构会因此败坏，所有对象都处于一种内部前后一致的状态，然而程序的真实状态是不可知的，也就是说客户需要额外检查程序处于哪种状态并作出对应的处理。  
强烈保证： 如果异常被抛出，程序状态完全不改变，换句话说，程序会回复到“调用函数之前”的状态。  
不抛掷（nothrow）保证： 承诺绝不抛出异常，因为程序总是能完成原先承诺的功能。作用于内置类型身上的所有操作都提供 nothrow 保证。  
原书中实现 nothrow 的方法是 throw()，不过这套异常规范在 C++11 中已经被弃用，取而代之的是 noexcept 关键字：
```
int DoSomething() noexcept;
```
注：使用 noexcept 并不代表函数绝对不会抛出异常，而是在抛出异常时，将代表出现严重错误，会有意想不到的函数被调用（可以通过 set_unexpected 设置），接着程序会直接崩溃。  
当异常被抛出时，带有异常安全性的函数会：  
- 不泄漏任何资源。
- 不允许数据败坏。
考虑以下 PrettyMenu 的 ChangeBackground 函数：
```
class PrettyMenu {
public:
    ...
    void ChangeBackground(std::vector<uint8_t>& imgSrc);
    ...
private:
    Mutex mutex;        // 互斥锁
    Image* bgImage;     // 目前的背景图像
    int imageChanges;   // 背景图像被改变的次数
};

// X
void PrettyMenu::ChangeBackground(std::vector<uint8_t>& imgSrc) {
    lock(&mutex);
    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imgSrc);
    unlock(&mutex);
}

// Y
void PrettyMenu::ChangeBackground(std::vector<uint8_t>& imgSrc) {
    Lock m1(&mutex);
    bgImage.reset(std::make_shared<Image>(imgSrc));

    ++imageChanges;
}

// Y
struct PMImpl {
    std::shared_ptr<Image> bgImage;
    int imageChanges;
};

class PrettyMenu {
    ...
private:
    Mutex mutex;
    std::shared_ptr<PMImpl> pImpl;
};

void PrettyMenu::ChangeBackground(std::vector<uint8_t>& imgSrc) {
    Lock m1(&mutex);

    auto pNew = std::make_shared<PMImpl>(*pImpl);    // 获取副本
    pNew->bgImage.reset(std::make_shared<Image>(imgSrc));
    ++pNew->imageChanges;

    std::swap(pImpl, pNew);
}
```
注：函数提供的异常安全保证通常最高只等于其调用的各个函数中最弱级别的异常安全性。
## 条款 30：透彻了解 inlining 的里里外外
- 显式指定 inline 关键字
- 直接将成员函数的定义式写在类中
```
class Person {
public:
    ...
    int Age() const { return theAge; }  // 隐式声明为 inline
    ...
private:
    int theAge;
};
```
在 inline 诞生之初，它被当作是一种对编译器的优化建议，即将“对此函数的每一个调用”都以函数本体替换之。但在编译器的具体实现中，该行为完全被优化等级所控制，与函数是否内联无关。  
在现在的 C++ 标准中，inline 作为优化建议的含义已经被完全抛弃，取而代之的是“允许函数在不同编译单元中多重定义”，使得可以在头文件中直接给出函数的实现。  
在 C++17 中，引入了一个新的 inline 用法，使静态成员变量可以在类中直接定义：
```
class Person {
public:
    ...
private:
    static inline int theAge = 0;  // since C++17
};
```
一个声明为 inline 的函数是否进行 inlining，取决于编译器。
- 大部分编译器拒绝将太过复杂的函数(如带有递归或循环)进行内联；
- 编译器也会拒绝将虚函数内联，这是因为虚函数意味着在运行时才确定调用的函数，而内联发生在编译期；
- 如果程序要获取 inline 函数的地址，编译器通常会为此函数生成一个非内联的函数体；
- 编译器不会对通过函数指针调用的函数进行 inlining。
## 条款 31：将文件间的编译依存关系降至最低
接口与实现分离：用“声明的依存性”替换“定义的依存性”
- 尽可能使用对象引用或对象指针替换直接使用对象
- 用 class 声明式代替 class 定义式
- 为声明式和定义式提供不同的头文件

1. pimpl idiom（pointer to implemention）：将原来的一个类分割为两个类，一个只提供接口，另一个负责实现该接口，称作句柄类（handle class）：
```
// person.hpp 负责声明类

class PersonImpl;

class Person {
public:
    Person();
    void Print();
    ...
private:
    std::shared_ptr<PersonImpl> pImpl;
};

// person.cpp 负责实现类

class PersonImpl {
public:
    int data{ 0 };
};

Person::Person() {
    pImpl = std::make_shared<PersonImpl>();
}

void Person::Print() {
    std::cout << pImpl->data;
}
```
- 如果使用对象引用或对象指针可以完成任务，就不要使用对象本身
- 只靠一个类型声明式就定义出指向该类型的引用和指针；但定义某类型的对象，就需要用到该类型的定义式
- 尽量以类声明式替换类定义式
```
class Date;                     // 类的声明式
Date Today();
void ClearAppointments(Date d); // 此处并不需要得知类的定义

#include "datefwd.h"            // 这个头文件内声明 class Date
Date Today();
void ClearAppointments(Date d);
```
2. 将句柄类定义为抽象基类，称作接口类（interface class）：
```
class Person {
public:
    virtual ~Person() {}
    virtual void Print();

// 工厂模式
    static std::shared_ptr<Person> Create();
    ...
};

class RealPerson : public Person {
public:
    RealPerson(...) { ... }
    virtual ~RealPerson() {}
    void Print() override { ... }

private:
    int data{ 0 };
};

static std::shared_ptr<Person> Person::Create() {
    return std::make_shared<RealPerson>();
}
```
# 第六章：继承与面向对象设计
## 条款 32：确定你的public继承塑模出 is-a 关系
“public继承”意味着 is-a，所谓 is-a，就是指适用于基类身上的每一件事情一定也适用于继承类身上，可以认为每一个派生类对象也都是一个基类对象。即适用于基类的属性都要适用于派生类。  
- 企鹅是鸟，但企鹅不会飞

## 条款 33：避免遮掩继承而来的名称
注：编译器会优先查找派生类覆盖的作用域，再去查找基类的作用域，最后再查找全局作用域。
1. 使用using关键字：
```
class Derived : public Base {
public:
    using Base::mf;
};
```
2. 只想要单一版本（特别是在private继承时），可以考虑使用转发函数（forwarding function）：
```
class Base {
public:
    virtual void mf();
    virtual void mf(double);
};

class Derived : public Base {
public:
    virtual void mf() {
        Base::mf();
    }
};
```
## 条款 34：区分接口继承和实现继承
- 接口继承和实现继承不一样。在public继承下，派生类总是继承基类的接口。
- 声明一个纯虚函数的目的，是为了让派生类只继承函数接口。
- 声明简朴的非纯虚函数的目的，是让派生类继承该函数的接口和缺省实现。
- 声明非虚函数的目的，是为了令派生类继承函数的接口及一份强制性实现。
- 允许为纯虚函数提供具体实现。

1. 用非纯虚函数提供缺省的默认实现
```
class Airplane {
public:
    virtual void Fly() {
        // 缺省实现
    }
};

class Model : public Airplane { ... };
```
2. 使用纯虚函数并提供默认实现
```
class Airplane {
public:
    virtual void Fly() = 0;
protected:
    void DefaultFly() {
        // 缺省实现
    }
};

class Model : public Airplane { 
public:
    virtual void Fly() override {
        DefaultFly();
    }
};
```
上述写法可以替代为：
```
class Airplane {
public:
    virtual void Fly() = 0;
};

void Airplane::Fly() {
        // 缺省实现
}

class Model : public Airplane { 
public:
    virtual void Fly() override {
        Airplane::Fly();
    }
};
```
## 条款 35：考虑虚函数以外的其它选择
1. 通过NVI实现模板方法模式
非虚接口（non-virtual interface，NVI） 设计手法的核心就是用一个非虚函数作为 wrapper，将虚函数隐藏在封装之下  
解释：将所有的虚函数声明为private，然后通过一个public non-virtual函数来调用该virtual函数，这个public函数称为virtual函数的包装器(wrapper)。
```
class GameCharacter {
public:
    int HealthValue() const {
        ...    // 做一些前置工作
        int retVal = DoHealthValue();
        ...    // 做一些后置工作
        return retVal;
    }
    ...
private:
    virtual int DoHealthValue() const {
        ...    // 缺省算法
    }
};
```
- 在 wrapper 中做一些前置和后置工作，确保得以在一个虚函数被调用之前设定好适当场景，并在调用结束之后清理场景
- 允许派生类重新定义虚函数
- 在NVI手法中虚函数除了可以是private，也可以是protected

2. 由函数指针实现 Strategy 模式
```
class GameCharacter;
int DefaultHealthCalc(const GameCharacter&);        // 缺省算法
class GameCharacter {
public:
    using HealthCalcFunc = int(*)(const GameCharacter&);    // 定义函数指针类型
    explicit GameCharacter(HealthCalcFunc hcf = DefaultHealthCalc)
        : healthFunc(hcf) {}
    int HealthValue() const { return healthFunc(*this); }
    ...
private:
    HealthCalcFunc healthFunc;
};
```
弱化类的封装，引入友元或提供public访问函数。

3. 由 std::function 完成 Strategy 模式

std::function是 C++11 中引入的函数包装器，使用它能提供比函数指针更强的灵活度：
```
class GameCharacter;
int DefaultHealthCalc(const GameCharacter&);        // 缺省算法
class GameCharacter {
public:
    using HealthCalcFunc = std::function<int(const GameCharacter&)>;    // 定义函数包装器类型
    explicit GameCharacter(HealthCalcFunc hcf = DefaultHealthCalc)
        : healthFunc(hcf) {}
    int HealthValue() const { return healthFunc(*this); }
    ...
private:
    HealthCalcFunc healthFunc;
};

// 使用返回值不同的函数
short CalcHealth(const GameCharacter&);

GameCharacter chara1(CalcHealth);

// 使用函数对象（仿函数）
struct HealthCalculator {
    int operator()(const GameCharacter&) const { ... }
};

GameCharacter chara2(HealthCalculator());

// 使用某个成员函数
class GameLevel {
public:
    float Health(const GameCharacter&) const;
    ...
};

GameLevel currentLevel;
GameCharacter chara2(std::bind(&GameLevel::Health, currentLevel, std::placeholders::_1));
```
4. 古典的 Strategy 模式：
并非直接利用函数指针（或包装器）调用函数，而是内含一个指针指向来自HealthCalcFunc继承体系的对象
```
class GameCharacter;

class HealthCalcFunc {
public:
    virtual int Calc(const GameCharacter& gc) const { ... }
    ...
};

HealthCalcFunc defaultHealthCalc;

class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc)
        : pHealthCalc(phcf) {}
    int HealthValue() const { return pHealthCalc->Calc(*this); }
    ...
private:
    HealthCalcFunc* pHealthCalc;
};
```
## 条款 36：绝不重新定义继承而来的非虚函数
非虚函数和虚函数具有本质上的不同：非虚函数执行的是静态绑定（statically bound，又称前期绑定，early binding），由对象类型本身（称之静态类型）决定要调用的函数；  
而虚函数执行的是动态绑定（dynamically bound，又称后期绑定，late binding），决定因素不在对象本身，而在于“指向该对象之指针”当初的声明类型（称之动态类型）。

- 在基类中声明一个非虚函数将会为该类建立起一种不变性（invariant），凌驾其特异性（specialization）。  
- 在派生类中重新定义该非虚函数，打破了基类“不变性凌驾特异性”的性质
## 条款 37：绝不重新定义继承而来的缺省参数值
虚函数是动态绑定而来，意思是调用一个虚函数时，究竟调用哪一份函数实现代码，取决于发出调用的那个对象的动态类型。  
缺省参数值却是静态绑定，在“调用一个定义于派生类的虚函数”的同时，可能使用基类所指定的缺省参数值。
```
class Shape {
public:
    enum class ShapeColor { Red, Green, Blue };
    virtual void Draw(ShapeColor color = ShapeColor::Red) const = 0;
    ...
};

class Rectangle : public Shape {
public:
    virtual void Draw(ShapeColor color = ShapeColor::Green) const;
};

class Circle : public Shape {
public:
    virtual void Draw(ShapeColor color) const;
};
```
此时若对派生类对象调用Draw函数，则会发现：
```
Shape* pr = new Rectangle;
Shape* pc = new Circle;

pr->Draw(Shape::ShapeColor::Green);    // 调用 Rectangle::Draw(Shape::Green)
pr->Draw();                            // 调用 Rectangle::Draw(Shape::Red)
pc->Draw();                            // 调用 Rectangle::Draw(Shape::Red)
```
解决方法：
```
class Shape {
public:
    enum class ShapeColor { Red, Green, Blue };
    void Draw(ShapeColor color = ShapeColor::Red) const { DoDraw(color); }
    ...
private:
    virtual void DoDraw(ShapeColor color) const = 0;
};

class Rectangle : public Shape {
public:
    ...
private:
    virtual void DoDraw(ShapeColor color) const;
};
```
## 条款 38：通过复合塑模出 has-a 或“根据某物实现出”
所谓复合（composition），指的是某种类型的对象内含它种类型的对象。  
复合通常意味着 has-a 或根据某物实现出（is-implemented-in-terms-of） 的关系，
- 当复合发生于应用域（application domain）内的对象之间，表现出 has-a 的关系；
- 当它发生于实现域（implementation domain）内则是表现出“根据某物实现出”的关系。

1. has-a
```
class Address { ... };
class PhoneNumber { ... };

class Person {
public:
    ...
private:
    std::string name;           // 合成成分物（composed object）
    Address address;            // 同上
    PhoneNumber voiceNumber;    // 同上
    PhoneNumber faxNumber;      // 同上
};
```
2. “根据某物实现出”
```
// 将 list 应用于 Set
template<class T>
class Set {
public:
    bool member(const T& item) const;
    void insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;

private:
    std::list<T> rep;           // 用来表述 Set 的数据
};
```
## 条款 39：明智而审慎地使用private继承
private继承的特点：
- 编译器不会自动将一个派生类对象转换为一个基类对象。
- private继承只继承实现，不继承接口。
- private继承的意义是“根据某物实现出”，绝大部分private继承的使用场合都可以被“public继承+复合”完美解决。
```
class Timer {
public:
    explicit Timer(int tickFrequency);
    virtual void OnTick() const;
    ...
};

class Widget : private Timer {
private:
    virtual void OnTick() const;
    ...
};
```
替代为：
```
class Widget {
private:
    class WidgetTimer : public Timer {
    public:
        virtual void OnTick() const;
        ...
    };
    WidgetTimer timer;
    ...
};
```
使用后者比前者好的原因有以下几点：
- private继承无法阻止派生类重新定义虚函数，但若使用public继承定义WidgetTimer类并复合在Widget类中，就能防止在Widget类中重新定义虚函数。
- 可以仅提供WidgetTimer类的声明，并将WidgetTimer类的具体定义移至实现文件中，从而降低Widget的编译依存性。

然而private继承并非完全一无是处，一个适用于它的极端情况是空白基类最优化（empty base optimization，EBO）
```
class Empty {};

class HoldsAnInt {
private:
    int x;
    Empty e;
};
```
注：C++ 规定凡是独立对象都必须有非零大小
使用private继承可以避免产生额外存储空间
```
class HoldsAnInt : private Empty {
private:
    int x;
};
```
## 条款 40：明智而审慎地使用多重继承
多重继承是一个可能会造成很多歧义和误解的设计，程序有可能从一个以上的基类继承相同名称，那会导致较多的歧义机会
- 必须明确地指出要调用哪一个基类中的函数

这时候必须面对这样一个问题：是否打算让基类内的成员变量经由每一条路径被复制？如果不想要这样，应当使用虚继承，指出其愿意共享基类：
- 非必要不使用虚继承，如果必须使用虚继承，尽可能避免在虚基类中放置数据。  
- 多重继承可用于结合public继承和private继承，public继承用于提供接口，private继承用于提供实现：
```
// IPerson 类指出要实现的接口
class IPerson {
public:
    virtual ~IPerson();
    virtual std::string Name() const = 0;
    virtual std::string BirthDate() const = 0;
};

class DatabaseID {  ...  };

// PersonInfo 类有若干已实现的函数
// 可用以实现 IPerson 接口
class PersonInfo {
public:
    explicit PersonInfo(DatabaseID pid);
    virtual ~PersonInfo();
    virtual const char* TheName() const;
    virtual const char* TheBirthDate() const;
    virtual const char* ValueDelimOpen() const;
    virtual const char* ValueDelimClose() const;
    ...
};

// CPerson 类使用多重继承
class CPerson: public IPerson, private PersonInfo {
public:
    explicit CPerson(DatabaseID pid): PersonInfo(pid) {}

    virtual std::string Name() const {       // 实现必要的 IPerson 成员函数
        return PersonInfo::TheName();  
    }

    virtual std::string BirthDate() const {  // 实现必要的 IPerson 成员函数
        return PersonInfo::TheBirthDate();  
    }
private:
    // 重新定义继承而来的虚函数
    const char* ValueDelimOpen() const {  return "";  }
    const char* ValueDelimClose() const {  return "";  }
};
```
# 第七章：模板与泛型编程
## 条款 41：了解隐式接口和编译期多态
类与模板都支持接口和多态。
- 对于类而言接口是显式的，以函数签名为中心，多态则是通过虚函数发生于运行期；
- 对模板参数而言，接口是隐式的，奠基于有效表达式，多态则是通过模板具现化和函数重载解析（function overloading resolution）发生于编译期。

```
template<typename T>
void DoProcessing(T& w) {
    if (w.size() > 10 && w != someNastyWidget) {
    ...
```
# 条款 42：了解 typename 的双重含义
- 在模板声明式中，使用class和typename关键字并没有什么不同
- 在模板内部，typename拥有更多的一重含义。

从属名称（dependent names）：模板内出现的名称如果相依于某个模板参数
嵌套从属名称（nested dependent name）： 如果从属名称在类内呈嵌套状
非从属名称（non-dependent names）：如果一个名称并不倚赖任何模板参数的名称

显式指明嵌套从属类型名称的方法就是将typename关键字作为其前缀词
`typename C::const_iterator iter(container.begin());`
若嵌套从属名称出现在模板函数声明部分，也需要显式地指明是否为类型名称
```
template<typename C>
void Print2nd(const C& container, const typename C::iterator iter);
```
注：typename 不可以出现在基类列表内的嵌套从属类型名称之前，也不可以在成员初始化列表中作为基类的修饰符：
```
template<typename T>
class Derived : public Base<T>::Nested {    // 基类列表中不允许使用 typename
public:
    explicit Derived(int x)
        : Base<T>::Nested(x) {                 // 成员初始化列表中不允许使用 typename
        typename Base<T>::Nested temp;
        ...
    }
    ...
};
```
在类型名称过于复杂时，可以使用using或typedef来进行简化：
```
using value_type = typename std::iterator_traits<IterT>::value_type;
```
## 条款 43：学习处理模板化基类内的名称
在模板编程中，模板类的继承并不像普通类那么自然
```
class MsgInfo { ... };

template<typename Company>
class MsgSender {
public:
    void SendClear(const MsgInfo& info) { ... }
    ...
};

template<typename Company>
class LoggingMsgSender : public MsgSender<Company> {
public:
    void SendClearMsg(const MsgInfo& info) {
        SendClear(info);        // 调用基类函数，这段代码无法通过编译
    }
    ...
};
```
很明显，由于直到模板类被真正实例化之前，编译器并不知道MsgSender<Company>具体长什么样  
注：C++ 的设计策略是宁愿较早进行诊断

为了解决这个问题，我们需要令 C++“进入模板基类观察”的行为生效，有三种办法达成这个目标：

第一种：在基类函数调用动作之前加上this->：
```
this->SendClear(info);
```
第二种：使用using声明式：
```
using MsgSender<Company>::SendClear;
SendClear(info);
```
第三种：明白指出被调用的函数位于基类内：
```
MsgSender<Company>::SendClear(info);
```
第三种做法是最不令人满意的，如果被调用的是虚函数，上述的明确资格修饰（explicit qualification）会使“虚函数绑定行为”失效。

## 条款 44：将与参数无关的代码抽离模板
模板可以节省时间和避免代码重复，编译器会为填入的每个不同模板参数具现化出一份对应的代码，但长此以外，可能会造成代码膨胀（code bloat），生成浮夸的二进制目标码。

基于共性和变性分析（commonality and variability analysis） 的方法，我们需要分析模板中重复使用的部分，将其抽离出模板，以减轻模板具现化带来的代码量。

因非类型模板参数而造成的代码膨胀，往往可以消除，做法是以函数参数或类成员变量替换模板参数。
因类型模板参数而造成的代码膨胀，往往可以降低，做法是让带有完全相同二进制表述的具现类型共享实现代码。
例：
```
template<typename T, std::size_t n>
class SquareMatrix {
public:
    void Invert();
    ...
private:
    std::array<T, n * n> data;
};

//修改为：
template<typename T>
class SquareMatrixBase {
protected:
    void Invert(std::size_t matrixSize);
    ...
private:
    std::array<T, n * n> data;
};

template<typename T, std::size_t n>
class SquareMatrix : private SquareMatrixBase<T> {  // private 继承实现，见条款 39
    using SquareMatrixBase<T>::Invert;              // 避免掩盖基类函数，见条款 33

public:
    void Invert() { this->Invert(n); }              // 调用模板基类函数，见条款 43
    ...
};
```
将数据放在派生类中，修改代码如下：
```
template<typename T>
class SquareMatrixBase {
protected:
    SquareMatrixBase(std::size_t n, T* pMem)
        : size(n), pData(pMem) {}
    void SetDataPtr(T* ptr) { pData = ptr; }
    ...
private:
    std::size_t size;
    T* pData;
};

template<typename T, std::size_t n>
class SquareMatrix : private SquareMatrixBase<T> {
public:
    SquareMatrix() : SquareMatrixBase<T>(n, data.data()) {}
    ...
private:
    std::array<T, n * n> data;
};
```
## 条款 45：运用成员函数模板接受所有兼容类型
C++ 视模板类的不同具现体为完全不同的的类型，但在泛型编程中，可能需要一个模板类的不同具现体能够相互类型转换。

模板拷贝构造函数：
```
template<typename T>
class SmartPtr {
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other)
        : heldPtr(other.get()) { ... }

    template<typename U>
    explicit SmartPtr(U* p)
        : heldPtr(p) { ... }

    T* get() const { return heldPtr; }
    ...
private:
    T* heldPtr;
};
```
使用get获取原始指针，并将在原始指针之间进行类型转换本身提供了一种保障，如果原始指针之间不能隐式转换，那么其对应的智能指针之间的隐式转换会造成编译错误。

模板构造函数并不会阻止编译器暗自生成默认的构造函数，必须同时声明泛化拷贝构造函数和普通拷贝构造函数，相同规则也适用于赋值运算符：
```
template<typename T>
class shared_ptr {
public:
    shared_ptr(shared_ptr const& r);                // 拷贝构造函数

    template<typename Y>
    shared_ptr(shared_ptr<Y> const& r);             // 泛化拷贝构造函数

    shared_ptr& operator=(shared_ptr const& r);     // 拷贝赋值运算符

    template<typename Y>
    shared_ptr& operator=(shared_ptr<Y> const& r);  // 泛化拷贝赋值运算符

    ...
};
```
## 条款 46：需要类型转换时请为模板定义非成员函数
模板实参在推导过程中，从不将隐式类型转换纳入考虑。可以使用友元声明式在模板类内指涉特定函数。  
```
template<typename T> class Rational;

template<typename T>
const Rational<T> DoMultiply(const Rational<T>& lhs, const Rational<T>& rhs) {
    return Rational<T>(lhs.Numerator() * rhs.Numerator(), lhs.Denominator() * rhs.Denominator());
}

template<typename T>
class Rational {
public:
    ...
    friend const Rational operator*(const Rational& lhs, const Rational& rhs) {
        return DoMultiply(lhs, rhs);
    }
    ...
};
```
## 条款 47：请使用 traits classes 表现类型信息
traits classes 可以使我们在编译期就能获取某些类型信息，它被广泛运用于 C++ 标准库中。traits 并不是 C++ 关键字或一个预先定义好的构件：它们是一种技术，也是 C++ 程序员所共同遵守的协议，并要求对用户自定义类型和内置类型表现得一样好。

设计并实现一个 trait class 的步骤如下：
1. 确认若干你希望将来可取得的类型相关信息。
2. 为该类型选择一个名称。
3. 提供一个模板和一组特化版本，内含你希望支持的类型相关信息。

以迭代器为例，标准库中拥有多种不同的迭代器种类，它们各自拥有不同的功用和限制：

input_iterator_tag：单向输入迭代器，只能向前移动，一次一步，客户只可读取它所指的东西。
output_iterator_tag：单向输出迭代器，只能向前移动，一次一步，客户只可写入它所指的东西。
forward_iterator_tag：单向访问迭代器，只能向前移动，一次一步，读写均允许。
bidirectional_iterator_tag：双向访问迭代器，去除了只能向前移动的限制。
random_access_iterator_tag：随机访问迭代器，没有一次一步的限制，允许随意移动，可以执行“迭代器算术”。

标准库为这些迭代器种类提供的卷标结构体（tag struct）的继承关系如下：
```
struct input_iterator_tag {};

struct output_iterator_tag {};

struct forward_iterator_tag : input_iterator_tag {};

struct bidirectional_iterator_tag : forward_iterator_tag {};

struct random_access_iterator_tag : bidirectional_iterator_tag {};
```
将iterator_category作为迭代器种类的名称，嵌入容器的迭代器中，并且确认使用适当的卷标结构体：
```
template< ... >
class deque {
public:
    class iterator {
    public:
        using iterator_category = random_access_iterator;
        ...
    }
    ...
}

template< ... >
class list {
public:
    class iterator {
    public:
        using iterator_category = bidirectional_iterator;
        ...
    }
    ...
}
```
为了做到类型的 traits 信息可以在类型自身之外获得，标准技术是把它放进一个模板及其一个或多个特化版本中。这样的模板在标准库中有若干个，其中针对迭代器的是iterator_traits：
```
template<class IterT>
struct iterator_traits {
    using iterator_category = IterT::iterator_category;
    ...
};
```
为了支持指针迭代器，iterator_traits特别针对指针类型提供一个偏特化版本，而指针的类型和随机访问迭代器类似，所以可以写出如下代码：
```
template<class IterT>
struct iterator_traits<IterT*> {
    using iterator_category = random_access_iterator_tag;
    ...
};
```
当我们需要为不同的迭代器种类应用不同的代码时，traits classes 就派上用场了
- 利用函数重载
```
template<typename IterT, typename DisT>
void doAdvance(IterT& iter, DisT d, std::random_access_iterator_tag) {
    ...
}   


template<typename IterT, typename DisT>
void doAdvance(IterT& iter, DisT d, std::bidirectional_iterator_tag) {
    ...
}

template<typename IterT, typename DisT>
void doAdvance(IterT& iter, DisT d, std::input_iterator_tag) {
    if (d < 0) {
        throw std::out_of_range("Negative distance");       // 单向迭代器不允许负距离
    }
    ...
}

template<typename IterT, typename DisT>
void advance(IterT& iter, DisT d) {
    doAdvance(iter, d, std::iterator_traits<IterT>::iterator_category());
}
```
- 使用if constexpr：
```
template<typename IterT, typename DisT>
void Advance(IterT& iter, DisT d) {
    if constexpr (typeid(std::iterator_traits<IterT>::iterator_category)
        == typeid(std::random_access_iterator_tag)) {
        ...
    }
}
```
## 条款 48：认识模板元编程
模板元编程（Template metaprogramming，TMP）是编写基于模板的 C++ 程序并执行于编译期的过程，  
由于模板元程序执行于 C++ 编译期，因此可以将一些工作从运行期转移至编译期
- 在编译期时发现一些原本要在运行期时才能察觉的错误
- 得到较小的可执行文件、较短的运行期、较少的内存需求
- 会使编译时间变长  
- 在模板元编程中没有真正意义上的循环，所有循环效果只能藉由递归实现，而递归在模板元编程中是由 “递归模板具现化（recursive template instantiation）” 实现的

```
template<unsigned n>            // Factorial<n> = n * Factorial<n-1>
struct Factorial {
    enum { value = n * Factorial<n-1>::value };
};

template<>
struct Factorial<0> {           // 处理特殊情况：Factorial<0> = 1
    enum { value = 1 };
};

std::cout << Factorial<5>::value;
```
应用场所：
1. 确保量度单位正确。
2. 优化矩阵计算。
3. 可以生成客户定制之设计模式（custom design pattern）实现品。
# 第八章：定制 new 和 delete
## 条款 49：了解 new-handler 的行为
当operator new无法满足某一内存分配需求时，会不断调用一个客户指定的错误处理函数，即所谓的 new-handler，直到找到足够内存为止，调用声明于<new>中的set_new_handler可以指定这个函数。new_handler和set_new_handler的定义如下：
```
namespace std {
    using new_handler = void(*)();
    new_handler set_new_handler(new_handler) noexcept;    // 返回值为原来持有的 new-handler
}
```
一个设计良好的 new-handler 函数必须做以下事情之一：  
1. 让更多的内存可被使用： 程序开始执行就分配一大块内存，而后当 new-handler 第一次被调用，将内存释还给程序使用。 
2. 安装另一个 new-handler。
3. 卸除 new-handler： 将nullptr传给set_new_handler，使operator new在内存分配不成功时抛出异常。  
4. 抛出 bad_alloc（或派生自 bad_alloc）的异常： 异常不会被operator new捕捉，会被传播到内存分配处。
5. 不返回： 通常调用std::abort或std::exit。

使用不同的方式处理内存分配失败情况——使用静态成员
```
public:
    static std::new_handler set_new_handler(std::new_handler p) noexcept;
    static void* operator new(std::size_t size);
private:
    static std::new_handler currentHandler;
};

// 做和 std::set_new_handler 相同的事情
std::new_handler Widget::set_new_handler(std::new_handler p) noexcept {
    std::new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler; 
}

void* Widget::operator new(std::size_t size) {
    auto globalHandler = std::set_new_handler(currentHandler);  // 切换至 Widget 的专属 new-handler
    void* ptr = ::operator new(size);                           // 分配内存或抛出异常
    std::set_new_handler(globalHandler);                        // 切换回全局的 new-handler
    return globalHandler;
}

std::new_handler Widget::currentHandler = nullptr;
```
Widget的客户应该类似这样使用其 new-handling：
```
void OutOfMem();

Widget::set_new_handler(OutOfMem);
auto pw1 = new Widget;              // 若分配失败，则调用 OutOfMem

Widget::set_new_handler(nullptr);
auto pw2 = new Widget;              // 若分配失败，则抛出异常

template<typename T>
class NewHandlerSupport {       // “mixin”风格的基类
public:
    static std::new_handler set_new_handler(std::new_handler p) noexcept;
    static void* operator new(std::size_t size);
    ...                         // 其它的 operator new 版本，见条款 52
private:
    static std::new_handler currentHandler;
};

template<typename T>
std::new_handler NewHandlerSupport<T>::set_new_handler(std::new_handler p) noexcept {
    std::new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler;
}

template<typename T>
void* NewHandlerSupport<T>::operator new(std::size_t size) {
    auto globalHandler = std::set_new_handler(currentHandler);
    void* ptr = ::operator new(size);
    std::set_new_handler(globalHandler);
    return globalHandler;
}

template<typename T>
std::new_handler NewHandlerSupport<T>::currentHandler = nullptr;

class Widget : public NewHandlerSupport<Widget> {
public:
    ...                         // 不必再声明 set_new_handler 和 operator new
};
```
注意此处的模板参数T并没有真正被当成类型使用，而仅仅是用来区分不同的派生类，使得模板机制为每个派生类具现化出一份对应的currentHandler。

这个做法用到了所谓的 CRTP（curious recurring template pattern，奇异递归模板模式） ，除了在上述设计模式中用到之外，它也被用于实现静态多态：
```
template <class Derived> 
struct Base {
    void Interface() {
        static_cast<Derived*>(this)->Implementation();      // 在基类中暴露接口
    }
};

struct Derived : Base<Derived> {
    void Implementation();                                  // 在派生类中提供实现
};
```
除了会调用 new-handler 的operator new以外，C++ 还保留了传统的“分配失败便返回空指针”的operator new，称为 nothrow new，通过std::nothrow对象来使用它：
```
Widget* pw1 = new Widget;                   // 如果分配失败，抛出 bad_alloc
if (pw1 == nullptr) ...                     // 这个测试一定失败

Widget* pw2 = new (std::nothrow) Widget;    // 如果分配失败，返回空指针
if (pw2 == nullptr) ...                     // 这个测试可能成功
```
nothrow new 对异常的强制保证性并不高，使用它只能保证operator new不抛出异常，而无法保证像new (std::nothrow) Widget这样的表达式不会导致异常，因此实际上并没有使用 nothrow new 的必要。

## 条款 50：了解 new 和 delete 的合理替换时机
以下是常见的替换默认operator new和operator delete的理由：  
- 用来检测运用上的错误： 如果将“new 所得内存”delete 掉却不幸失败，会导致内存泄漏；  
- 如果在“new 所得内存”身上多次 delete 则会导致未定义行为。
- 如果令operator new持有一串动态分配所得地址，而operator delete将地址从中移除，就很容易检测出上述错误用法。  
- 此外各式各样的编程错误可能导致 “overruns”（写入点在分配区块尾端之后） 和 “underruns”（写入点在分配区块起点之前），以额外空间放置特定的 byte pattern 签名，检查签名是否原封不动就可以检测此类错误。

例：
```
static const int signature = 0xDEADBEEF;              // 调试“魔数”
using Byte = unsigned char;

void* operator new(std::size_t size) {
    using namespace std;
    size_t realSize = size + 2 * sizeof(int);         // 分配额外空间以塞入两个签名

    void* pMem = malloc(realSize);                    // 调用 malloc 取得内存
    if (!pMem) throw bad_alloc();

    // 将签名写入内存的起点和尾端
    *(static_cast<int*>(pMem)) = signature;
    *(reinterpret_cast<int*>(static_cast<Byte*>(pMem) + realSize - sizeof(int))) = signature;

    return static_cast<Byte*>(pMem) + sizeof(int);    // 返回指针指向第一个签名后的内存位置
}
```
为了收集使用上的统计数据： 定制 new 和 delete 动态内存的相关信息：分配区块的大小分布，寿命分布，FIFO（先进先出）、LIFO（后进先出）或随机次序的倾向性，不同的分配/归还形态，使用的最大动态分配量等等。

为了增加分配和归还的速度： 泛用型分配器往往（虽然并非总是）比定制型分配器慢，特别是当定制型分配器专门针对某特定类型之对象设计时。类专属的分配器可以做到“区块尺寸固定”，例如 Boost 提供的 Pool 程序库。

为了降低缺省内存管理器带来的空间额外开销： 泛用型分配器往往（虽然并非总是）还比定制型分配器使用更多内存，

为了弥补缺省分配器中的非最佳内存对齐（suboptimal alignment）： 许多计算机体系架构要求特定的类型必须放在特定的内存地址上，如果没有奉行这个约束条件，可能导致运行期硬件异常，或者访问速度变低。std::max_align_t用来返回当前平台的最大默认内存对齐类型，对于malloc分配的内存，其对齐和max_align_t类型的对齐大小应当是一致的，但若对malloc返回的指针进行偏移，就没有办法保证内存对齐。

在 C++11 中，提供了以下内存对齐相关方法：
```
// alignas 用于指定栈上数据的内存对齐要求
struct alignas(8) testStruct { double data; };

// alignof 和 std::alignment_of 用于得到给定类型的内存对齐要求
std::cout << alignof(std::max_align_t) << std::endl;
std::cout << std::alignment_of<std::max_align_t>::value << std::endl;

// std::align 用于在一大块内存中获取一个符合指定内存要求的地址
char buffer[] = "memory alignment";
void* ptr = buffer;
std::size_t space = sizeof(buffer) - 1;
std::align(alignof(int), sizeof(char), ptr, space);
```
在 C++17 后，可以使用std::align_val_t来重载需求额外内存对齐的operator new：
```
void* operator new(std::size_t count, std::align_val_t al);
```
- 为了将相关对象成簇集中： 如果特定的某个数据结构往往被一起使用，希望在处理这些数据时将“内存页错误（page faults）”的频率降至最低，可以考虑为此数据结构创建一个堆，将它们成簇集中在尽可能少的内存页上。使用 placement new 达成这个目标

- 为了获得非传统的行为： 希望operator new和operator delete做编译器版不会做的事情，例如分配和归还共享内存（shared memory），而这些事情只能被 C API 完成，则可以将 C API 封在 C++ 的外壳里，写在定制的 new 和 delete 中。

## 条款 51：编写 new 和 delete 时需固守常规
即使客户需求为0字节，operator new也得返回一个合法的指针
```
void* operator new(std::size_t size) {
    using namespace std;

    if (size == 0)      // 处理0字节申请
        size = 1;       // 将其视为1字节申请

    while (true) {
        if (...)        // 如果分配成功
            return ...; // 返回指针指向分配得到的内存

        // 如果分配失败，调用目前的 new-handler
        auto globalHandler = get_new_handler(); // since C++11

        if (globalHandler) (*globalHandler)();
        else throw std::bad_alloc();
    }
}
```
operator new的成员函数版本一般只会分配大小刚好为类的大小的内存空间，但派生类可以使用其基类的 new 分配方式，但派生类和基类的大小很多时候是不同的。  
处理此情况的最佳做法是将“内存申请量错误”的调用行为改为采用标准的operator new：
```
void* Base::operator new(std::size_t size) {
    if (size != sizeof(Base))
        return ::operator new(size);    // 转交给标准的 operator new 进行处理

    ...
}
```
实现operator new[]，即所谓的 array new，唯一要做的一件事就是分配一块未加工的原始内存。  
operator delete的规约更加简单，唯一一件事情就是 C++ 保证 “删除空指针永远安全”：
```
void operator delete(void* rawMemory) noexcept {
    if (rawMemory == 0) return;

    // 归还 rawMemory 所指的内存
}
```
operator delete的成员函数版本要多做的唯一一件事就是将大小有误的删除行为转交给标准的operator delete：
```
void Base::operator delete(void* rawMemory, std::size_t size) noexcept {
    if (rawMemory == 0) return;
    if (size != sizeof(Base)) {
        ::operator delete(rawMemory);    // 转交给标准的 operator delete 进行处理
        return;
    }

    // 归还 rawMemory 所指的内存
}
```
## 条款 52：写了 placement new 也要写 placement delete
placement new 最初的含义指的是“接受一个指针指向对象该被构造之处”的operator new版本，它在标准库中的用途广泛，其中之一是负责在 vector 的未使用空间上创建对象，它的声明如下：
```
void* operator new(std::size_t, void* pMemory) noexcept;
```
广义上的 placement new，即带有附加参数的operator new，例：
```
void* operator new(std::size_t, std::ostream& logStream);

auto pw = new (std::cerr) Widget;
```
当使用 new 表达式创建对象时，共有两个函数被调用：
- 一个是用以分配内存的operator new，
- 一个是对象的构造函数。  
 
假设第一个函数调用成功，而第二个函数却抛出异常，那么会由 C++ runtime 调用operator delete，归还已经分配好的内存。  
这一切的前提是 C++ runtime 能够找到operator new对应的operator delete。  
```
class Widget {
public:
    static void* operator new(std::size_t size, std::ostream& logStream);   // placement new

    static void operator delete(void* pMemory);                             // delete 时调用的正常 operator delete
    static void operator delete(void* pMemory, std::ostream& logStream);    // placement delete
};
```
另一个要注意的问题是，由于成员函数的名称会掩盖其外部作用域中的相同名称，所以提供 placement new 会导致无法使用正常版本的operator new：
```
class Base {
public:
    static void* operator new(std::size_t size, std::ostream& logStream);
    ...
};

auto pb = new Base;             // 无法通过编译！
auto pb = new (std::cerr) Base; // 正确
```
同样道理，派生类中的operator new会掩盖全局版本和继承而得的operator new版本：
```
class Derived : public Base {
public:
    static void* operator new(std::size_t size);
    ...
};

auto pd = new (std::clog) Derived;  // 无法通过编译！
auto pd = new Derived;              // 正确
```
为了避免名称遮掩问题，需要确保以下形式的operator new对于定制类型仍然可用。
```
void* operator(std::size_t) throw(std::bad_alloc);           // normal new
void* operator(std::size_t, void*) noexcept;                 // placement new
void* operator(std::size_t, const std::nothrow_t&) noexcept; // nothrow new
```
一个最简单的实现方式是，准备一个基类，内含所有正常形式的 new 和 delete：
```
class StadardNewDeleteForms{
public:
    // normal new/delete
    static void* operator new(std::size_t size){
        return ::operator new(size);
    }
    static void operator delete(void* pMemory) noexcept {
        ::operator delete(pMemory);
    }

    // placement new/delete
    static void* operator new(std::size_t size, void* ptr) {
        return ::operator new(size, ptr);
    }
    static void operator delete(void* pMemory, void* ptr) noexcept {
        ::operator delete(pMemory, ptr);
    }

    // nothrow new/delete
    static void* operator new(std::size_t size, const std::nothrow_t& nt) {
        return ::operator new(size,nt);
    }
    static void operator delete(void* pMemory,const std::nothrow_t&) noexcept {
        ::operator delete(pMemory);
    }
};
```
凡是想以自定义形式扩充标准形式的客户，可以利用继承和using声明式（见条款 33）取得标准形式：
```
class Widget: public StandardNewDeleteForms{
public:
    using StandardNewDeleteForms::operator new;
    using StandardNewDeleteForms::operator delete;

    static void* operator new(std::size_t size, std::ostream& logStream);
    static void operator detele(std::size_t size, std::ostream& logStream) noexcept;
    ...
};
```
# 第九章：杂项讨论
## 条款 53：不要轻忽编译器的警告
- 严肃对待编译器发出的警告信息
- 不要过度依赖编译器的警告能力
## 条款 54：让自己熟悉包括 TR1 在内的标准程序库
## 条款 55：让自己熟悉 Boost