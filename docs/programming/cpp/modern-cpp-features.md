---

---


# 现代 C++ 特性

!!! info "原文链接"
	[A cheatsheet of modern C++ language and library features.](https://github.com/AnthonyCalandra/modern-cpp-features)

!!! warning "注意"
    本文由 chatgpt 辅助翻译后，人工核对整理，已尽可能保证内容的准确性，如发现内容性错误也烦请您联系指正。

## C++11

### C++11 语言特性

#### 移动语义（Move semantics）

移动一个对象意味着将其管理的某些资源的所有权转移给另一个对象。

移动语义的第一个好处是性能优化。当一个对象即将结束其生命周期时，无论是因为它是一个临时对象还是通过显式调用 `std::move`，移动通常是更廉价的资源转移方式。例如，移动一个 `std::vector` 只需将一些指针和内部状态复制到新向量中，而复制则需要复制向量中的每个元素，这既昂贵又不必要，尤其是在旧向量即将被销毁的情况下。

移动还使得像 `std::unique_ptr`（[智能指针](#smart-pointers)）这样不可复制的类型可以在语言层面上保证资源在任何时候只有一个实例在被管理，同时还能在不同作用域之间转移实例。

参见以下章节：[右值引用](#rvalue-references)、[用于移动语义的特殊成员函数](#special-member-functions-for-move-semantics)、[`std::move`](#stdmove)、[`std::forward`](#stdforward)、[转发引用](#forwarding-references)。

#### 右值引用（Rvalue references）

C++11 引入了一种新的引用类型，称为*右值引用*。对 `T` 的右值引用（`T` 是一个非模板类型参数，如 `int` 或用户定义类型）使用 `T&&` 语法创建。右值引用只绑定到右值。

左值和右值的类型推导：

```c++
int x = 0; // `x` 是类型为 `int` 的左值
int& xl = x; // `xl` 是类型为 `int&` 的左值
int&& xr = x; // 编译错误 -- `x` 是一个左值
int&& xr2 = 0; // `xr2` 是类型为 `int&&` 的左值 -- 绑定到右值临时对象 `0`

void f(int& x) {}
void f(int&& x) {}

f(x);  // 调用 f(int&)
f(xl); // 调用 f(int&)
f(3);  // 调用 f(int&&)
f(std::move(x)); // 调用 f(int&&)

f(xr2);           // 调用 f(int&)
f(std::move(xr2)); // 调用 f(int&& x)
```

另见：[`std::move`](#stdmove)、[`std::forward`](#stdforward)、[转发引用](#forwarding-references)。

#### 转发引用（Forwarding references）

非正式名称也叫*通用引用*。转发引用使用 `T&&` 语法创建，其中 `T` 是一个模板类型参数，或者使用 `auto&&`。这实现了*完美转发*：能够在保持其值类别（例如左值保持为左值，临时对象作为右值转发）的同时传递参数。

转发引用允许引用根据类型绑定到左值或右值。转发引用遵循*引用折叠*规则：

- `T& &` 变为 `T&`
- `T& &&` 变为 `T&`
- `T&& &` 变为 `T&`
- `T&& &&` 变为 `T&&`

使用左值和右值的 `auto` 类型推导：
```c++
int x = 0; // `x` 是类型为 `int` 的左值
auto&& al = x; // `al` 是类型为 `int&` 的左值 -- 绑定到左值 `x`
auto&& ar = 0; // `ar` 是类型为 `int&&` 的左值 -- 绑定到右值临时对象 `0`
```

模板类型参数与左值和右值的推导：
```c++
// C++14 及更高版本：
void f(auto&& t) {
  // ...
}

// C++11 及更高版本：
template <typename T>
void f(T&& t) {
  // ...
}

int x = 0;
f(0); // T 是 int，推导为 f(int &&) => f(int&&)
f(x); // T 是 int&，推导为 f(int& &&) => f(int&)

int& y = x;
f(y); // T 是 int&，推导为 f(int& &&) => f(int&)

int&& z = 0; // 注意：`z` 是一个类型为 `int&&` 的左值。
f(z); // T 是 int&，推导为 f(int& &&) => f(int&)
f(std::move(z)); // T 是 int，推导为 f(int &&) => f(int&&)
```

另见：[`std::move`](#stdmove)、[`std::forward`](#stdforward)、[右值引用](#rvalue-references)。

#### 可变参数模板（Variadic templates）

`...` 语法创建一个*参数包*或展开一个参数包。模板*参数包*是接受零个或多个模板参数（非类型、类型或模板）的模板参数。至少有一个参数包的模板称为*可变参数模板*。
```c++
template <typename... T>
struct arity {
  constexpr static int value = sizeof...(T);
};
static_assert(arity<>::value == 0);
static_assert(arity<char, short, int>::value == 3);
```

一个有趣的用法是从*参数包*创建一个*初始化列表*，以便遍历可变参数函数的参数。

```c++
template <typename First, typename... Args>
auto sum(const First first, const Args... args) -> decltype(first) {
  const auto values = {first, args...};
  return std::accumulate(values.begin(), values.end(), First{0});
}

sum(1, 2, 3, 4, 5); // 15
sum(1, 2, 3);       // 6
sum(1.5, 2.0, 3.7); // 7.2
```

#### 初始化列表（Initializer lists）

使用大括号语法创建的轻量级类数组容器。例如，`{ 1, 2, 3 }` 创建一个整数序列，其类型为 `std::initializer_list<int>`。用于替代将对象向量传递给函数。

```c++
int sum(const std::initializer_list<int>& list) {
  int total = 0;
  for (auto& e : list) {
    total += e;
  }

  return total;
}

auto list = {1, 2, 3};
sum(list); // == 6
sum({1, 2, 3}); // == 6
sum({}); // == 0
```

#### 静态断言（Static assertions）

在编译时评估的断言。

```c++
constexpr int x = 0;
constexpr int y = 1;
static_assert(x == y, "x != y");
```

#### auto

`auto` 类型的变量根据其初始化器的类型由编译器推导。

```c++
auto a = 3.14; // double
auto b = 1; // int
auto& c = b; // int&
auto d = { 0 }; // std::initializer_list<int>
auto&& e = 1; // int&&
auto&& f = b; // int&
auto g = new auto(123); // int*
const auto h = 1; // const int
auto i = 1, j = 2, k = 3; // int, int, int
auto l = 1, m = true, n = 1.61; // 错误 -- `l` 被推导为 int，`m` 是 bool
auto o; // 错误 -- `o` 需要初始化器
```

对复杂类型的可读性特别有用：

```c++
std::vector<int> v = ...;
std::vector<int>::const_iterator cit = v.cbegin();
// vs.
auto cit = v.cbegin();
```

函数也可以使用 `auto` 推导返回类型。在 C++11 中，返回类型必须显式指定，或者使用 `decltype`，如下所示：

```c++
template <typename X, typename Y>
auto add(X x, Y y) -> decltype(x + y) {
  return x + y;
}
add(1, 2); // == 3
add(1, 2.0); // == 3.0
add(1.5, 1.5); // == 3.0
```

上例中的尾置返回类型是表达式 `x + y` 的*声明类型*（参见 [`decltype`](#decltype) 部分）。例如，如果 `x` 是整数而 `y` 是双精度类型，则 `decltype(x + y)` 是双精度类型。因此，上述函数将根据表达式 `x + y` 所产生的类型来推导返回类型。注意，尾置返回类型可以访问其参数和 `this`（如果适用）。

#### Lambda 表达式（Lambda expressions）

`lambda` 是一个未命名的函数对象，能够捕获当前作用域中的变量。它具有：一个*捕获列表*；一个可选的参数集（带有可选的尾置返回类型）；以及一个函数体。捕获列表的示例：

- `[]`：不捕获任何变量。
- `[=]`：通过值捕获当前作用域中的局部对象（局部变量，参数）。
- `[&]`：通过引用捕获当前作用域中的局部对象（局部变量，参数）。
- `[this]`：通过引用捕获 `this`。
- `[a, &b]`：通过值捕获对象 `a`，通过引用捕获对象 `b`。

```c++
int x = 1;

auto getX = [=] { return x; };
getX(); // == 1

auto addX = [=](int y) { return x + y; };
addX(1); // == 2

auto getXRef = [&]() -> int& { return x; };
getXRef(); // 返回 `x` 的引用
```

默认情况下，值捕获在 lambda 内无法被修改，因为编译器生成的方法被标记为 `const`。`mutable` 关键字允许在 lambda 内修改被捕获的变量。该关键字位于参数列表之后（即使参数列表为空，也必须存在）。

```c++
int x = 1;

auto f1 = [&x] { x = 2; }; // 正确: x 是一个引用，修改了原始变量

auto f2 = [x] { x = 2; }; // 错误：lambda 只能对捕获的值执行 const 操作
// vs.
auto f3 = [x]() mutable { x = 2; }; // 正确: lambda 可以对捕获的值执行任何操作
```
#### decltype

`decltype` 是一个操作符，它返回传递给它的表达式的*声明类型*。如果 `cv` 限定符和引用是表达式的一部分，那么它们会被保留。以下是 `decltype` 的一些示例：

```c++
int a = 1; // `a` 被声明为类型 `int`
decltype(a) b = a; // `decltype(a)` 是 `int`
const int& c = a; // `c` 被声明为类型 `const int&`
decltype(c) d = a; // `decltype(c)` 是 `const int&`
decltype(123) e = 123; // `decltype(123)` 是 `int`
int&& f = 1; // `f` 被声明为类型 `int&&`
decltype(f) g = 1; // `decltype(f)` 是 `int&&`
decltype((a)) h = g; // `decltype((a))` 是 `int&`
```

```c++
template <typename X, typename Y>
auto add(X x, Y y) -> decltype(x + y) {
  return x + y;
}
add(1, 2.0); // `decltype(x + y)` => `decltype(3.0)` => `double`
```

参见：[decltype(auto) (C++14)](#decltypeauto)。

#### 类型别名（Type aliases）

类型别名语义上类似于使用 `typedef`，但是使用 `using` 的类型别名更易于阅读并且与模板兼容。

```c++
template <typename T>
using Vec = std::vector<T>;
Vec<int> v; // std::vector<int>

using String = std::string;
String s {"foo"};
```

#### nullptr

C++11 引入了一种新的空指针类型，旨在替代 C 的 `NULL` 宏。`nullptr` 本身的类型是 `std::nullptr_t`，它可以隐式转换为指针类型，而不像 `NULL` 那样可以转换为整型（除了 `bool` 以外）。

```c++
void foo(int);
void foo(char*);
foo(NULL); // 错误 -- 不明确
foo(nullptr); // 调用 foo(char*)
```

#### 强类型枚举（Strongly-typed enums）

强类型枚举解决了 C 风格枚举的多种问题，包括：隐式转换、无法指定底层类型、作用域污染等。

```c++
// 指定底层类型为 `unsigned int`
enum class Color : unsigned int { Red = 0xff0000, Green = 0xff00, Blue = 0xff };
// `Alert` 中的 `Red`/`Green` 不会与 `Color` 冲突
enum class Alert : bool { Red, Green };
Color c = Color::Red;
```

#### 属性（Attributes）

属性提供了一种统一的语法来替代 `__attribute__(...)`，`__declspec` 等。

```c++
// `noreturn` 属性表示 `f` 不会返回。
[[ noreturn ]] void f() {
  throw "error";
}
```

#### constexpr

常量表达式是*可能*在编译时由编译器计算的表达式。只有非复杂的计算可以在常量表达式中进行（这些规则在后续版本中逐渐放宽）。使用 `constexpr` 说明符来指示变量、函数等是常量表达式。

```c++
constexpr int square(int x) {
  return x * x;
}

int square2(int x) {
  return x * x;
}

int a = square(2);  // mov DWORD PTR [rbp-4], 4

int b = square2(2); // mov edi, 2
                    // call square2(int)
                    // mov DWORD PTR [rbp-8], eax
```

在上面的代码片段中，可以注意到调用 `square` 时的计算是在编译时进行的，然后结果嵌入到代码生成中，而 `square2` 是在运行时调用的。

`constexpr` 值是编译器可以在编译时计算的值，但不一定会计算：

```c++
const int x = 123;
constexpr const int& y = x; // 错误 -- constexpr 变量 `y` 必须由常量表达式初始化
```

与类一起使用的常量表达式：

```c++
struct Complex {
  constexpr Complex(double r, double i) : re{r}, im{i} { }
  constexpr double real() { return re; }
  constexpr double imag() { return im; }

private:
  double re;
  double im;
};

constexpr Complex I(0, 1);
```

#### 委托构造函数（Delegating constructors）

构造函数现在可以使用初始化列表调用同一个类中的其他构造函数。

```c++
struct Foo {
  int foo;
  Foo(int foo) : foo{foo} {}
  Foo() : Foo(0) {}
};

Foo foo;
foo.foo; // == 0
```

#### 用户定义的字面量（User-defined literals）

用户定义的字面量允许您扩展语言并添加自己的语法。要创建字面量，请定义一个 `T operator "" X(...) { ... }` 函数，该函数返回一个类型 `T`，名称为 `X`。请注意，此函数的名称定义了字面量的名称。任何不以下划线开头的字面量名称都是保留的，不会被调用。根据字面量的类型，用户定义的字面量函数应该接受的参数有一些规则。

将摄氏温度转换为华氏温度：

```c++
// 整型字面量需要 `unsigned long long` 参数。
long long operator "" _celsius(unsigned long long tempCelsius) {
  return std::llround(tempCelsius * 1.8 + 32);
}
24_celsius; // == 75
```

字符串到整数转换：

```c++
// 参数需要是 `const char*` 和 `std::size_t`。
int operator "" _int(const char* str, std::size_t) {
  return std::stoi(str);
}

"123"_int; // == 123, 类型为 `int`
```

#### 显式虚函数覆盖（Explicit virtual overrides）

指定一个虚函数覆盖了另一个虚函数。如果虚函数没有覆盖父类的虚函数，则抛出编译器错误。

```c++
struct A {
  virtual void foo();
  void bar();
};

struct B : A {
  void foo() override; // 正确 -- B::foo 覆盖了 A::foo
  void bar() override; // 错误 -- A::bar 不是虚函数
  void baz() override; // 错误 -- B::baz 没有覆盖 A::baz
};
```

#### final 说明符（Final specifier）

指定虚函数不能在派生类中被重写，或者指定类不能被继承。

```c++
struct A {
  virtual void foo();
};

struct B : A {
  virtual void foo() final;
};

struct C : B {
  virtual void foo(); // 错误 -- 声明 'foo' 覆盖了一个 'final' 函数
};
```

类不能被继承：

```c++
struct A final {};
struct B : A {}; // 错误 -- 基类 'A' 被标记为 'final'
```

#### 默认函数（Default functions）

提供函数默认实现的更优雅、高效的方式，例如构造函数。

```c++
struct A {
  A() = default;
  A(int x) : x{x} {}
  int x {1};
};
A a; // a.x == 1
A a2 {123}; // a.x == 123
```

与继承一起使用：

```c++
struct B {
  B() : x{1} {}
  int x;
};

struct C : B {
  // 调用 B::B
  C() = default;
};

C c; // c.x == 1
```

#### 已删除函数（Deleted functions）

提供函数删除实现的更优雅、高效的方式。用于防止对象被复制。

```c++
class A {
  int x;

public:
  A(int x) : x{x} {};
  A(const A&) = delete;
  A& operator=(const A&) = delete;
};

A x {123};
A y = x; // 错误 -- 调用已删除的复制构造函数
y = x; // 错误 -- `operator=` 已删除
```

#### 基于范围的 for 循环（Range-based for loops）

用于迭代容器元素的语法糖。

```c++
std::array<int, 5> a {1, 2, 3, 4, 5};
for (int& x : a) x *= 2;
// a == { 2, 4, 6, 8, 10 }
```

注意使用 `int` 和 `int&` 的区别：

```c++
std::array<int, 5> a {1, 2, 3, 4, 5};
for (int x : a) x *= 2;
// a == { 1, 2, 3, 4, 5 }
```

#### 移动语义的特殊成员函数（Special member functions for move semantics）

当进行复制时，会调用复制构造函数和复制赋值运算符，而随着 C++11 引入的移动语义，现在也有了移动构造函数和移动赋值运算符。

```c++
struct A {
  std::string s;
  A() : s{"test"} {}
  A(const A& o) : s{o.s} {}
  A(A&& o) : s{std::move(o.s)} {}
  A& operator=(A&& o) {
   s = std::move(o.s);
   return *this;
  }
};

A f(A a) {
  return a;
}

A a1 = f(A{}); // 从右值临时对象移动构造
A a2 = std::move(a1); // 使用 std::move 移动构造
A a3 = A{};
a2 = std::move(a3); // 使用 std::move 移动赋值
a1 = f(A{}); // 从右值临时对象移动赋值
```

#### 转换构造函数（Converting constructors）

转换构造函数将大括号列表语法的值转换为构造函数参数。

```c++
struct A {
  A(int) {}
  A(int, int) {}
  A(int, int, int) {}
};

A a {0, 0}; // 调用 A::A(int, int)
A b(0, 0); // 调用 A::A(int, int)
A c = {0, 0}; // 调用 A::A(int, int)
A d {0, 0, 0}; // 调用 A::A(int, int, int)
```

请注意，大括号列表语法不允许缩小：

```c++
struct A {
  A(int) {}
};

A a(1.1); // 正确
A b {1.1}; // 错误 缩小转换从 double 到 int
```

请注意，如果构造函数接受 `std::initializer_list`，它将被调用：

```c++
struct A {
  A(int) {}
  A(int, int) {}
  A(int, int, int) {}
  A(std::initializer_list<int>) {}
};

A a {0, 0}; // 调用 A::A(std::initializer_list<int>)
A b(0, 0); // 调用 A::A(int, int)
A c = {0, 0}; // 调用 A::A(std::initializer_list<int>)
A d {0, 0, 0}; // 调用 A::A(std::initializer_list<int>)
```

#### 显式转换函数（Explicit conversion functions）

转换函数现在可以使用 `explicit` 说明符来显式指定。

```c++
struct A {
  operator bool() const { return true; }
};

struct B {
  explicit operator bool() const { return true; }
};

A a;
if (a); // 正确 调用 A::operator bool()
bool ba = a; // 正确 拷贝初始化选择 A::operator bool()

B b;
if (b); // 正确 调用 B::operator bool()
bool bb = b; // 错误 拷贝初始化不考虑 B::operator bool()
```

#### 内联命名空间（Inline namespaces）

内联命名空间的所有成员都被视为其父命名空间的一部分，允许函数的特化并简化版本控制。这是一种传递性质，如果 A 包含 B，而 B 又包含 C，且 B 和 C 都是内联命名空间，则 C 的成员可以像在 A 中一样使用。

```c++
namespace Program {
  namespace Version1 {
    int getVersion() { return 1; }
    bool isFirstVersion() { return true; }
  }
  inline namespace Version2 {
    int getVersion() { return 2; }
  }
}

int version {Program::getVersion()};              // 使用 Version2 中的 getVersion()
int oldVersion {Program::Version1::getVersion()}; // 使用 Version1 中的 getVersion()
bool firstVersion {Program::isFirstVersion()};    // 当 Version2 被添加时，编译失败
```

#### 非静态数据成员初始化器（Non-static data member initializers）

允许在声明非静态数据成员时进行初始化，可能会清理默认初始化的构造函数。

```c++
// C++11 之前的默认初始化
class Human {
    Human() : age{0} {}
  private:
    unsigned age;
};
// C++11 上的默认初始化
class Human {
  private:
    unsigned age {0};
};
```

#### 右尖括号（Right angle brackets）

C++11 现在能够推断一系列右尖括号是用作运算符还是作为 `typedef` 的结束语句，而不必添加空格。

```c++
typedef std::map<int, std::map <int, std::map <int, int> > > cpp98LongTypedef;
typedef std::map<int, std::map <int, std::map <int, int>>>   cpp11LongTypedef;
```

#### 引用限定成员函数（Ref-qualified member functions）

成员函数现在可以根据 *this* 是 lvalue 还是 rvalue 引用来进行限定。

```c++
struct Bar {
  // ...
};

struct Foo {
  Bar getBar() & { return bar; }
  Bar getBar() const& { return bar; }
  Bar getBar() && { return std::move(bar); }
private:
  Bar bar;
};

Foo foo{};
Bar bar = foo.getBar(); // 调用 `Bar getBar() &`

const Foo foo2{};
Bar bar2 = foo2.getBar(); // 调用 `Bar Foo::getBar() const&`

Foo{}.getBar(); // 调用 `Bar Foo::getBar() &&`
std::move(foo).getBar(); // 调用 `Bar Foo::getBar() &&`

std::move(foo2).getBar(); // 调用 `Bar Foo::getBar() const&&`
```

#### 尾置返回类型（Trailing return types）

C++11 允许函数和 lambda 使用替代语法来指定其返回类型。

```c++
int f() {
  return 123;
}
// vs.
auto f() -> int {
  return 123;
}

auto g = []() -> int {
  return 123;
};
```

此特性在某些返回类型无法解析时特别有用：

```c++
// 注意：这段代码无法编译！
template <typename T, typename U>
decltype(a + b) add(T a, U b) {
    return a + b;
}

// 尾置返回类型允许这样：
template <typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}
```

在 C++14 中，可以改用 [decltype(auto) (C++14)](#decltypeauto)。
#### noexcept 说明符（noexcept specifier）

`noexcept` 说明符用于指定一个函数是否可能抛出异常。它是 `throw()` 的改进版本。

```c++
void func1() noexcept;        // 不会抛出异常
void func2() noexcept(true);  // 不会抛出异常
void func3() throw();         // 不会抛出异常

void func4() noexcept(false); // 可能会抛出异常
```

不抛出异常的函数可以调用可能抛出异常的函数。当异常被抛出并且在寻找处理程序时遇到不抛出异常的函数的外部块，函数 `std::terminate` 会被调用。

```c++
extern void f();  // 可能抛出异常
void g() noexcept {
    f();          // 有效，即使 f 抛出异常
    throw 42;     // 有效，实际上是调用 std::terminate
}
```

#### char32_t 和 char16_t

提供了表示 UTF-8 字符串的标准类型。

```c++
char32_t utf8_str[] = U"\u0123";
char16_t utf8_str[] = u"\u0123";
```

#### 原始字符串字面量（Raw string literals）

C++11 引入了一种新的声明字符串字面量的方式，称为“原始字符串字面量”。从转义序列（如制表符、换行符、单个反斜杠等）发出的字符可以以原始方式输入，同时保持格式。这对于书写可能包含大量引号或特殊格式的文学文本特别有用。这样可以使字符串字面量更易读和维护。

原始字符串字面量使用以下语法声明：

```python
R"delimiter(raw_characters)delimiter"
```

其中：

- `delimiter` 是一个可选的字符序列，由除了括号、反斜杠和空格以外的任何源字符组成。
- `raw_characters` 是任何原始字符序列；不得包含结束序列 `")delimiter"`。

示例：

```cpp
// msg1 和 msg2 是等价的。
const char* msg1 = "\nHello,\n\tworld!\n";
const char* msg2 = R"(
Hello,
	world!
)";
```

### C++11 标准库特性

#### std::move

`std::move` 表明传递给它的对象可以将其资源转移。使用已被移动的对象时应谨慎，因为它们可能会处于未指定的状态（参见：[What can I do with a moved-from object?](https://stackoverflow.com/questions/7027523/what-can-i-do-with-a-moved-from-object)）。

`std::move` 的定义（执行移动实际上只是将其转换为右值引用）：

```c++
template <typename T>
typename remove_reference<T>::type&& move(T&& arg) {
  return static_cast<typename remove_reference<T>::type&&>(arg);
}
```

转移 `std::unique_ptr`：

```c++
std::unique_ptr<int> p1 {new int{0}};  // 实际上，使用 std::make_unique
std::unique_ptr<int> p2 = p1; // 错误 -- 不能复制唯一指针
std::unique_ptr<int> p3 = std::move(p1); // 将 `p1` 移动到 `p3`
                                         // 现在无法解引用 `p1` 持有的对象
```

#### std::forward

返回传递给它的参数，同时保持其值类别和 cv-qualifiers。对于通用代码和工厂非常有用。与 [`forwarding references`](#forwarding-references) 一起使用。

`std::forward` 的定义：

```c++
template <typename T>
T&& forward(typename remove_reference<T>::type& arg) {
  return static_cast<T&&>(arg);
}
```

一个 `wrapper` 函数的示例，它只是将其他 `A` 对象转发到新 `A` 对象的拷贝或移动构造函数：

```c++
struct A {
  A() = default;
  A(const A& o) { std::cout << "copied" << std::endl; }
  A(A&& o) { std::cout << "moved" << std::endl; }
};

template <typename T>
A wrapper(T&& arg) {
  return A{std::forward<T>(arg)};
}

wrapper(A{}); // 移动
A a;
wrapper(a); // 复制
wrapper(std::move(a)); // 移动
```

另见：[`forwarding references`](#forwarding-references)，[`rvalue references`](#rvalue-references)。

#### std::thread

`std::thread` 库提供了一种标准方式来控制线程，例如创建和终止它们。在下面的示例中，多个线程被创建来进行不同的计算，然后程序等待所有线程完成。

```c++
void foo(bool clause) { /* 做一些事情... */ }

std::vector<std::thread> threadsVector;
threadsVector.emplace_back([]() {
  // 将要调用的 Lambda 函数
});
threadsVector.emplace_back(foo, true);  // 线程将运行 foo(true)
for (auto& thread : threadsVector) {
  thread.join(); // 等待线程完成
}
```

#### std::to_string

将一个数字参数转换为 `std::string`。

```c++
std::to_string(1.2); // == "1.2"
std::to_string(123); // == "123"
```

#### 类型特性（Type traits）

类型特性定义了一个基于模板的编译时接口，用于查询或修改类型的属性。

```c++
static_assert(std::is_integral<int>::value);
static_assert(std::is_same<int, int>::value);
static_assert(std::is_same<std::conditional<true, int, double>::type, int>::value);
```

#### 智能指针（Smart pointers）

C++11 引入了新的智能指针：`std::unique_ptr`，`std::shared_ptr`，`std::weak_ptr`。`std::auto_ptr` 现在被弃用，并最终在 C++17 中移除。

`std::unique_ptr` 是一个不可复制、可移动的指针，它管理自己的堆分配内存。**注意：建议使用 `std::make_X` 辅助函数，而不是使用构造函数。有关更多信息，请参见 [`std::make_unique` (c++14)](#stdmake_unique) 和 [`std::make_shared` (C++11)](#stdmake_shared) 的部分。**

```c++
std::unique_ptr<Foo> p1 { new Foo{} };  // `p1` 拥有 `Foo`
if (p1) {
  p1->bar();
}

{
  std::unique_ptr<Foo> p2 {std::move(p1)};  // 现在 `p2` 拥有 `Foo`
  f(*p2);

  p1 = std::move(p2);  // 所有权返回到 `p1` -- `p2` 被销毁
}

if (p1) {
  p1->bar();
}
// 当 `p1` 超出作用域时，`Foo` 实例被销毁
```

`std::shared_ptr` 是一种智能指针，管理一个在多个所有者之间共享的资源。共享指针持有一个*控制块*，控制块有几个组件，如托管对象和引用计数器。所有控制块访问都是线程安全的，但操作托管对象本身*不是*线程安全的。

```c++
void foo(std::shared_ptr<T> t) {
  // 对 `t` 做一些操作...
}

void bar(std::shared_ptr<T> t) {
  // 对 `t` 做一些操作...
}

void baz(std::shared_ptr<T> t) {
  // 对 `t` 做一些操作...
}

std::shared_ptr<T> p1 {new T{}};
// 也许这些操作发生在其他线程中？
foo(p1);
bar(p1);
baz(p1);
```

#### std::chrono

`chrono` 库包含一组处理*持续时间*，*时钟*和*时间点*的实用函数和类型。此库的一个用例是基准测试代码：

```c++
std::chrono::time_point<std::chrono::steady_clock> start, end;
start = std::chrono::steady_clock::now();
// 一些计算...
end = std::chrono::steady_clock::now();

std::chrono::duration<double> elapsed_seconds = end - start;
double t = elapsed_seconds.count(); // t 秒，表示为 `double`
```

#### 元组（Tuples）

元组是一个固定大小的异质值集合。可以通过解包 [`std::tie`](#stdtie) 或使用 `std::get` 访问 `std::tuple` 的元素。

```c++
// `playerProfile` 的类型是 `std::tuple<int, const char*, const char*>`。
auto playerProfile = std::make_tuple(51, "Frans Nielsen", "NYI");
std::get<0>(playerProfile); // 51
std::get<1>(playerProfile); // "Frans Nielsen"
std::get<2>(playerProfile); // "NYI"
```

#### std::tie

创建一个包含左值引用的元组。对 `std::pair` 和 `std::tuple` 对象的解包很有用。使用 `std::ignore` 作为被忽略值的占位符。在 C++17 中，应该使用结构化绑定。

```c++
// 使用元组...
std::string playerName;
std::tie(std::ignore, playerName, std::ignore) = std::make_tuple(91, "John Tavares", "NYI");

// 使用对...
std::string yes, no;
std::tie(yes, no) = std::make_pair("yes", "no");
```

#### std::array

`std::array` 是基于 C 风格数组构建的容器。支持常见的容器操作，如排序。

```c++
std::array<int, 3> a = {2, 1, 3};
std::sort(a.begin(), a.end()); // a == { 1, 2, 3 }
for (int& x : a) x *= 2; // a == { 2, 4, 6 }
```

#### 无序容器（Unordered containers）

这些容器保持平均常数时间复杂度的搜索、插入和删除操作。为了实现常数时间复杂度，通过将元素哈希到桶中来牺牲顺序以换取速度。无序容器有四种：

- `unordered_set`
- `unordered_multiset`
- `unordered_map`
- `unordered_multimap`

#### std::make_shared

`std::make_shared` 是创建 `std::shared_ptr` 实例的推荐方法，原因如下：

- 避免使用 `new` 操作符。
- 避免在指定指针所持有的底层类型时重复代码。
- 提供异常安全性。假设我们像这样调用函数 `foo`：

```c++
foo(std::shared_ptr<T>{new T{}}, function_that_throws(), std::shared_ptr<T>{new T{}});
```

编译器可以先调用 `new T{}`，然后调用 `function_that_throws()`，然后是... 由于我们在第一次构造 `T` 时在堆上分配了数据，我们在这里引入了内存泄漏。使用 `std::make_shared`，我们获得了异常安全性：

```c++
foo(std::make_shared<T>(), function_that_throws(), std::make_shared<T>());
```

- 避免进行两次分配。当调用 `std::shared_ptr{ new T{} }` 时，我们需要为 `T` 分配内存，然后在共享指针中为控制块分配内存。

有关 `std::unique_ptr` 和 `std::shared_ptr` 的更多信息，请参见智能指针的部分。

#### std::ref

`std::ref(val)` 用于创建类型为 `std::reference_wrapper` 的对象，该对象持有 `val` 的引用。在通常的引用传递使用 `&` 时编译失败或由于类型推断而丢失 `&` 的情况下使用。`std::cref` 类似，但创建的引用包装器持有 `val` 的常量引用。

```c++
// 创建一个容器来存储对象的引用。
auto val = 99;
auto _ref = std::ref(val);
_ref++;
auto _cref = std::cref(val);
// _cref++ 不会编译
std::vector<std::reference_wrapper<int>>vec; // vector<int&>vec 不会编译
vec.push_back(_ref); // vec.push_back(&i) 不会编译
cout << val << endl; // 打印 100
cout << vec[0] << endl; // 打印 100
cout << _cref; // 打印 100
```

#### 内存模型（Memory model）

C++11 引入了 C++ 的内存模型，这意味着库支持线程和原子操作。其中一些操作包括（但不限于）原子加载/存储、比较并交换、原子标志、promise、future、锁和条件变量。

参见：[`std::thread`](#stdthread)

#### std::async

`std::async` 以异步或延迟评估的方式运行给定的函数，然后返回一个 `std::future`，该 `std::future` 持有该函数调用的结果。

第一个参数是策略，可以是：

1. `std::launch::async | std::launch::deferred` 由实现决定是否执行异步执行或延迟评估。
2. `std::launch::async` 在新线程上运行可调用对象。
3. `std::launch::deferred` 在当前线程上执行延迟评估。

```c++
int foo() {
  /* 在这里做一些事情，然后返回结果。 */
  return 1000;
}

auto handle = std::async(std::launch::async, foo);  // 创建一个异步任务
auto result = handle.get();  // 等待结果
```

#### std::begin/end

`std::begin` 和 `std::end` 自由函数被添加用于通用地返回容器的开始和结束迭代器。这些函数也适用于没有 `begin` 和 `end` 成员函数的原始数组。

```c++
template <typename T>
int CountTwos(const T& container) {
  return std::count_if(std::begin(container), std::end(container), [](int item) {
    return item == 2;
  });
}

std::vector<int> vec = {2, 2, 43, 435, 4543, 534};
int arr[8] = {2, 43, 45, 435, 32, 32, 32, 32};
auto a = CountTwos(vec); // 2
auto b = CountTwos(arr);  // 1
```



## C++14

### C++14 语言特性

#### 二进制字面量（Binary literals）

二进制字面量提供了一种方便的方式来表示以 2 为基数的数字。可以使用 `'` 分隔数字。

```c++
0b110 // == 6
0b1111'1111 // == 255
```

#### 泛型 lambda 表达式（Generic lambda expressions）

C++14 允许在参数列表中使用 `auto` 类型说明符，从而启用多态 lambda。

```c++
auto identity = [](auto x) { return x; };
int three = identity(3); // == 3
std::string foo = identity("foo"); // == "foo"
```

#### Lambda 捕获初始化器（Lambda capture initializers）

这允许使用任意表达式初始化 lambda 捕获。赋予捕获值的名称不需要与封闭范围内的任何变量相关，并且会在 lambda 内部引入一个新名称。初始化表达式在 lambda *创建*时求值（而不是在*调用*时求值）。

```c++
int factory(int i) { return i * 10; }
auto f = [x = factory(2)] { return x; }; // 返回 20

auto generator = [x = 0] () mutable {
  // 如果没有 'mutable' 则无法编译，因为我们在每次调用时都在修改 x
  return x++;
};
auto a = generator(); // == 0
auto b = generator(); // == 1
auto c = generator(); // == 2
```

由于现在可以将值*移动*（或*转发*）到 lambda 中，这些值以前只能通过复制或引用捕获，我们现在可以在 lambda 中按值捕获仅能移动的类型。请注意，在下面的示例中，`task2` 的捕获列表中的 `p` 是 lambda 内部的一个新变量，而不是指原始的 `p`。

```c++
auto p = std::make_unique<int>(1);

auto task1 = [=] { *p = 5; }; // 错误：std::unique_ptr 不能被复制
// vs.
auto task2 = [p = std::move(p)] { *p = 5; }; // 正确：p 通过移动构造到闭包对象中
// 原始的 p 在创建 task2 后为空
```

使用这种方式，引用捕获的名称可以与被引用的变量不同。

```c++
auto x = 1;
auto f = [&r = x, x = x * 10] {
  ++r;
  return r + x;
};
f(); // 将 x 设置为 2 并返回 12
```

#### 返回类型推导（Return type deduction）

在 C++14 中使用 `auto` 返回类型，编译器会尝试为你推导类型。对于 lambda，现在可以使用 `auto` 推导其返回类型，这使得返回推导出的引用或右值引用成为可能。

```c++
// 推导返回类型为 `int`。
auto f(int i) {
 return i;
}
```

```c++
template <typename T>
auto& f(T& t) {
  return t;
}

// 返回推导出的类型的引用。
auto g = [](auto& x) -> auto& { return f(x); };
int y = 123;
int& z = g(y); // 引用 `y`
```

#### decltype(auto)

`decltype(auto)` 类型说明符也像 `auto` 一样推导类型。然而，它在保留引用和 cv 限定符的同时推导返回类型，而 `auto` 不会。

```c++
const int x = 0;
auto x1 = x; // int
decltype(auto) x2 = x; // const int
int y = 0;
int& y1 = y;
auto y2 = y1; // int
decltype(auto) y3 = y1; // int&
int&& z = 0;
auto z1 = std::move(z); // int
decltype(auto) z2 = std::move(z); // int&&
```

```c++
// 注意：在泛型代码中特别有用！

// 返回类型为 `int`。
auto f(const int& i) {
 return i;
}

// 返回类型为 `const int&`。
decltype(auto) g(const int& i) {
 return i;
}

int x = 123;
static_assert(std::is_same<const int&, decltype(f(x))>::value == 0);
static_assert(std::is_same<int, decltype(f(x))>::value == 1);
static_assert(std::is_same<const int&, decltype(g(x))>::value == 1);
```

参见：[`decltype (C++11)`](#decltype).

#### 放宽对 constexpr 函数的限制（Relaxing constraints on constexpr functions）

在 C++11 中，`constexpr` 函数体只能包含非常有限的语法集，包括（但不限于）：`typedef`、`using` 和一个单一的 `return` 语句。在 C++14 中，允许的语法集大大扩展，涵盖了最常见的语法，如 `if` 语句、多个 `return`、循环等。

```c++
constexpr int factorial(int n) {
  if (n <= 1) {
    return 1;
  } else {
    return n * factorial(n - 1);
  }
}
factorial(5); // == 120
```

#### 变量模板（Variable templates）

C++14 允许变量进行模板化：

```c++
template<class T>
constexpr T pi = T(3.1415926535897932385);
template<class T>
constexpr T e  = T(2.7182818284590452353);
```

#### \[\[deprecated\]\] attribute

C++14 引入了 `[[deprecated]]` 属性，以表明某个单元（函数、类等）已不推荐使用，并可能产生编译警告。如果提供了原因，它将包含在警告信息中。

```c++
[[deprecated]]
void old_method();
[[deprecated("Use new_method instead")]]
void legacy_method();
```

### C++14 库特性

#### 标准库类型的用户定义字面量（User-defined literals for standard library types）

新的用户定义字面量适用于标准库类型，包括适用于 `chrono` 和 `basic_string` 的新的内置字面量。这些字面量可以是 `constexpr`，意味着它们可以在编译时使用。对这些字面量的一些使用包括编译时整数解析、二进制字面量和虚数字面量。

```c++
using namespace std::chrono_literals;
auto day = 24h;
day.count(); // == 24
std::chrono::duration_cast<std::chrono::minutes>(day).count(); // == 1440
```

#### 编译时整数序列（Compile-time integer sequences）

类模板 `std::integer_sequence` 表示一个编译时整数序列。在其之上构建了一些辅助类：

- `std::make_integer_sequence<T, N>`：创建一个从 `0` 到 `N - 1` 的序列，类型为 `T`。
- `std::index_sequence_for<T...>`：将模板参数包转换为一个整数序列。

将数组转换为元组：

```c++
template<typename Array, std::size_t... I>
decltype(auto) a2t_impl(const Array& a, std::integer_sequence<std::size_t, I...>) {
  return std::make_tuple(a[I]...);
}

template<typename T, std::size_t N, typename Indices = std::make_index_sequence<N>>
decltype(auto) a2t(const std::array<T, N>& a) {
  return a2t_impl(a, Indices());
}
```

#### std::make_unique

`std::make_unique` 是创建 `std::unique_ptr` 实例的推荐方法，原因如下：

- 避免使用 `new` 操作符。
- 防止在指定指针持有的底层类型时重复代码。
- 最重要的是，它提供了异常安全性。假设我们像这样调用函数 `foo`：

```c++
foo(std::unique_ptr<T>{new T{}}, function_that_throws(), std::unique_ptr<T>{new T{}});
```

编译器可以先调用 `new T{}`，然后调用 `function_that_throws()`，然后是... 由于我们在第一次构造 `T` 时在堆上分配了数据，我们在这里引入了内存泄漏。使用 `std::make_unique`，我们获得了异常安全性：

```c++
foo(std::make_unique<T>(), function_that_throws(), std::make_unique<T>());
```

参见[智能指针（C++11）](#smart-pointers)部分，了解有关 `std::unique_ptr` 和 `std::shared_ptr` 的更多信息。




## C++17

### C++17 语言特性

#### 类模板的模板参数推导（Template argument deduction for class templates）

自动模板参数推导类似于函数的模板参数推导，但现在也包括类构造函数。

```c++
template <typename T = float>
struct MyContainer {
  T val;
  MyContainer() : val{} {}
  MyContainer(T val) : val{val} {}
  // ...
};
MyContainer c1 {1}; // 正确，推导为 MyContainer<int>
MyContainer c2; // 正确，推导为 MyContainer<float>
```

#### 使用 auto 声明非类型模板参数（Declaring non-type template parameters with auto）


遵循 `auto` 的推导规则，同时尊重非类型模板参数列表中的可允许类型(1)，模板参数可以从其参数的类型中推导出来：
{ .annotate }

1. 例如，你不能使用 `double` 作为模板参数类型，这也使得使用 `auto` 进行这种推导是无效的。

```c++
template <auto... seq>
struct my_integer_sequence {
  // 实现 ...
};

// 显式地将类型 `int` 作为模板参数传递。
auto seq = std::integer_sequence<int, 0, 1, 2>();
// 类型被推导为 `int`。
auto seq2 = my_integer_sequence<0, 1, 2>();
```

#### 折叠表达式（Folding expressions）

折叠表达式通过一个二元运算符对模板参数包进行折叠。

- 形如 `(... op e)` 或 `(e op ...)` 的表达式，其中 `op` 是一个折叠操作符，`e` 是一个未展开的参数包，称为*一元折叠*。
- 形如 `(e1 op ... op e2)` 的表达式，其中 `op` 是折叠操作符，称为*二元折叠*。`e1` 或 `e2` 中的一个是未展开的参数包，但不能是两个都未展开。

```c++
template <typename... Args>
bool logicalAnd(Args... args) {
    // 二元折叠。
    return (true && ... && args);
}
bool b = true;
bool& b2 = b;
logicalAnd(b, b2, true); // == true
```

```c++
template <typename... Args>
auto sum(Args... args) {
    // 一元折叠。
    return (... + args);
}
sum(1.0, 2.0f, 3); // == 6.0
```

#### 使用大括号初始化列表时 auto 的新规则（New rules for auto deduction from braced-init-list）

使用统一初始化语法时，`auto` 推导的更改。以前，`auto x {3};` 推导为 `std::initializer_list<int>`，现在推导为 `int`。

```c++
auto x1 {1, 2, 3}; // 错误：不是单个元素
auto x2 = {1, 2, 3}; // x2 是 std::initializer_list<int>
auto x3 {3}; // x3 是 int
auto x4 {3.0}; // x4 是 double
```

#### constexpr lambda 表达式（constexpr lambda）

使用 `constexpr` 的编译时 lambda 表达式。

```c++
auto identity = [](int n) constexpr { return n; };
static_assert(identity(123) == 123);
```

```c++
constexpr auto add = [](int x, int y) {
  auto L = [=] { return x; };
  auto R = [=] { return y; };
  return [=] { return L() + R(); };
};

static_assert(add(1, 2)() == 3);
```

```c++
constexpr int addOne(int n) {
  return [n] { return n + 1; }();
}

static_assert(addOne(1) == 2);
```

#### Lambda 捕获 this 的值（Lambda capture this by value）

在 lambda 的环境中捕获 `this` 以前只能通过引用来捕获。一个示例是异步代码使用回调函数，这些回调函数要求对象在其生命周期内可用，并且可能超出其生命周期。`*this`（C++17）现在将创建当前对象的副本，而 `this`（C++11）继续通过引用捕获。

```c++
struct MyObj {
  int value {123};
  auto getValueCopy() {
    return [*this] { return value; };
  }
  auto getValueRef() {
    return [this] { return value; };
  }
};
MyObj mo;
auto valueCopy = mo.getValueCopy();
auto valueRef = mo.getValueRef();
mo.value = 321;
valueCopy(); // 123
valueRef(); // 321
```

#### 内联变量（Inline variables）

`inline` 说明符可以应用于变量以及函数。一个被声明为 `inline` 的变量与被声明为 `inline` 的函数具有相同的语义。

```c++
// 使用 compiler explorer 的反汇编示例。
struct S { int x; };
inline S x1 = S{321}; // mov esi, dword ptr [x1]
                      // x1: .long 321

S x2 = S{123};        // mov eax, dword ptr [.L_ZZ4mainE2x2]
                      // mov dword ptr [rbp - 8], eax
                      // .L_ZZ4mainE2x2: .long 123
```

它还可以用于声明和定义静态成员变量，使其不需要在源文件中初始化。

```c++
struct S {
  S() : id{count++} {}
  ~S() { count--; }
  int id;
  static inline int count{0}; // 在类内声明并初始化 count 为 0
};
```

#### 嵌套命名空间（Nested namespaces）

使用命名空间解析操作符来创建嵌套的命名空间定义。

```c++
namespace A {
  namespace B {
    namespace C {
      int i;
    }
  }
}
```

上面的代码可以写成：

```c++
namespace A::B::C {
  int i;
}
```tu

#### 结构化绑定（Structured bindings）

提议用于解构初始化，这允许编写 `auto [ x, y, z ] = expr;` 这样的代码，其中 `expr` 的类型是一个元组样对象，其元素将绑定到变量 `x`、`y` 和 `z`（此构造声明）。*元组样对象*包括 `std::tuple`，`std::pair`，`std::array` 和聚合结构。

```c++
using Coordinate = std::pair<int, int>;
Coordinate origin() {
  return Coordinate{0, 0};
}

const auto [ x, y ] = origin();
x; // == 0
y; // == 0
```

```c++
std::unordered_map<std::string, int> mapping {
  {"a", 1},
  {"b", 2},
  {"c", 3}
};

// 通过引用解构。
for (const auto& [key, value] : mapping) {
  // 对键和值进行操作
}
```

#### 带初始化器的选择语句（Selection statements with initializer）

`if` 和 `switch` 语句的新版本，这些版本简化了常见的代码模式并帮助用户保持紧凑的作用域。

```c++
{
  std::lock_guard<std::mutex> lk(mx);
  if (v.empty()) v.push_back(val);
}
// 对比：
if (std::lock_guard<std::mutex> lk(mx); v.empty()) {
  v.push_back(val);
}
```

```c++
Foo gadget(args);
switch (auto s = gadget.status()) {
  case OK: gadget.zip(); break;
  case Bad: throw BadFoo(s.message());
}
// 对比：
switch (Foo gadget(args); auto s = gadget.status()) {
  case OK: gadget.zip(); break;
  case Bad: throw BadFoo(s.message());
}
```

#### constexpr if

编写根据编译时条件实例化的代码。

```c++
template <typename T>
constexpr bool isIntegral() {
  if constexpr (std::is_integral<T>::value) {
    return true;
  } else {
    return false;
  }
}
static_assert(isIntegral<int>() == true);
static_assert(isIntegral<char>() == true);
static_assert(isIntegral<double>() == false);
struct S {};
static_assert(isIntegral<S>() == false);
```

#### UTF-8 字符字面量（UTF-8 character literals）

以 `u8` 开头的字符字面量是 `char` 类型的字符字面量。UTF-8 字符字面量的值等于其 ISO 10646 代码点值。

```c++
char x = u8'x';
```

#### 枚举的直接列表初始化（Direct list initialization of enums）

枚举现在可以使用大括号语法初始化。

```c++
enum byte : unsigned char {};
byte b {0}; // 正确
byte c {-1}; // 错误
byte d = byte{1}; // 正确
byte e = byte{256}; // 错误
```

#### \[\[fallthrough\]\], \[\[nodiscard\]\], \[\[maybe_unused\]\] attributes

C++17 引入了三个新的属性：`[[fallthrough]]`、`[[nodiscard]]` 和 `[[maybe_unused]]`。

- `[[fallthrough]]` 表示在 switch 语句中进行穿透是有意的行为。此属性只能用于 switch 语句中，必须放在下一个 case/default 标签之前。

```c++
switch (n) {
  case 1: 
    // ...
    [[fallthrough]];
  case 2:
    // ...
    break;
  case 3:
    // ...
    [[fallthrough]];
  default:
    // ...
}
```

- `[[nodiscard]]` 当一个函数或类具有此属性且其返回值被丢弃时，会发出警告。

```c++
[[nodiscard]] bool do_something() {
  return is_success; // true 表示成功，false 表示失败
}

do_something(); // 警告: 忽略了 'bool do_something()' 的返回值，
                // 声明时带有 'nodiscard' 属性
```

```c++
// 只有当 `error_info` 被按值返回时才发出警告。
struct [[nodiscard]] error_info {
  // ...
};

error_info do_something() {
  error_info ei;
  // ...
  return ei;
}

do_something(); // 警告: 忽略了类型 'error_info' 的返回值，
                // 声明时带有 'nodiscard' 属性
```

- `[[maybe_unused]]` 表示变量或参数可能未使用且这是有意的。

```c++
void my_callback(std::string msg, [[maybe_unused]] bool error) {
  // 不关心 `msg` 是否为错误消息，只记录它。
  log(msg);
}
```

#### \_\_has\_include

`__has_include (operand)` 运算符可以在 `#if` 和 `#elif` 表达式中使用，以检查头文件或源文件（`operand`）是否可供包含。

一个使用案例是使用两种工作方式相同的库，如果系统上找不到首选的库，则使用备份/实验性库。

```c++
##ifdef __has_include
##  if __has_include(<optional>)
##    include <optional>
##    define have_optional 1
##  elif __has_include(<experimental/optional>)
##    include <experimental/optional>
##    define have_optional 1
##    define experimental_optional
##  else
##    define have_optional 0
##  endif
##endif
```

它还可用于在不同平台上以不同名称或位置存在的头文件而无需知道程序运行的平台，例如 OpenGL 头文件在 macOS 上位于 `OpenGL\` 目录，而在其他平台上位于 `GL\` 目录。

```c++
##ifdef __has_include
##  if __has_include(<OpenGL/gl.h>)
##    include <OpenGL/gl.h>
##    include <OpenGL/glu.h>
##  elif __has_include(<GL/gl.h>)
##    include <GL/gl.h>
##    include <GL/glu.h>
##  else
##    error No suitable OpenGL headers found.
## endif
##endif
```

#### 类模板参数推导（Class template argument deduction）

*类模板参数推导*（CTAD）允许编译器从构造函数参数中推导模板参数。

```c++
std::vector v{ 1, 2, 3 }; // 推导为 std::vector<int>

std::mutex mtx;
auto lck = std::lock_guard{ mtx }; // 推导为 std::lock_guard<std::mutex>

auto p = new std::pair{ 1.0, 2.0 }; // 推导为 std::pair<double, double>
```

对于用户定义类型，如果适用，*推导指南*可以用来指导编译器如何推导模板参数：

```c++
template <typename T>
struct container {
  container(T t) {}

  template <typename Iter>
  container(Iter beg, Iter end);
};

// 推导指南
template <typename Iter>
container(Iter b, Iter e) -> container<typename std::iterator_traits<Iter>::value_type>;

container a{ 7 }; // 正确: 推导为 container<int>

std::vector<double> v{ 1.0, 2.0, 3.0 };
auto b = container{ v.begin(), v.end() }; // 正确: 推导为 container<double>

container c{ 5, 6 }; // 错误: std::iterator_traits<int>::value_type 不是一种类型
```

### C++17 库功能

#### std::variant

类模板 `std::variant` 表示一个类型安全的 `union`。`std::variant` 的实例在任何给定时间内保存其备选类型之一的值（也可能无值）。

```c++
std::variant<int, double> v{ 12 };
std::get<int>(v); // == 12
std::get<0>(v); // == 12
v = 12.0;
std::get<double>(v); // == 12.0
std::get<1>(v); // == 12.0
```

#### std::optional

类模板 `std::optional` 管理一个可选的包含值，即一个值可能存在也可能不存在。`optional` 的一个常见用例是可能失败的函数的返回值。

```c++
std::optional<std::string> create(bool b) {
  if (b) {
    return "Godzilla";
  } else {
    return {};
  }
}

create(false).value_or("empty"); // == "empty"
create(true).value(); // == "Godzilla"
// 返回 optional 的工厂函数可以用作 while 和 if 的条件
if (auto str = create(true)) {
  // ...
}
```

#### std::any

一个类型安全的单值容器，可以容纳任何类型的值。

```c++
std::any x {5};
x.has_value() // == true
std::any_cast<int>(x) // == 5
std::any_cast<int&>(x) = 10;
std::any_cast<int>(x) // == 10
```

#### std::string_view

一个非拥有的字符串引用。对于提供字符串之上的抽象（例如解析）很有用。

```c++
// 常规字符串。
std::string_view cppstr {"foo"};
// 宽字符串。
std::wstring_view wcstr_v {L"baz"};
// 字符数组。
char array[3] = {'b', 'a', 'r'};
std::string_view array_v(array, std::size(array));
```

```c++
std::string str {"   trim me"};
std::string_view v {str};
v.remove_prefix(std::min(v.find_first_not_of(" "), v.size()));
str; //  == "   trim me"
v; // == "trim me"
```

#### std::invoke

用参数调用一个 `Callable` 对象。*可调用*对象的例子有 `std::function` 或 lambdas；可以像常规函数一样调用的对象。

```c++
template <typename Callable>
class Proxy {
  Callable c_;

public:
  Proxy(Callable c) : c_{ std::move(c) } {}

  template <typename... Args>
  decltype(auto) operator()(Args&&... args) {
    // ...
    return std::invoke(c_, std::forward<Args>(args)...);
  }
};

const auto add = [](int x, int y) { return x + y; };
Proxy p{ add };
p(1, 2); // == 3
```

#### std::apply

用一个参数元组调用一个 `Callable` 对象。

```c++
auto add = [](int x, int y) {
  return x + y;
};
std::apply(add, std::make_tuple(1, 2)); // == 3
```

#### std::filesystem

新的 `std::filesystem` 库提供了操作文件、目录和文件系统中路径的标准方式。

在这里，如果有可用的空间，将一个大文件复制到临时路径：
```c++
const auto bigFilePath {"bigFileToCopy"};
if (std::filesystem::exists(bigFilePath)) {
  const auto bigFileSize {std::filesystem::file_size(bigFilePath)};
  std::filesystem::path tmpPath {"/tmp"};
  if (std::filesystem::space(tmpPath).available > bigFileSize) {
    std::filesystem::create_directory(tmpPath.append("example"));
    std::filesystem::copy_file(bigFilePath, tmpPath.append("newFile"));
  }
}
```

#### std::byte

新的 `std::byte` 类型提供了表示数据为字节的标准方式。使用 `std::byte` 优于 `char` 或 `unsigned char` 的好处在于它不是字符类型，也不是算术类型；而且只有位运算符重载。

```c++
std::byte a {0};
std::byte b {0xFF};
int i = std::to_integer<int>(b); // 0xFF
std::byte c = a & b;
int j = std::to_integer<int>(c); // 0
```

注意，`std::byte` 只是一个枚举；并且由于[枚举的直接列表初始化](#direct-list-initialization-of-enums)的支持，使得枚举的花括号初始化变得可能。

#### 映射和集合的拼接（Splicing for maps and sets）

移动节点和合并容器而无需昂贵的复制、移动或堆分配/释放开销。

将元素从一个 map 移动到另一个 map：

```c++
std::map<int, string> src {{1, "one"}, {2, "two"}, {3, "buckle my shoe"}};
std::map<int, string> dst {{3, "three"}};
dst.insert(src.extract(src.find(1))); // 将 { 1, "one" } 从 `src` 中廉价地移除并插入到 `dst` 中。
dst.insert(src.extract(2)); // 将 { 2, "two" } 从 `src` 中廉价地移除并插入到 `dst` 中。
// dst == { { 1, "one" }, { 2, "two" }, { 3, "three" } };
```

插入整个集合：

```c++
std::set<int> src {1, 3, 5};
std::set<int> dst {2, 4, 5};
dst.merge(src);
// src == { 5 }
// dst == { 1, 2, 3, 4, 5 }
```

插入超出容器生命周期的元素：

```c++
auto elementFactory() {
  std::set<...> s;
  s.emplace(...);
  return s.extract(s.begin());
}
s2.insert(elementFactory());
```

更改 map 元素的键：

```c++
std::map<int, string> m {{1, "one"}, {2, "two"}, {3, "three"}};
auto e = m.extract(2);
e.key() = 4;
m.insert(std::move(e));
// m == { { 1, "one" }, { 3, "three" }, { 4, "two" } }
```

#### 并行算法（Parallel algorithms）

许多 STL 算法，例如 `copy`、`find` 和 `sort` 方法，开始支持*并行执行策略*：`seq`、`par` 和 `par_unseq`，它们分别表示“顺序地”、“并行地”和“并行无序地”。

```c++
std::vector<int> longVector;
// 使用并行执行策略查找元素
auto result1 = std::find(std::execution::par, std::begin(longVector), std::end(longVector), 2);
// 使用顺序执行策略排序元素
auto result2 = std::sort(std::execution::seq, std::begin(longVector), std::end(longVector));
```

#### std::sample

在给定序列中采样 n 个元素（无替换），其中每个元素被选中的机会相等。

```c++
const std::string ALLOWED_CHARS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
std::string guid;
// 从 ALLOWED_CHARS 中采样 5 个字符。
std::sample(ALLOWED_CHARS.begin(), ALLOWED_CHARS.end(), std::back_inserter(guid),
  5, std::mt19937{ std::random_device{}() });

std::cout << guid; // 例如 G1fW2
```

#### std::clamp

将给定值限定在下界和上界之间。

```c++
std::clamp(42, -1, 1); // == 1
std::clamp(-42, -1, 1); // == -1
std::clamp(0, -1, 1); // == 0

// `std::clamp` 还接受自定义比较器：
std::clamp(0, -1, 1, std::less<>{}); // == 0
```

#### std::reduce

对给定范围内的元素进行折叠。概念上类似于 `std::accumulate`，但 `std::reduce` 将并行执行折叠。由于折叠是并行完成的，如果您指定一个二元操作，它需要是结合律和交换律。给定的二元操作还不应更改任何元素或使给定范围内的任何迭代器无效。

默认的二元操作是 `std::plus`，初始值为 0。

```c++
const std::array<int, 3> a{ 1, 2, 3 };
std::reduce(std::cbegin(a), std::cend(a)); // == 6
// 使用自定义二元操作：
std::reduce(std::cbegin(a), std::cend(a), 1, std::multiplies<>{}); // == 6
```

此外，您可以为 reducer 指定转换：

```c++
std::transform_reduce(std::cbegin(a), std::cend(a), 0, std::plus<>{}, times_ten); // == 60

const std::array<int, 3> b{ 1, 2, 3 };
const auto product_times_ten = [](const auto a, const auto b) { return a * b * 10; };

std::transform_reduce(std::cbegin(a), std::cend(a), std::cbegin(b), 0, std::plus<>{}, product_times_ten); // == 140
```

#### 前缀和算法（Prefix sum algorithms）

支持前缀和（包括包含和排除扫描）及其转换。

```c++
const std::array<int, 3> a{ 1, 2, 3 };

std::inclusive_scan(std::cbegin(a), std::cend(a),
    std::ostream_iterator<int>{ std::cout, " " }, std::plus<>{}); // 1 3 6

std::exclusive_scan(std::cbegin(a), std::cend(a),
    std::ostream_iterator<int>{ std::cout, " " }, 0, std::plus<>{}); // 0 1 3

const auto times_ten = [](const auto n) { return n * 10; };

std::transform_inclusive_scan(std::cbegin(a), std::cend(a),
    std::ostream_iterator<int>{ std::cout, " " }, std::plus<>{}, times_ten); // 10 30 60

std::transform_exclusive_scan(std::cbegin(a), std::cend(a),
    std::ostream_iterator<int>{ std::cout, " " }, 0, std::plus<>{}, times_ten); // 0 10 30
```

#### 最大公约数和最小公倍数（GCD and LCM）

最大公约数（GCD）和最小公倍数（LCM）。

```c++
const int p = 9;
const int q = 3;
std::gcd(p, q); // == 3
std::lcm(p, q); // == 9
```

#### std::not_fn（std::not_fn）

一个返回给定函数结果的取反值的实用函数。

```c++
const std::ostream_iterator<int> ostream_it{ std::cout, " " };
const auto is_even = [](const auto n) { return n % 2 == 0; };
std::vector<int> v{ 0, 1, 2, 3, 4 };

// 打印所有的偶数。
std::copy_if(std::cbegin(v), std::cend(v), ostream_it, is_even); // 0 2 4
// 打印所有的奇数（非偶数）。
std::copy_if(std::cbegin(v), std::cend(v), ostream_it, std::not_fn(is_even)); // 1 3
```

#### 字符串与数字的相互转换（String conversion to/from numbers）

将整数和浮点数转换为字符串，或反之亦然。转换是非抛出的，不会分配，并且比 C 标准库的等效函数更安全。

用户负责为 `std::to_chars` 分配足够的存储，否则函数会通过设置返回值中的错误码对象来失败。

这些函数允许您选择传递一个基数（默认为十进制）或浮点类型输入的格式说明符。

- `std::to_chars` 返回一个（非 `const`）`char` 指针，该指针是函数写入给定缓冲区的字符串尾后一位置的指针，以及一个错误码对象。
- `std::from_chars` 返回一个 `const char` 指针，成功时等于传递给函数的末尾指针，以及一个错误码对象。

这两个函数返回的错误码对象在成功时等于默认初始化的错误码对象。

将数字 `123` 转换为 `std::string`：

```c++
const int n = 123;

// 可以使用任何容器，字符串，数组等。
std::string str;
str.resize(3); // 为每个数字的 `n` 保持足够的存储

const auto [ ptr, ec ] = std::to_chars(str.data(), str.data() + str.size(), n);

if (ec == std::errc{}) { std::cout << str << std::endl; } // 123
else { /* 处理失败 */ }
```

将值为 `"123"` 的 `std::string` 转换为整数：

```c++
const std::string str{ "123" };
int n;

const auto [ ptr, ec ] = std::from_chars(str.data(), str.data() + str.size(), n);

if (ec == std::errc{}) { std::cout << n << std::endl; } // 123
else { /* 处理失败 */ }
```


## C++20
### C++20 语言特性

#### 协程（Coroutines）

*协程*是可以暂停和恢复其执行的特殊函数。要定义一个协程，函数体中必须包含 `co_return`、`co_await` 或 `co_yield` 关键字。C++20 的协程是无栈的；除非编译器优化掉，否则它们的状态会被分配在堆上。

协程的一个例子是*生成器*函数，它在每次调用时生成一个值：

```c++
generator<int> range(int start, int end) {
  while (start < end) {
    co_yield start;
    start++;
  }

  // 此函数末尾的隐式 co_return:
  // co_return;
}

for (int n : range(0, 10)) {
  std::cout << n << std::endl;
}
```

上面的 `range` 生成器函数从 `start` 开始生成值，直到 `end`（不包括 `end`），每次迭代步骤都会生成当前存储在 `start` 中的值。生成器在每次调用 `range` 时（在这种情况下，每次迭代是在 for 循环中）维护其状态。`co_yield` 接收给定的表达式，生成（即返回）其值，并在该点挂起协程。恢复后，执行将在 `co_yield` 之后继续。

协程的另一个例子是*任务*，它是一种在任务被等待时执行的异步计算：

```c++
task<void> echo(socket s) {
  for (;;) {
    auto data = co_await s.async_read();
    co_await async_write(s, data);
  }

  // 此函数末尾的隐式 co_return:
  // co_return;
}
```

在这个例子中，引入了 `co_await` 关键字。该关键字接收一个表达式，并在等待的对象（在此例中是读或写操作）尚未准备好时挂起执行，否则继续执行。（请注意，在底层，`co_yield` 使用 `co_await`。）

使用任务来惰性计算一个值：

```c++
task<int> calculate_meaning_of_life() {
  co_return 42;
}

auto meaning_of_life = calculate_meaning_of_life();
// ...
co_await meaning_of_life; // == 42
```

**注意**：虽然这些例子演示了如何在基础级别上使用协程，但在代码编译时会有更多的事情发生。这些例子并不意味着完全覆盖 C++20 的协程。由于 `generator` 和 `task` 类尚未由标准库提供，我使用了 `cppcoro` 库来编译这些例子。

#### 概念（Concepts）

*概念*是命名的编译时谓词，用于约束类型。它们采用以下形式：

```
template < template-parameter-list >
concept concept-name = constraint-expression;
```

其中 `constraint-expression` 计算为 `constexpr` 布尔值。*约束*应该建模语义要求，例如一个类型是否为数值或可哈希。如果给定类型不满足其绑定的概念（即 `constraint-expression` 返回 `false`），则会产生编译器错误。因为约束在编译时进行评估，所以它们可以提供更有意义的错误消息和运行时安全性。

```c++
// `T` 不受任何约束的限制。
template <typename T>
concept always_satisfied = true;
// 将 `T` 限制为整型。
template <typename T>
concept integral = std::is_integral_v<T>;
// 将 `T` 限制为既满足 `integral` 约束又是有符号的类型。
template <typename T>
concept signed_integral = integral<T> && std::is_signed_v<T>;
// 将 `T` 限制为既满足 `integral` 约束又不满足 `signed_integral` 约束的类型。
template <typename T>
concept unsigned_integral = integral<T> && !signed_integral<T>;
```

有多种语法形式用于实施概念：

```c++
// 函数参数的形式：
// `T` 是一个受约束的类型模板参数。
template <my_concept T>
void f(T v);

// `T` 是一个受约束的类型模板参数。
template <typename T>
  requires my_concept<T>
void f(T v);

// `T` 是一个受约束的类型模板参数。
template <typename T>
void f(T v) requires my_concept<T>;

// `v` 是一个受约束的推导参数。
void f(my_concept auto v);

// `v` 是一个受约束的非类型模板参数。
template <my_concept auto v>
void g();

// 自动推导变量的形式：
// `foo` 是一个受约束的自动推导值。
my_concept auto foo = ...;

// Lambda 的形式：
// `T` 是一个受约束的类型模板参数。
auto f = []<my_concept T> (T v) {
  // ...
};
// `T` 是一个受约束的类型模板参数。
auto f = []<typename T> requires my_concept<T> (T v) {
  // ...
};
// `T` 是一个受约束的类型模板参数。
auto f = []<typename T> (T v) requires my_concept<T> {
  // ...
};
// `v` 是一个受约束的推导参数。
auto f = [](my_concept auto v) {
  // ...
};
// `v` 是一个受约束的非类型模板参数。
auto g = []<my_concept auto v> () {
  // ...
};
```

`requires` 关键字用于开始 `requires` 子句或 `requires` 表达式：

```c++
template <typename T>
  requires my_concept<T> // `requires` 子句。
void f(T);

template <typename T>
concept callable = requires (T f) { f(); }; // `requires` 表达式。

template <typename T>
  requires requires (T x) { x + x; } // `requires` 子句和表达式在同一行。
T add(T a, T b) {
  return a + b;
}
```

请注意，`requires` 表达式中的参数列表是可选的。`requires` 表达式中的每个要求都是以下之一：

- **简单要求**：断言给定的表达式是有效的。

```c++
template <typename T>
concept callable = requires (T f) { f(); };
```

- **类型要求**：由 `typename` 关键字后跟一个类型名称表示，断言给定的类型名称是有效的。

```c++
struct foo {
  int foo;
};

struct bar {
  using value = int;
  value data;
};

struct baz {
  using value = int;
  value data;
};

// 使用 SFINAE，在 `T` 是 `baz` 时启用。
template <typename T, typename = std::enable_if_t<std::is_same_v<T, baz>>>
struct S {};

template <typename T>
using Ref = T&;

template <typename T>
concept C = requires {
                     // 对类型 `T` 的要求：
  typename T::value; // A) 有一个名为 `value` 的内部成员
  typename S<T>;     // B) 必须有一个有效的 `S` 类模板特化
  typename Ref<T>;   // C) 必须是一个有效的别名模板替换
};

template <C T>
void g(T a);

g(foo{}); // 错误：不满足要求 A。
g(bar{}); // 错误：不满足要求 B。
g(baz{}); // 通过。
```

- **复合要求**：大括号中的表达式后面跟随一个返回类型或类型约束。

```c++
template <typename T>
concept C = requires(T x) {
  {*x} -> std::convertible_to<typename T::inner>; // 表达式 `*x` 的类型可转换为 `T::inner`
  {x + 1} -> std::same_as<int>; // 表达式 `x + 1` 满足 `std::same_as<decltype((x + 1))>`
  {x * 1} -> std::convertible_to<T>; // 表达式 `x * 1` 的类型可转换为 `T`
};
```

- **嵌套要求**：由 `requires` 关键字表示，指定附加约束（例如对局部参数的约束）。

```c++
template <typename T>
concept C = requires(T x) {
  requires std::same_as<sizeof(x), size_t>;
};
```

另见：[概念库](#concepts-library)。

#### 指定初始化器（Designated initializers）

C风格的指定初始化器语法。任何未在指定初始化器列表中显式列出的成员字段都会被默认初始化。

```c++
struct A {
  int x;
  int y;
  int z = 123;
};

A a {.x = 1, .z = 2}; // a.x == 1, a.y == 0, a.z == 2
```
#### 模板语法用于 Lambda 表达式（Template syntax for lambdas）

在 Lambda 表达式中使用熟悉的模板语法。

```c++
auto f = []<typename T>(std::vector<T> v) {
  // ...
};
```

#### 带初始化器的基于范围的 for 循环（Range-based for loop with initializer）

此特性简化了常见的代码模式，帮助保持作用域紧凑，并提供了优雅的解决方案来解决常见的生命周期问题。

```c++
for (auto v = std::vector{1, 2, 3}; auto& e : v) {
  std::cout << e;
}
// 打印 "123"
```

#### \[\[likely\]\] and \[\[unlikely\]\] attributes

向优化器提供提示，表明标记的语句具有较高的执行概率。

```c++
switch (n) {
case 1:
  // ...
  break;

[[likely]] case 2:  // n == 2 被认为比其他任何值更可能
  // ...            // 被执行
  break;
}
```

如果一个 likely/unlikely 属性出现在 if 语句的右括号之后，它表示该分支很可能/不太可能执行其子语句（主体）。

```c++
int random = get_random_number_between_x_and_y(0, 3);
if (random > 0) [[likely]] {
  // if 语句的主体
  // ...
}
```

它也可以应用于迭代语句的子语句（主体）。

```c++
while (unlikely_truthy_condition) [[unlikely]] {
  // while 语句的主体
  // ...
}
```

#### 弃用隐式捕获 this（Deprecate implicit capture of this）

隐式捕获 `this` 使用 `[=]` 现在被弃用；建议使用 `[=, this]` 或 `[=, *this]` 显式捕获。

```c++
struct int_value {
  int n = 0;
  auto getter_fn() {
    // 不好：
    // return [=]() { return n; };

    // 好：
    return [=, *this]() { return n; };
  }
};
```

#### 非类型模板参数中的类类型（Class types in non-type template parameters）

现在可以在非类型模板参数中使用类。作为模板参数传递的对象具有 `const T` 类型，其中 `T` 是对象的类型，并且具有静态存储持续时间。

```c++
struct foo {
  foo() = default;
  constexpr foo(int) {}
};

template <foo f>
auto get_foo() {
  return f;
}

get_foo(); // 使用隐式构造函数
get_foo<foo{123}>();
```

#### constexpr 虚函数（constexpr virtual functions）

虚函数现在可以是 `constexpr` 并在编译时进行评估。`constexpr` 虚函数可以覆盖非 `constexpr` 虚函数，反之亦然。

```c++
struct X1 {
  virtual int f() const = 0;
};

struct X2: public X1 {
  constexpr virtual int f() const { return 2; }
};

struct X3: public X2 {
  virtual int f() const { return 3; }
};

struct X4: public X3 {
  constexpr virtual int f() const { return 4; }
};

constexpr X4 x4;
x4.f(); // == 4
```

#### explicit(bool)

在编译时有条件地选择是否将构造函数设为显式。`explicit(true)` 与指定 `explicit` 相同。

```c++
struct foo {
  // 指定非整数类型（字符串、浮点数等）需要显式构造。
  template <typename T>
  explicit(!std::is_integral_v<T>) foo(T) {}
};

foo a = 123; // 正确
foo b = "123"; // 错误：显式构造函数不是候选（explicit 说明符的值为 true）
foo c {"123"}; // 正确
```

#### 立即函数（Immediate functions）

类似于 `constexpr` 函数，但具有 `consteval` 说明符的函数必须产生常量。这些被称为*立即函数*。

```c++
consteval int sqr(int n) {
  return n * n;
}

constexpr int r = sqr(100); // 正确
int x = 100;
int r2 = sqr(x); // 错误：'x' 的值在常量表达式中不可用
                 // 如果 `sqr` 是 `constexpr` 函数则正确
```

#### using enum

将枚举的成员引入作用域以提高可读性。之前：

```c++
enum class rgba_color_channel { red, green, blue, alpha };

std::string_view to_string(rgba_color_channel channel) {
  switch (channel) {
    case rgba_color_channel::red:   return "red";
    case rgba_color_channel::green: return "green";
    case rgba_color_channel::blue:  return "blue";
    case rgba_color_channel::alpha: return "alpha";
  }
}
```

之后：

```c++
enum class rgba_color_channel { red, green, blue, alpha };

std::string_view to_string(rgba_color_channel my_channel) {
  switch (my_channel) {
    using enum rgba_color_channel;
    case red:   return "red";
    case green: return "green";
    case blue:  return "blue";
    case alpha: return "alpha";
  }
}
```

#### Lambda 捕获参数包（Lambda capture of parameter pack）

按值捕获参数包：

```c++
template <typename... Args>
auto f(Args&&... args){
    // 按值：
    return [...args = std::forward<Args>(args)] {
        // ...
    };
}
```

按引用捕获参数包：

```c++
template <typename... Args>
auto f(Args&&... args){
    // 按引用：
    return [&...args = std::forward<Args>(args)] {
        // ...
    };
}
```

#### char8_t

提供了一个用于表示 UTF-8 字符串的标准类型。

```c++
char8_t utf8_str[] = u8"\u0123";
```

#### constinit

`constinit` 说明符要求变量必须在编译时初始化。

```c++
const char* g() { return "动态初始化"; }
constexpr const char* f(bool p) { return p ? "常量初始化器" : g(); }

constinit const char* c = f(true); // 正确
constinit const char* d = g(false); // 错误：`g` 不是 constexpr，所以 `d` 不能在编译时进行评估。
```


### C++20 库特性

#### Concepts library

标准库也提供了一些 concepts 用于构建更复杂的 concepts。一些包括：

**核心语言概念**：

- `same_as`：指定两个类型是相同的。
- `derived_from`：指定一个类型派生自另一个类型。
- `convertible_to`：指定一个类型可以隐式转换为另一个类型。
- `common_with`：指定两个类型共享一个公共类型。
- `integral`：指定一个类型是整数类型。
- `default_constructible`：指定一个类型的对象可以进行默认构造。

**比较概念**：

- `boolean`：指定一个类型可以用于布尔上下文。
- `equality_comparable`：指定 `operator==` 是一个等价关系。

**对象概念**：

- `movable`：指定一个类型的对象可以被移动和交换。
- `copyable`：指定一个类型的对象可以被复制、移动和交换。
- `semiregular`：指定一个类型的对象可以被复制、移动、交换，并进行默认构造。
- `regular`：指定一个类型是 *regular*，即它既是 `semiregular` 也是 `equality_comparable`。

**可调用概念**：

- `invocable`：指定一个可调用类型可以用一组给定的参数类型进行调用。
- `predicate`：指定一个可调用类型是一个布尔谓词。

另见：[概念](#concepts)。

#### 同步缓冲输出流（Synchronized buffered outputstream）

为包装的输出流缓冲输出操作，确保同步（即输出不会交错）。

```c++
std::osyncstream{std::cout} << "The value of x is:" << x << std::endl;
```

#### std::span

`span` 是一个视图（即非拥有的）容器，提供对一组连续元素的边界检查访问。由于视图不拥有其元素，因此构造和复制开销较小——简化地说，视图持有对其数据的引用。与其维护指针/迭代器和长度字段相比，`span` 将这两者包装在一个对象中。

`span` 可以是动态大小或固定大小（称为其 *extent*）。固定大小的 `span` 受益于边界检查。

`span` 不传播 `const`，因此要构造只读 `span`，请使用 `std::span<const T>`。

示例：使用动态大小的 `span` 打印来自各种容器的整数。

```c++
void print_ints(std::span<const int> ints) {
    for (const auto n : ints) {
        std::cout << n << std::endl;
    }
}

print_ints(std::vector{ 1, 2, 3 });
print_ints(std::array<int, 5>{ 1, 2, 3, 4, 5 });

int a[10] = { 0 };
print_ints(a);
// 等等
```

示例：对于与 `span` 的 extent 不匹配的容器，静态大小的 `span` 将无法编译。

```c++
void print_three_ints(std::span<const int, 3> ints) {
    for (const auto n : ints) {
        std::cout << n << std::endl;
    }
}

print_three_ints(std::vector{ 1, 2, 3 }); // 错误
print_three_ints(std::array<int, 5>{ 1, 2, 3, 4, 5 }); // 错误
int a[10] = { 0 };
print_three_ints(a); // 错误

std::array<int, 3> b = { 1, 2, 3 };
print_three_ints(b); // 正确

// 如果需要，可以手动构造 span：
std::vector c{ 1, 2, 3 };
print_three_ints(std::span<const int, 3>{ c.data(), 3 }); // 正确：设置指针和长度字段。
print_three_ints(std::span<const int, 3>{ c.cbegin(), c.cend() }); // 正确：使用迭代器对。
```

#### 位操作（Bit operations）

C++20 提供了一个新的 `<bit>` 头文件，提供了一些位操作，包括 popcount。

```c++
std::popcount(0u); // 0
std::popcount(1u); // 1
std::popcount(0b1111'0000u); // 4
```

#### 数学常数（Math constants）

包括 PI、欧拉数等在内的数学常数在 `<numbers>` 头文件中定义。

```c++
std::numbers::pi; // 3.14159...
std::numbers::e; // 2.71828...
```

#### std::is_constant_evaluated

在编译时上下文中调用时为真值的谓词函数。

```c++
constexpr bool is_compile_time() {
    return std::is_constant_evaluated();
}

constexpr bool a = is_compile_time(); // true
bool b = is_compile_time(); // false
```

#### std::make_shared 支持数组（std::make_shared supports arrays）

```c++
auto p = std::make_shared<int[]>(5); // 指向 `int[5]` 的指针
// 或
auto p = std::make_shared<int[5]>(); // 指向 `int[5]` 的指针
```

#### starts_with 和 ends_with 用于字符串（starts_with and ends_with on strings）

字符串（和字符串视图）现在具有 `starts_with` 和 `ends_with` 成员函数，用于检查字符串是否以给定字符串开始或结束。

```c++
std::string str = "foobar";
str.starts_with("foo"); // true
str.ends_with("baz"); // false
```

#### 检查关联容器是否包含元素（Check if associative container has element）

关联容器，如集合和映射，具有 `contains` 成员函数，可用于替代“查找并检查迭代器结束”习惯用法。

```c++
std::map<int, char> map {{1, 'a'}, {2, 'b'}};
map.contains(2); // true
map.contains(123); // false

std::set<int> set {1, 2, 3};
set.contains(2); // true
```

#### std::bit_cast

一种更安全的将对象从一种类型重新解释为另一种类型的方法。

```c++
float f = 123.0;
int i = std::bit_cast<int>(f);
```

#### std::midpoint

安全地计算两个整数的中点（无溢出）。

```c++
std::midpoint(1, 3); // == 2
```

#### std::to_array

将给定的数组/“似数组”对象转换为 `std::array`。

```c++
std::to_array("foo"); // 返回 `std::array<char, 4>`
std::to_array<int>({1, 2, 3}); // 返回 `std::array<int, 3>`

int a[] = {1, 2, 3};
std::to_array(a); // 返回 `std::array<int, 3>`
```

## 致谢

- [cppreference](http://en.cppreference.com/w/cpp) - 特别有用，用于查找新库特性的示例和文档。
- [C++ Rvalue References Explained](http://thbecker.net/articles/rvalue_references/section_01.html) - 一个很好的介绍，用于理解右值引用、完美转发和移动语义。
- [clang](http://clang.llvm.org/cxx_status.html) 和 [gcc](https://gcc.gnu.org/projects/cxx-status.html) 的标准支持页面。这里还包含了我用来查找特性描述、修复内容和示例的语言/库提案。
- [Compiler explorer](https://godbolt.org/)
- [Scott Meyers' Effective Modern C++](https://www.amazon.com/Effective-Modern-Specific-Ways-Improve/dp/1491903996) - 强烈推荐的书籍！
- [Jason Turner's C++ Weekly](https://www.youtube.com/channel/UCxHAlbZQNFU2LgEtiqd2Maw) - 很好的 C++ 相关视频集合。
- [What can I do with a moved-from object?](http://stackoverflow.com/questions/7027523/what-can-i-do-with-a-moved-from-object)
- [What are some uses of decltype(auto)?](http://stackoverflow.com/questions/24109737/what-are-some-uses-of-decltypeauto)
- 还有许多我忘记的 SO 帖子...

## 作者
Anthony Calandra

## 内容贡献者

参见：[https://github.com/AnthonyCalandra/modern-cpp-features/graphs/contributors](https://github.com/AnthonyCalandra/modern-cpp-features/graphs/contributors)

## 许可证
MIT