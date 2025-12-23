# 前言

模板是 C++ 泛型编程的基础，泛型编程即以一种独立于特定类型的方式编写代码。  

模板是为了一种或者多种未明确定义的类型而定义的函
数或者类。在使用模板时，需要显式地或者隐式地指定模板参数。     

目前模板正在被广泛使用。比如在C++标准库中，几乎所有的代码都用到了模板。  

# 函数模板  
函数模板提供了适用于不同数据类型的函数行为。即函数模板代表的是一组函数。

建立一个通用函数，凡是函数体相同的函数都用模板代替，系统根据实参类型来取代模板中的虚拟类型，实现不同函数的功能。  
```
template <typename T>     // 类型形式参数表
T func(T a, T b)          // 类型 函数名(形式参数表)
{                         // {
    return a > b ? a : b; //    函数体
}                         // }
```  
注：
- 虚拟参数类型，可以定义多个，但必须全部使用
- typename 和 class 效果相同，但 class 存在歧义
- 函数模板可以使用<>显示类型调用，同时存在自动数据类型推导
- 参数指定只允许第一个或全部指定
- 函数模板和函数重载同时存在其参数匹配时，普通重载函数优先匹配
- 函数模板不允许自动类型转化  

本质：编译器通过函数模板生成多个模板函数，调用对应模板函数执行  

## 两阶段编译检查
### 模板定义
- 语法检查
- 存在未定义名称
- 未使用模板参数的 static assertions

### 模板实例化
- 类型参数

1. 按值传递 或引用传递?
   - 语法简单。
   - 编译器能够更好地进行优化。
   - 移动语义通常使拷贝成本比较低。
   - 某些情况下可能没有拷贝或者移动。
2. 不适应 inline？
   - 函数模板不需要被声明成inline。可以把非 inline 的函
数模板定义在头文件里，然后在多个编译单元里 include
   - inline 只意味着在程序中函数的定义可以出现很多次
  
# 类模板
```
template<typename T>
 class Stack {
    …
 };
```
注：**不可以在函数内部或者块作用域内{...}声明和定义模板**

定义类模板的成员函数时，必须指出它是一个模板，也必须使用该类模板的所有类型限制  
```
 template<typename T>
 void Stack<T>::push (T const& elem)
 {
 elems.push_back(elem); // append copy of passed elem
 }
```
- 如果一个类模板有static成员，对每一个用到这个类模板
的类型，相应的静态成员也只会被实例化一次。
- 模板函数和模板成员函数只有在被调用的时候才会实例化，类模板中的普通成员函数同理

## concept

concept通常被用来表示一组反复被模板库要求的限制条件。

从C++11 开始，可以通过关键字static_assert 和其它一些预定义的类型萃取（typetraits）来做一些简单的检查。比如：
```
template<typename T>
 class C
 {
 };
 static_assert(std::is_default_constructible<T>::value,
 "Class C requires default-constructible elements");

```

## 友元
### 1.隐式声明函数模板
```
template<typename T>
 class Stack {
    …
    template<typename U>
    friend std::ostream& operator<< (std::ostream&, Stack<U> const&);
 };
```
### 2.操作声明为模板
```
template<typename T>
class Stack;
template<typename T>
std::ostream& operator<< (std::ostream&, Stack<T> const&);
```
接着就可以将这一模板声明为 Stack<T> 的友元：
```
template<typename T>
 class Stack {
    …
    friend std::ostream& operator<< <T> (std::ostream&, Stack<T>
 const&);
 }
```

## 特例化
```
template<>
 class Stack<std::string> {
    …
 };
```