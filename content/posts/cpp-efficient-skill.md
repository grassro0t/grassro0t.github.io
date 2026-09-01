---
title: "高效C++系列-重要概念和常用代码"
slug: "cpp-efficient-skill"
date: 2026-08-27T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["语言特性", "c++", "常用代码", "编码风格"]
categories: ["技术笔记"]
summary: "c++重要概念和常用代码总结，包括编程建议、基本概念、分离式编译、编码风格、常用代码等。"
toc: true
comments: true
description: "高效C++-重要概念和常用代码"
---
本部分内容结合了C++ primer与Effective C++总结而成
# 编程建议
- 要明确初始化每一个内置类型的变量
- char要指明是unsigned char，不然是由编译器决定它的类型的
- ==T&&是最强万能绑定可以绑定普通左值、const 左值、右值，支持完美转发，const T&比他差在无法绑定普通左值而且无法分辨const 左值、右值（字面值）==
- 空指针必须使用nullptr
# 基本概念
- 声明式：声明存在，没有具体内容
- 定义式：必须明确定义具体结构
- 初始化：第一次出现进行初始结构设置
- 接口：也是成员函数
- 构造函数：
- 析构函数：
- TR1：提供c++标准库的许多新特性，在std::tr1中
- Boost：一个组织，提供可复审并开源的c++程序库
- 二八定律：20%时间运行80%的普通代码，80%时间运行20%的核心代码（**核心代码设置为inline且不使用虚函数**）
- 运行时类型识别RTTI：运行时获取对象真实类型（基类指针或引用指向派生类），建议直接使用虚函数，不推荐使用这种方法来管理动态绑定，1-typeid(表达式)直接获取真实对象，2-dynamic_cast\<派生类\*\>(基类对象)认定基类为派生类
- 运行时类型：派生类构造基类部分以及之前视为基类，销毁派生类部分以后也视为基类
\# 代码命名规范（Google）
- 基本原则：表意清晰不使用缩写，不同实体格式严格区分，名词命名变量、类型，动词动宾短语命名函数
- 头文件用`.h`，源文件用`.cc`
- 函数参数名`lhs/rhs`，指针名引用名与一般变量名相同
- 注释：
1. `//`：行内解释
2. `/**/`：文档说明、函数功能、代码段屏蔽

| 实体                 | 格式           | 示例                   |
|----------------------|----------------|------------------------|
| 文件名、命名空间     | 小写下划线     | `rpc_client.cc`、`net` |
| 类、枚举、别名       | 大驼峰         | `TaskQueue`            |
| const、constexpr常量 | k大驼峰        | `kDefaultTimeout`      |
| 普通变量             | 小写下划线     | `packet_len`           |
| 类成员变量           | 小写下划线结尾 | `packet_len_`          |
| 普通函数             | 大驼峰         | `SendMessage()`        |
| Get/Set 访问器       | 小写下划线     | `get_packet_len()`     |
| 宏                   | 大写下划线     | `BUFFER_MAX`           |

# 主函数入口

默认返回0（成功），argv\[0\]是null或者程序名，之后是命令行传递的实参，argv\[最后\]是返回值

``` cpp
#include <cstdlib>

int main(int argc,char* argv[]){
    //EXIT_FAILURE;
    int retCode(EXIT_SUCCESS);
    return retCode;
}
```

# ==C++11编码风格==

1.  不要忽视编译器的警告，也不要依赖编译器的警告，高手应当避免不明确行为
2.  初始化：统一优先列表初始化
3.  编译期常量：优先constexpr（替代const），字面值类
4.  数组：优先使用vector
5.  字符串：优先使用string
6.  类型推导：优先使用auto，容器遍历使用`const auto&`，数组或函数类型使用decltype
7.  别名：优先using
8.  右值：短暂使用的多用std::move转右值使用，自定义类主动实现移动成员，std::forward保留当前参数左右值、const等全部特性用于完美转发
9.  循环：优先范围for循环，只读用const &，修改用&，临时使用用&&，循环用!=代替\<
10. lambda表达式：捕获列表最小化，函数体最小化，功能单一简单
11. 函数：形参基础类型值传递，大类统一`const T&`传递，右值或支持移动`T&&`传递，返回值多个使用tuple、pair或struct
12. 类：类成员函数不改动数据成员的一律声明成const，单参数构造函数和类型转换运算符要加explicit
13. 容器：优先使用迭代器
14. 动态内存：变量和数组优先使用unique_ptr，少用裸指针，共享内存使用shared_ptr和weak_ptr，weak_ptr使用前用lock()提升shared_ptr
    \# 分离式编译
    \## 预处理器

- 预处理器变量在预处理阶段处理，名称不会进入记号表，少用
- 头文件管理：`#include`、`#pragma once`
- 调试开关：`#define/#ifndef`
- 定义常量：`#define MAX_SIZE 1024`，推荐用constexpr代替
- 简单函数：`#define SQUARE(x) ((x)*(x))`，推荐用inline函数模板代替

``` cpp
template <typename T>
inline void square(const T &a){
    return x*x;
}
```

## enum hack

- ==类内静态成员是不能类内初始化的必须类外初始化==
- 类内static const部分可以类内初始化部分必须类外初始化，为了避免出错一律类外初始化
- 就想类内初始化static const常量使用可以使用enum hack

``` cpp
class Demo { 
    public: 
        enum { MAX = 1000 }; 
};
```

## 头文件

- 头文件名字是name.h，主要负责变量的声明（一般变量、函数、类、类的友元函数）和类的定义（const变量、constexpr变量、内联函数、constexpr函数、类、模板类的成员方法）
- 头文件名字与类名一致
- 头文件别用`using namespace std`，要用全称
- include必须放在namespace外
- 标准库引用使用`#include <>`
- 自定义头文件引用使用`#include ""`
- 全局const/constexpr常量、全局变量、函数、类、结构体声明
- 类成员变量、成员函数、友元函数声明
- inline/constexpr函数声明定义
- 模板类、模板函数定义（绝大多数情况不用单独写声明）
- 模板类特例化定义
- 模板函数特例化定义（前面必须加inline不然会重复定义）
- 业务很少用模板实例化：如果要用，头文件只能写模板声明
  \## 源文件
- 主要负责变量的定义和类成员的定义，要包含对应头文件
- 可以使用`using namespace std`，放在namespace内
- include必须放在namespace外
- 全局const/constexpr常量、全局变量、函数、结构体定义
- 类成员变量、成员函数、友元函数定义
- 业务很少用模板实例化：如果要用，源文件写模板定义和显式实例化
  \## 功能函数分离
- 类的非成员功能函数根据用途分别安排到工具文件中
  \## 跨文件编译依存最小性
- 重新编译的文件数量尽可能少，反面就是头文件臃肿、互相嵌套 include、大量暴露内部细节
- 头文件前置声明，少include：`#include "Weapon.h"`变成`class Weapon;`
- 头文件多声明，实现和include放在源文件
- 头文件必须定义的内容比如constexpr、inline、模板收敛到少量工具头文件中
- PIMPL隐藏私有成员头文件只暴露接口
  \## PIMPL技巧
  \### handle class（标准 PIMPL）
- 接口 Player.h

``` cpp
#pragma once
// 仅前置声明，不引入任何重型头文件
struct PlayerImpl;

class Player
{
public:
    Player();
    ~Player();
    // 禁止拷贝（可选，按需实现拷贝构造/赋值）
    Player(const Player&) = delete;
    Player& operator=(const Player&) = delete;

    void Attack();
    void SetHp(int hp);
    int GetHp() const;

private:
    // 句柄：仅指针，完整定义放cpp
    PlayerImpl* m_impl;
};
```

- 实现 Player.cpp

``` cpp
#include "Player.h"
#include <vector>
#include "Weapon.h"  // 重型依赖仅在cpp引入

// 完整实现结构体
struct PlayerImpl
{
    int hp = 100;
    std::vector<Weapon> weapons;
};

Player::Player() : m_impl(new PlayerImpl{}) {}

Player::~Player()
{
    delete m_impl;
}

void Player::Attack()
{
    // 访问内部私有数据
    for (auto& w : m_impl->weapons)
    {
        w.Fire();
    }
}

void Player::SetHp(int hp)
{
    m_impl->hp = hp;
}

int Player::GetHp() const
{
    return m_impl->hp;
}
```

### interface class（工厂模式+多态）

- 接口 IPlayer.h

``` cpp
#pragma once
// 纯抽象基类，只有纯虚函数
class IPlayer
{
public:
    virtual ~IPlayer() = default;
    virtual void Attack() = 0;
    virtual void SetHp(int hp) = 0;
    virtual int GetHp() const = 0;
};

// 对外工厂函数，隐藏具体实现类
IPlayer* CreateWarriorPlayer();
IPlayer* CreateMagePlayer();
```

- 实现 PlayerImpl.cpp

``` cpp
#include "IPlayer.h"
#include <vector>
#include "Weapon.h"

// 战士实现
class WarriorPlayer : public IPlayer
{
private:
    int hp = 150;
    std::vector<Weapon> swords;
public:
    void Attack() override
    {
        for (auto& s : swords) s.Fire();
    }
    void SetHp(int hp) override { this->hp = hp; }
    int GetHp() const override { return hp; }
};

// 法师实现
class MagePlayer : public IPlayer
{
private:
    int hp = 80;
    std::vector<Weapon> spells;
public:
    void Attack() override
    {
        for (auto& s : spells) s.Fire();
    }
    void SetHp(int hp) override { this->hp = hp; }
    int GetHp() const override { return hp; }
};

// 工厂函数
IPlayer* CreateWarriorPlayer()
{
    return new WarriorPlayer{};
}

IPlayer* CreateMagePlayer()
{
    return new MagePlayer{};
}
```

# C

## 所有的前缀后缀修饰符

### ==函数==

- 前缀（按顺序）：template → friend/extern → explicit → virtual/static → inline → constexpr/consteval
- 后缀（按顺序）：const/volatile → &/&& → noexcept → override/final
- static与virtual、const、&/&&、override/final冲突（static没有this指针），static与extern冲突（功能相反）
- virtual与constexpr冲突（虚函数不能编译期确定）
- explict一般只用在单参数构造函数或类型转换运算符重载中
  \### 变量
- 前缀：template\<\> → extern/static/thread_local → mutable → constexpr/constinit → const/volatile
- mutable与static、constexpr冲突（功能相反）
- thread_local只能修饰静态生存期变量（全局变量、static变量），常用于线程独立buffer和无锁计数器
  \### 修饰符的声明位置：
- 仅限类内的修饰符：virtual、override/final、explicit、friend、mutable
- 除了以上这些再加一个static，这些修饰符是只需要类内声明类外定义不要重复的

| 修饰符                | 类内声明            | 类外定义          | 关键结论        |
|-----------------------|---------------------|-------------------|-----------------|
| virtual               | ✅ 写               | ❌ 禁止写         | 仅声明保留      |
| static (成员函数)     | ✅ 写               | ❌ 禁止写         | 仅声明保留      |
| override/final        | ✅ 写               | ❌ 禁止写         | 仅声明后缀      |
| explicit              | ✅ 写               | ❌ 禁止写         | 构造 / 转换专用 |
| friend                | ✅ 写               | ❌ 不存在         | 只用于类内声明  |
| inline                | ✅ 写               | ✅ 必须保留       | 两边都带        |
| constexpr/consteval   | ✅ 写               | ✅ 必须保留       | 两边都带        |
| const/&/&&/noexcept   | ✅ 写               | ✅ 必须保留       | 后缀全程一致    |
| template\<\>          | ✅ 写               | ✅ 必须保留       | 模板首尾都要    |
| static (静态成员变量) | ✅ 写               | ✅ 必须保留       | 变量两边都带    |
| const/volatile        | ✅ 写               | ✅ 必须保留       | 变量两边都带    |
| thread_local          | ✅ 写 (static 成员) | ✅ 必须保留       | 变量两边都带    |
| mutable               | ✅ 写 (仅普通成员)  | ------ 无类外定义 | 只在类内        |
| extern                | ❌ 类内不能写       | ✅ 仅类外         | 和类成员无关    |

## 对象初始化

- [2.11 哪些必须初始化？](../面经.md#2.11%20哪些必须初始化？)
- [2.25 初始化类型](../面经.md#2.25%20初始化类型)
- 内置类型对象必须手动初始化避免默认初始化
- 构造函数使用初始化列表初始化成员
- 类成员初始化顺序：全局静态 → 基类静态 → 基类非静态 → 基类构造函数体 → 子类静态 → 子类非静态 → 子类构造函数体
- 跨文件全局静态顺序无定义问题：全局变量如果不在一个文件初始化顺序是不一定的，使用局部static变量替代全局变量来实现规定顺序的初始化
- 延后定义：延后变量的初始化直到它第一次使用，局部变量定义放在分支内满足条件才创建，内置小类型定义放循环内，大类型放循环外
- [2.17 哪些类不能拷贝？哪些可以移动？](../面经.md#2.17%20哪些类不能拷贝？哪些可以移动？)
  \## const对象使用
- 能用const用const
- 函数：返回值尽量用const（阻止向结果赋值）是不是引用见下面单独讨论，参数尽量用const T& param（避免拷贝节省性能，保护实参，能接收右值），内置小类型作为参数可以值传递
- 类：成员函数能const用const，非const函数也能调用const函数，[统一private const工具函数](2.1%20C++编程技巧和常用代码%20*.md#%5Econst-nonconst)
  \### 函数返回引用
- 推荐场景：返回类成员或this指针（无悬空风险）
- 严禁返回：

1.  局部对象引用：空悬指针
2.  堆对象：容易忘了delete

``` cpp
const Rational &operator*(const Rational &lhs, const Rational &rhs)
{
    Rational *result = new Rational(lhs.n * rhs.numerator(), lhs.denominator() * rhs.denominator());
    return *result;
}
Rational w, x, y, z;
//x*y会new一个Rational，x*y*z会再次new一个Rational且丢失x*y的指针，赋值后x*y*z指针也丢失了，两个Rational对象丢失指针永久内存泄漏
w = x * y * z;
```

3.  局部静态对象：可以返回（单例模式），但是多个同时调用容易出错尤其是判断时

``` cpp
const Rational &operator*(const Rational &lhs, const Rational &rhs)
{
    static Rational result;
    result.numerator = lhs.numerator * rhs.numerator;
    result.denominator = lhs.denominator * rhs.denominator; 
    return result;
}
Rational a, b, c, d;
//此判断永远为真
if((a*b) == (c*d))
{
    cout << "a*b == c*d" << endl;
}
else
{
    cout << "a*b != c*d" << endl;
}
```

### bitwise constness和logical constness

- bitwise constness：编译器只查看对象内存有没有被修改，它不管你逻辑上有没有改动

``` cpp
class CTextBlock{
    private:
        char *pText;
        unsigned int length;
    public:
        char &operator[](std::size_t position) const
        {
            return pText[position];
        }
}
const CTextBlock block("hello"); 
block[0] = 'X'; // 合法编译，修改了堆上字符串内容
```

- logical constness（推荐）：结合mutable在内部处理const对象的信息

``` cpp
class CTextBlock
{
private:
    char* pText;
    unsigned int length;

    // mutable：const成员函数内可修改，仅内部缓存，不改变对外可见状态
    mutable std::size_t textLength;
    mutable bool isValid;

    // 私有懒加载：更新缓存长度、标记有效
    void refreshCache() const
    {
        textLength = std::strlen(pText);
        isValid = true;
    }

public:
    CTextBlock(char* str)
        : pText(str), length(std::strlen(str)), textLength(0), isValid(false)
    {}

    // 非const对象：返回可修改引用
    char& operator[](std::size_t position)
    {
        if (!isValid)
            refreshCache(); // mutable变量允许修改
        return pText[position];
    }

    // const对象：返回只读引用，严格逻辑const，外部不能修改文本
    const char& operator[](std::size_t position) const
    {
        if (!isValid)
            refreshCache(); // const函数修改mutable缓存，符合logical constness
        return pText[position];
    }

    // const接口获取缓存长度
    std::size_t getTextLen() const
    {
        if (!isValid)
            refreshCache();
        return textLength;
    }

    // 重置缓存标记（外部只读接口）
    void invalidateCache() const
    {
        isValid = false;
    }
};
```

## static对象使用

- 跨文件全局静态顺序无定义问题
  如果Y先构造就崩溃了

``` cpp
// file1.h
struct X;

// file1.cpp
#include "file1.h"
struct X { X() { cout << "X"; } };
X g_x;

// file2.h
struct Y;

// file2.cpp
#include "file1.h"
#include "file2.h"
struct Y { Y() { cout << g_x; } };
Y g_y;
```

解决方法：封装在函数里返回静态static对象

``` cpp
// file1.h
struct X;
X& getGlobalX();

// file1.cpp
#include "file1.h"
using std::cout;
struct X { X() { cout << "X"; } };
// Meyers单例：局部static，第一次调用才构造，无初始化顺序问题
X& getGlobalX() {
    static X g_x;
    return g_x;
}

// file2.h
struct Y;
Y& getGlobalY();

// file2.cpp
#include "file2.h"
#include "file1.h"
using std::cout;
// Y调用X单例
struct Y { Y() { cout << getGlobalX(); } };
Y& getGlobalY() {
    static Y g_y;
    return g_y;
}
```

## inline使用

- 仅限小型简单高频函数上
- inline函数也可以取地址的，只不过取地址的一瞬间就生成了独立函数实体，所有内联失效
- inline对调试不友好断点失效（因为没有独立函数实体没有函数栈）
- inline只是一个申请，具体还是看编译器怎么处理这个申请
  \## 强制类型转换
- 少用吧，尽量用在工具函数和内部函数中，dynamic_cast更是少用（依赖RTTI），你用了说明违背了多态的理念（不要判断是哪个子类，而是利用虚函数分发实现）
- static_cast：数值、`void*`指针、父子类互转（父类转子类有风险）
- const_cast：增删顶层、底层const和volatile，只不过对底层const和volatile的操作是有较大风险的，日常严禁使用
- dynamic_cast：安全父子类互转（异常抛出或空指针）
- reinterpret_cast：底层二进制重新解释（全手动检查）
  \## 引用折叠
- [2.22 引用折叠](../面经.md#2.22%20引用折叠)
  \## tips
- 成员函数传递本类实参会在类作用域中查找
- 可变形参可以使用可变参数列表和可变参数模板
- 未定义求值顺序时有些运算符使用容易出错：`cout << i << " " << ++i << endl;`，相当于`operator<<( operator<<(cout, a), b )`，函数的左右实参谁先算c++是没有规定的
  \# object-oriented C++
  \## 类的基础设计
- 创建销毁：构造、析构、new运算符、delete运算符
- 初始化赋值：构造、拷贝赋值运算符
- 拷贝移动：拷贝构造、拷贝赋值运算符、移动构造、移动赋值运算符、移动成员函数
- 继承：虚函数、虚析构（有动态创建并销毁子类场景时）
- 类型转换：构造、类型转换运算符
- 访问控制：访问控制符
- 内部工具：private工具函数、using别名
- 合理操作：成员函数、非成员函数
- 一般化：类模板
- 真正需求：is a还是has a，有时一个函数就可以代替整个类
  \### ==拷贝成员==
- 90%业务类一般不用手写拷贝成员（内部内置类型、智能指针），裸指针资源管理类必须手动实现
- 继承相关的类如果有动态创建并销毁子类场景时必须写虚析构函数，此时可以用=default，那么其他成员可以自动使用合成的，如果是自定义实现，那么必须同时实现3/5成员
- 不使用合成成员需要自己实现以明确拒绝
- 合成析构函数是非虚的，除非有基类
- 拷贝应当复制对象内所有成员和基类成员
- 如果增加了数据成员，那么拷贝成员也应当更新
- 不需要拷贝时使用=delete阻止，不要使用老版本的private或者什么boost::noncopyable接口了
- 拷贝赋值运算符应当具有异常安全性并处理自赋值情况（常用copy and swap技巧）

``` cpp
#include <cstring>
#include <algorithm>

class Buffer
{
private:
    char* buf;
    size_t size;
public:
    Buffer(size_t n = 0)
        : size(n), buf(new char[n]{})
    {}

    // 拷贝构造
    Buffer(const Buffer& other)
        : size(other.size), buf(new char[other.size])
    {
        std::memcpy(buf, other.buf, size);
    }

    // 交换函数
    void swap(Buffer& other) noexcept
    {
        std::swap(buf, other.buf);
        std::swap(size, other.size);
    }

    // 拷贝赋值（拷贝交换版）
    Buffer& operator=(Buffer other) // 值传递，自动调用拷贝构造
    {
        swap(other);
        return *this;
    }

    ~Buffer()
    {
        delete[] buf;
    }
};
```

### ==移动成员==

- 为了提高效率可以增加移动成员，包括移动构造、移动赋值运算符、移动成员函数
- 不管是啥移动函数记得后缀noexcept和源对象置空这一步
- 增加了移动构造、移动赋值运算符和移动成员函数的类如下

``` cpp
#include <cstring>
#include <algorithm>
#include <utility>

class Buffer
{
private:
    char* buf;
    size_t size;
public:
    // 普通构造
    Buffer(size_t n = 0)
        : size(n), buf(new char[n]{})
    {}

    // 拷贝构造
    Buffer(const Buffer& other)
        : size(other.size), buf(new char[other.size])
    {
        std::memcpy(buf, other.buf, size);
    }

    // 移动构造：复用swap实现
    Buffer(Buffer&& other) noexcept
        : buf(nullptr), size(0) // 初始化为空对象
    {
        swap(other); // 和传入右值交换，直接夺取资源
    }

    // 成员swap，noexcept
    void swap(Buffer& other) noexcept
    {
        std::swap(buf, other.buf);
        std::swap(size, other.size);
    }

    // copy-and-swap 拷贝赋值
    Buffer& operator=(Buffer other)
    {
        swap(other);
        return *this;
    }

    // 移动赋值：同样复用swap简化代码
    Buffer& operator=(Buffer&& other) noexcept
    {
        Buffer temp(std::move(other)); // 调用移动构造生成空临时
        swap(temp);
        return *this;
    }

    ~Buffer()
    {
        delete[] buf;
    }
};
```

### 成员函数

- 应当事先用UML规划清晰的接口设计，接口名称应当容易理解，容易使用
- 尽量让自己设计的接口行为与内置类型一致
- 接口的公共常用部分抽离出来作为工具函数放在private中封装降低耦合
- [swap成员函数](2.1%20C++编程技巧和常用代码%20*.md#%5Eswap)是一类重要的特殊的成员函数，推荐同空间再写一个配套的inline非成员函数，不推荐特例化std::swap嗷，应当设置为noexcept
  \### 高内聚低耦合要求
- 概念：单个类内部功能高度统一只管一件事，模块间依赖少关联弱
- 设计模式：单一职责原则、最小暴露原则\[\[../4.设计模式/设计模式\]\]
- 具体要求：头文件尽量使用前向声明少用include、工具辅助类放在同模块命名空间、高层代码只依赖抽象接口、多用组合少用继承
- 合格：修改某个功能只用改某个类和模块，修改底层不会影响上层
  \### 提高类的封装性
- 成员变量都是private，不准用户直接访问，实在不行只能用get/set方法访问
- PIMPL模式提高封装性
- 禁止返回内部裸指针、左值引用、迭代器
- 只读接口全部const
- 构造函数来接收外部抽象接口，不要直接使用外部变量

``` cpp
// 低封装，强耦合
class Upload {
private:
    FileLog log; // 写死日志实现，无法替换
};
// 高封装，依赖抽象
class Upload {
public:
    explicit Upload(ILog* logger) : m_log(logger) {}
private:
    ILog* m_log;
};
```

- 能不写成员函数就不写（用非成员替代）
- 友元越少越好
  \## 面向对象设计
- 世界上没有完美的设计，只有适合当前的设计
- 派生类与基类特性不同：重新设计继承关系

``` cpp
//企鹅是不会飞的鸟
class Bird{};
class FlyingBird: public Bird{
    public:
        virtual void fly() { std::cout << "I can fly!" << std::endl; }
};
class Penguin: public Bird{
    public:
        void fly() { std::cout << "I can't fly!" << std::endl; }
};
```

- 派生类同名覆盖问题：使用using声明，不需要的版本使用delete删除
- 空白基类最优化：空白基类在继承时不占用空间，但是单独声明对象有空间（1字节）
- 纯虚函数相当于接口：基类中的函数声明后加 = 0，纯虚函数理论不用定义，但实际上也可以定义作为默认行为且在工程中常用，派生类如果没有重写抽象基类全部的虚函数，那么也认为是个抽象基类，不能实例化，参见[抽象基类](C++primer笔记.md#抽象基类)
- 虚函数会增加类体积（有虚表和虚指针），虚函数的默认实参是静态绑定的
  \### 类间关系
- 参见[访问限定符](C++primer笔记.md#访问限定符)
- is a：public继承，用的最多
- is implement of：private继承，不提供对外接口，不允许外部向上转型因此阻止了多态
- has a：复合也就是拥有成员，尽可能使用复合不用private继承

``` cpp
class Widget{
    private:
        class WidgetTimer : public Timer{
            public:
                virtual void onTick() const;
        };
        WidgetTimer timer;
};
```

- [PIMPL技巧](2.1%20C++编程技巧和常用代码%20*.md#PIMPL技巧)完成接口与实现分离
  \### 虚函数的使用
- 设计模式-模板方法模式（NVI手法）
  对外接口都是非虚函数并提供基础框架调用虚函数，虚函数才是真正具体实现，所有虚函数都是private供派生类重写

``` cpp
#include <iostream>
#include <mutex>

class Task
{
public:
    // 对外唯一公共接口：非虚，固定执行流程
    void run()
    {
        // 统一前置逻辑（所有子类必走，不可跳过）
        std::lock_guard<std::mutex> lock(m_mtx);
        beforeExecute();

        // 子类自定义核心逻辑（虚函数）
        doExecute();

        // 统一后置收尾
        afterExecute();
    }

    virtual ~Task() = default;

private:
    // 留给子类重写的纯虚函数，private，外部不能调用
    virtual void doExecute() = 0;

    // 统一固定步骤，所有任务共用
    void beforeExecute()
    {
        std::cout << "前置：初始化资源、记录日志\n";
    }
    void afterExecute()
    {
        std::cout << "后置：释放临时资源\n";
    }

    std::mutex m_mtx;
};

// 子类仅重写私有虚函数 doExecute
class DownloadTask : public Task
{
private:
    void doExecute() override
    {
        std::cout << "子类自定义：执行文件下载\n";
    }
};

int main()
{
    Task* t = new DownloadTask();
    t->run(); // 只调用公共非虚接口
    delete t;
    return 0;
}
```

- 设计模式-Strategy模式
  抽象业务持有private的抽象策略类，抽象策略类中虚函数都是public的支持派生，可以用具体策略方便替换抽象策略

``` cpp
#include <iostream>
#include <memory>

// 1. 抽象策略：支付策略接口，虚函数统一规范
class PaymentStrategy
{
public:
    virtual ~PaymentStrategy() = default;
    // 纯虚函数：所有支付方式必须实现
    virtual void pay(double money) const = 0;
};

// 2. 具体策略1：微信支付
class WechatPay : public PaymentStrategy
{
public:
    void pay(double money) const override
    {
        std::cout << "微信支付：" << money << " 元\n";
    }
};

// 具体策略2：支付宝
class Alipay : public PaymentStrategy
{
public:
    void pay(double money) const override
    {
        std::cout << "支付宝支付：" << money << " 元\n";
    }
};

// 具体策略3：银行卡
class BankCardPay : public PaymentStrategy
{
public:
    void pay(double money) const override
    {
        std::cout << "银行卡支付：" << money << " 元\n";
    }
};

// 3. Context 上下文主体（业务主体，依赖抽象策略）
class Order
{
private:
    // 持有抽象策略，不绑定具体实现，低耦合
    std::unique_ptr<PaymentStrategy> m_strategy;
public:
    // 注入策略
    explicit Order(std::unique_ptr<PaymentStrategy> s)
        : m_strategy(std::move(s)) {}

    // 运行时动态更换策略
    void setStrategy(std::unique_ptr<PaymentStrategy> s)
    {
        m_strategy = std::move(s);
    }

    // 对外统一接口，内部调用虚函数触发多态
    void checkout(double total) const
    {
        m_strategy->pay(total);
    }
};

int main()
{
    // 下单默认微信支付
    Order order(std::make_unique<WechatPay>());
    order.checkout(99.9);

    // 运行时切换策略：支付宝
    order.setStrategy(std::make_unique<Alipay>());
    order.checkout(199.9);

    // 切换银行卡
    order.setStrategy(std::make_unique<BankCardPay>());
    order.checkout(299.9);
    return 0;
}
```

### 虚析构函数

- 存在对基类指针和派生类指针的new和delete时必须写虚析构函数
- 析构函数无论是不是虚析构都禁止抛出异常：[2.30 析构函数为什么禁止抛出异常？如果有异常咋办？](../面经.md#2.30%20析构函数为什么禁止抛出异常？如果有异常咋办？)
- [2.31 为什么禁止在构造和析构函数中调用虚函数？](../面经.md#2.31%20为什么禁止在构造和析构函数中调用虚函数？)
- 非虚函数继承接口固定实现，禁止子类重新定义
  \### ==多重继承与虚继承==
- ==尽量使用单一继承==，多重继承实际使用中public继承接口，private继承实现
- ==尽量使用非虚继承==，如果必须使用虚继承避免在虚基类放置过多数据
- 多重继承使用中如果多个基类有同名函数则调用会发生歧义，需要指明基类作用域
- [2.32 虚继承是什么？解决什么问题？](../面经.md#2.32%20虚继承是什么？解决什么问题？)
  \# template C++
- 面向对象是显式接口和运行期多态，模板编程是隐式接口和编译期多态，表达式约束类的使用方法，编译期确定不同的类型
  \## 成员函数模板
- 成员函数模板不阻止合成函数
- 实参推导过程只有部分简单转换，大部分隐式类型转换不生效，[模板支持的实参隐式转换](C++primer笔记.md#模板支持的实参隐式转换)
  \## 模板继承
- 类模板实例间不存在继承关系是两个无关类型，根源上不存在继承关系
- 派生类对象转基类对象（智能指针底层实现）

``` cpp
template <typename T>
class SmartPtr
{
public:
    template <typename U>
    SmartPtr(const SmartPtr<U> &other) 
        : heldPtr(other.get())
    {
        // 注释：只有 U* 能够隐式/合法转换为 T*，代码才能编译通过
        // 以此实现「模板继承」效果
    }

    T *get() const { return heldPtr; }

private:
    T *heldPtr;
};

class Base{};
class Derived : public Base{};

SmartPtr<Derived> dp(new Derived);
SmartPtr<Base> bp = dp; // ✅ 可以成功构造
```

- 运行时多态（Concept-Model）

``` cpp
#include <memory>
#include <iostream>

// 1. 无模板抽象基类：所有模板实例的统一父类（真正继承关系）
struct Concept {
    virtual void show() const = 0;
    virtual ~Concept() = default;
};

// 2. 模板包装子类，继承统一Concept，持有任意T
template<typename T>
struct Model : Concept {
    T data;
    Model(T t) : data(std::move(t)) {}
    void show() const override {
        data.print();
    }
};

// 对外封装
class AnyWrapper {
    std::unique_ptr<Concept> ptr;
public:
    template<typename T>
    AnyWrapper(T t) : ptr(std::make_unique<Model<T>>(std::move(t))) {}
    void show() const { ptr->show(); }
};

// 业务继承类
struct Base { void print() const { std::cout << "Base\n"; } };
struct Derived : Base { void print() const { std::cout << "Derived\n"; } };

int main()
{
    AnyWrapper w1(Base{});
    AnyWrapper w2(Derived{});
    w1.show();
    w2.show();

    // 可存入容器，异构类型统一管理（动态多态）
    // std::vector<AnyWrapper> arr{Base{}, Derived{}};
    return 0;
}
```

### c++拒绝在模板化基类内寻找成员函数

- 因为模板化基类没实例化之前默认不知道里面有啥（模板基类可以特例化成员完全不一样）

``` cpp
template<typename T>
struct Base {
    void func() {} // func 和 T 无关：非依赖名
};

template<typename U>
struct Derived : Base<U> {
    void test() {
        func(); // 报错！找不到func
    }
};
```

- 解决方法：

1.  this调用（推荐）

``` cpp
template<typename U>
struct Derived : Base<U> {
    void test() {
        this->func(); // 正常编译
    }
};
```

2.  基类作用域运算符

``` cpp
void test() {
    Base<U>::func();
}
```

2.  using声明

``` cpp
template<typename U>
struct Derived : Base<U> {
    using Base<U>::func; // 将基类func引入当前类域
    void test() {
        func(); // 直接调用无报错
    }
};
```

## 阻止模板代码膨胀

### 非类型模板参数

- 用成员变量代替非类型模板参数
  \### 类型模板参数
- 指针类型都有相同的大小，可以使用`void*`代替
  \## ==traits技巧==
- 类型萃取：用模板封装一套类型定制标签和行为，编译期就能获得类型属性和相关参数
- 标准库：is_pointer、is_base_of、is_array
- std::true_type是一个编译期布尔常量，作为traits技巧承载判断结果的载体
- std::void_t无论输入统一输出void类型

``` cpp
#include <type_traits>

// 1. 基础主模板
template<typename T, typename = void>
struct MyTrait : std::false_type {};

// 2. 条件偏特化版本
template<typename T>
struct MyTrait<T, std::void_t<decltype( std::declval<T>().func() )>>
    : std::true_type {};

// 3. 快捷变量模板
template<typename T>
constexpr bool MyTrait_v = MyTrait<T>::value;

struct A {
    void func() {}
};

// 匹配偏特化，MyTrait<A, void> → true
static_assert(MyTrait_v<A> == true);
```

## 奇异递归模板（CRTP）

- 概念：派生类自身作为模板实参传递给基类模板，实现编译期静态多态

``` cpp
template <typename Derived>
struct Base {
    void work() {
        // 调用派生类实现
        derived().impl_work();
    }
private:
    Derived& derived() { return static_cast<Derived&>(*this); }
};

struct A : Base<A> {
    void impl_work() {
        puts("A 执行逻辑");
    }
};

struct B : Base<B> {
    void impl_work() {
        puts("B 执行逻辑");
    }
};

int main() {
    A{}.work();
    B{}.work();
    return 0;
}
```

## 模板元编程

- 概念：在编译期执行计算、类型变换，把开销前置到编译阶段，编译时间长但运行期高效。
- 应用：确保量度单位正确（早期错误侦测）、优化矩阵运算、生成定制类的设计模式
- 数据：类型、编译期常量（`constexpr`）；
- 变量：模板参数；
- 分支：模板偏特化、`if constexpr`；
- 循环：模板递归实例化；
- 返回值：产出新类型 / 编译期常量数值。

``` cpp
// 递归版本
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};
// 边界特化：递归终止条件
template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

// 编译期直接算出结果，运行时只是读取常量
constexpr int res = Factorial<5>::value; // 120
```

## 常用模板

1.  返回序列元素的引用

``` cpp
template <typename It>
auto fcn(It beg, It end) -> decltype(*beg)
{
    return *beg;
}
```

2.  返回序列元素拷贝

``` cpp
template <typename It>
auto fcn2(It beg, It end) -> typename remove_reference<decltype(*beg)>::type
{
    return *beg;
}
```

3.  实参转发

``` cpp
template <typename F, typename... Args>
void flip1(F f, Args &&...args)
{
    f(std::forward<Args>(args)...);
}
```

# STL

## 迭代器分类

1.  [按功能分类](C++primer笔记.md#类型：)
2.  [按读写顺序分类](C++primer笔记.md#迭代器类别)

- 输入（前向、只读、单扫）：istream_iterator
- 输出（前向、只写、单扫）：ostream_iterator
- 前向（前向、读写、多扫）：forward_list、unordered_map/unordered_set的迭代器（桶内链表是单向指针）
- 双向（双向、读写、多扫）：list、set、map、multiset、multimap的迭代器
- 随机访问（任意、读写、多扫）：vector、string、array、deque、裸指针
  \## 常用IO代码

1.  cin和cout解锁

``` cpp
ios::sync_with_stdio(false);//关闭与printf/scanf的同步
cin.tie(nullptr);//关闭cin与cout的绑定，读取cin时不刷新cout缓冲区
cout.tie(nullptr);
```

2.  读取int

``` cpp
istream_iterator<int> in_iter(std::cin), eof_iter;
vector<int> vec(in_iter, eof_iter);
```

3.  读取string

``` cpp
struct PersonInfo
{
    string name;
    vector<string> phones;
};
string line, word;
vector<PersonInfo> people;
while (getline(cin, line))
{
    PersonInfo info;
    istringstream record(line);
    record >> info.name;
    while (record >> word)
        info.phones.push_back(word);
    people.push_back(info);
}
```

4.  读取二进制

``` cpp
bitset<16> bits;
cin >> bits;
```

5.  非格式化io读取

``` cpp
int ch;
while((ch = getchar()) != EOF)
{
    cout.putchar(ch);
}
```

6.  非格式化io标记流

``` cpp
ostringstream os;
ostringstream::pos_type pos = os.tellp();
os.seekg(0, ostringstream::end);
os.seekp(pos);
```

7.  输出序列

``` cpp
ostream_iterator<int> out_iter(std::cout, " ");
copy(vec.begin(), vec.end(), out_iter);
cout << endl;
```

## 常用字符串代码

1.  字符串排序

``` cpp
sort(sv.begin(), sv.end(), greater<string>());
```

2.  大小写转换

``` cpp
string s;
transform(s.begin(), s.end(), s.begin(), ::toupper);
transform(s.begin(), s.end(), s.begin(), ::tolower);
```

3.  字符串大量拼接

``` cpp
for (const auto &entry : people)
{
    ostringstream formatted, badNums;
    for (const auto &phone : entry.phones)
    {
        if (!valid(phone))
        {
            badNums << " " << phone;
        }
        else
        {
            formatted << " " << format(phone);
        }
    }
}
```

4.  循环搜索子字符串位置

``` cpp
string::size_type pos = 0;
while ((pos = name.find_first_of("aeiou", pos)) != string::npos)
{
    cout << pos << endl;
    ++pos;
}
```

## 常用迭代器代码

1.  循环重新定位迭代器

``` cpp
vector<int> vi = {0,1,2,3,4,5};
auto iter = vi.begin();
while (iter != vi.end()){
    cout << *iter++ << " ";
}
```

## 常用关联容器代码

1.  查找子集

``` cpp
string search_item("Alex");
multimap<string> authors;
for (auto pos = authors.equal_range(search_item); pos.first != pos.second; ++pos.first)
{
    cout << pos.first->second << endl;
}
```

# 异常安全（少用）

- 异常安全性：不泄漏资源，不允许不一致性，整体的异常安全取决于局部的最弱异常安全
  \## 异常安全函数设计
- 能有基本型就不错了
- 基本型：函数内部不会有不一致性，但是现实状态无法预料
- 强烈型：函数执行失败会恢复到调用前的状态，将原资源拷贝一份后修改，等所有改变都成功才进行交换（[copy and swap技巧](2.1%20C++编程技巧和常用代码%20*.md#%5Ecopy-and-swap)）
- 不抛出异常保证：承诺绝对不抛出异常
  \# 动态内存管理
- RAII对象：资源在构造时获得析构时释放，被称为资源类，复制RAII对象也需要复制资源
- 推荐使用智能指针进行动态内存管理
  \## 智能指针
- 智能指针默认删除操作是delete而不是delete\[\]
- 推荐unique_ptr代替裸指针管理堆
  \### shared_ptr
- 通常用于共享资源管理、异步回调、多线程
- 存在循环引用缺陷：两个对象互相用shared_ptr指向对方引用计数无法释放，内存永久泄漏
- 解决new dll delete问题：每个dll有各自的堆管理器，如果a的裸指针在b中delete可能会删除错堆导致崩溃

``` cpp
// dll.h
struct Foo;
__declspec(dllexport) std::shared_ptr<Foo> CreateFoo();
void DestroyFoo(Foo* p); // 定义在DLL内部，使用DLL自身的delete

// dll.cpp
#include <memory>
struct Foo {};

void DestroyFoo(Foo* p)
{
    delete p; // 此处是DLL模块的operator delete，匹配本模块new
}

std::shared_ptr<Foo> CreateFoo()
{
    Foo* raw = new Foo;
    // 绑定DLL内部的释放函数作为删除器
    return std::shared_ptr<Foo>(raw, DestroyFoo);
}

#include "dll.h"
int main()
{
    auto sp = CreateFoo();
    // sp 析构时，自动调用 DLL 里的 DestroyFoo
    // 分配、释放全程都在同一个DLL堆，安全无崩溃
    return 0;
}
```

- 解互斥锁：

``` cpp
#include <mutex>
#include <memory>
#include <iostream>

std::mutex mtx;

int main()
{
    mtx.lock();

    // 传入空指针，删除器绑定解锁逻辑
    std::shared_ptr<std::mutex> lockGuard(nullptr, [&](std::mutex*) {
        mtx.unlock();
    });

    // 临界区代码
    std::cout << "临界区运行中\n";

    // 离开作用域 lockGuard 析构 → 自动调用lambda，执行mtx.unlock()
    return 0;
}
```

- 独占空间代码：

``` cpp
auto p = make_shared<string>("1234");
if (!p.unique())
{
    p.reset(new string(*p));
}
```

### unique_ptr

- 独占型指针，性能比shared_ptr更高，通常用于代替裸指针，常与容器联合使用
- 常用代码：

``` cpp
unique_ptr<int> q(new int(42));
auto res = q.release();
delete res; //release() returns the pointer and releases ownership, so we need to delete it manually to avoid memory leak.

unique_ptr<int, decltype(end_connection) *> p2(new int(42), end_connection);

unique_ptr<int[]> up(new int[10]); // unique_ptr for array of ints
for(size_t i = 0; i != 10; ++i) {
    up[i] = i; // Assign values to the array
}
```

### weak_ptr

- 用于检查shared_ptr计数

``` cpp
weak_ptr<string> w(p);
auto p1 = w.lock();
if(!p1) {
    cout << "The object has been deleted." << endl;
    throw runtime_error("The object has been deleted.");
} else {
    cout << "The object is still alive: " << *p1 << endl;
}
```

- 常用于解决shared_ptr的循环引用缺陷

``` cpp
struct Node {
    int val;
    std::weak_ptr<Node> next; // 换成弱指针，不计引用计数
    Node(int v) : val(v) {}
    ~Node() {
        std::cout << "Node 析构释放\n";
    }
};

int main() {
    auto A = std::make_shared<Node>(1);
    auto B = std::make_shared<Node>(2);

    A->next = B;
    B->next = A;

    return 0;
    // 出作用域，A、B计数依次归零，正常调用析构，无内存泄漏
}
```

## 资源管理类

- 常用方法：阻止复制、引用计数、深度拷贝、移动操作
- 资源管理类应当提供对原始资源的访问接口，且支持转换，为了安全必须是显式转换

``` cpp
class Font
{
private:
    FontHandle handle;

public:
    FontHandle getHandle() const { return handle; }
};
```

## 手工管理（少用）

- 容器分配的内存是allocator管理的不是手工管理的
- new与delete必须成对使用尤其是\[\]
- ==与智能指针联合使用时必须独立出来防止初始化次序不确定==

``` cpp
auto p = new Widget;
std::shared_ptr<Widget> p1(p);
process(p1, priority());
```

- nothrow new只能保证new不抛出异常，不保证建立的对象不抛出异常

``` cpp
Widget *pw2 = new (std::nothrow) Widget;
```

### new-handler

- 当new抛出异常时（比如无法满足内存申请）会自动调用全局注册的new-handler
- 默认new-handler：抛出std::bad_alloc异常
- 获取当前new-handler：get_new_handler()
- 设置当前new-handler：set_new_handler()，会返回旧的handler
- 全局new-handler：

``` cpp
void outofMemory()
{
    std::cerr << "out of memory" << std::endl;
    std::abort();
}
auto old_pf = std::set_new_handler(outofMemory);
```

- 类new-handler：

``` cpp
#include <new>
#include <iostream>

class Foo
{
public:
    static void foo_new_handler()
    {
        std::cerr << "Foo类专属内存分配失败\n";
        throw std::bad_alloc();
    }

    // 重载类专属operator new
    static void* operator new(std::size_t sz)
    {
        // 保存当前全局handler
        new_handler old = std::set_new_handler(foo_new_handler);

        void* ptr = ::operator new(sz);

        // 分配完毕，恢复原先全局handler
        std::set_new_handler(old);
        return ptr;
    }
};

int main()
{
    // 只有 new Foo 失败才会触发 foo_new_handler
    new Foo;
    return 0;
}
```

### 自定义new/delete操作符（少用）

- 写 `new A` 看不到 `size_t`，**这个尺寸由编译器在编译期自动计算、自动填进 `operator new` 的第一个形参**，不需要手动传递
- 自定义new/delete功能应当与内置类型一致，自定义new\[\]需要分配一块较大的未加工内存
- 常用于：检测内存上的错误（内存泄漏、缓冲区溢出）、提高分配和回收速度、收集使用上的统计数据、降低空间开销、解决字节对齐问题、减少换页损失、获得非传统行为
- C++语言new返回的指针都会字节对齐，而C语言malloc也是返回的指针会字节对齐，底层常用malloc实现
- 大部分时候不建议自定义new/delete，可以使用商业或开源的内存管理器（比如boost::pool），只不过多一个relink的步骤
- 自定义new/delete运算符

``` cpp
#include <iostream>
#include <new>
#include <cstdlib>

struct MyObj
{
    int a;

    // 类专属 new
    static void* operator new(std::size_t sz)
    {
        std::cout << "MyObj::new 分配，字节数：" << sz << "\n";
        void* p = std::malloc(sz);
        if (!p)
            throw std::bad_alloc{};
        return p;
    }

    // 类专属 delete
    static void operator delete(void* ptr) noexcept
    {
        if (!ptr)
            return;
        std::cout << "MyObj::delete 释放\n";
        std::free(ptr);
    }

    // 数组 new[]
    static void* operator new[](std::size_t sz)
    {
        std::cout << "MyObj::new[] 分配：" << sz << "\n";
        return ::operator new(sz);
    }

    // 数组 delete[]
    static void operator delete[](void* ptr) noexcept
    {
        std::cout << "MyObj::delete[] 释放\n";
        ::operator delete(ptr);
    }
};

int main()
{
    MyObj* p1 = new MyObj;
    delete p1;

    MyObj* arr = new MyObj[5];
    delete[] arr;

    // 普通 int 不会走上面的重载
    int* pi = new int;
    delete pi;
    return 0;
}
```

### placement new/delete

- 广义：形参中有除了默认参数以外参数的new/delete运算符
- 狭义：`new(std::size_t, void *pMemory)`
- 必须成对定义，系统自动寻找参数个数类型都匹配的placement new和delete，注意直接使用delete会调用正常版本，只有在placement new出现异常时才调用placement delete
- 类中定义placement new会覆盖global new
- 默认行为：不分配堆内存，仅在一块已经提前准备好的内存地址上调用构造函数初始化对象
- 自定义placement new/delete

``` cpp
#include <new>
#include <iostream>
#include <cstdlib>

struct Demo
{
    int data;
    Demo() : data(666)
    {
        // 测试构造抛异常触发placement delete
        // throw std::runtime_error("construct error");
    }

    // 仅自定义 placement new，不实现普通 operator new(size_t)
    static void* operator new(std::size_t sz, void* addr) noexcept
    {
        std::cout << "类内自定义 placement new, size = " << sz << "\n";
        return addr;
    }

    // 配套同签名 placement delete
    // 仅在 placement new 分配成功、构造函数抛出异常时编译器自动调用
    static void operator delete(void* ptr, void* addr) noexcept
    {
        std::cout << "类内 placement delete 触发，构造异常回滚清理\n";
        // 此处一般无需释放堆内存，placement本身没有申请堆内存
    }

    // 数组版本 placement new / delete（按需）
    static void* operator new[](std::size_t sz, void* addr) noexcept
    {
        std::cout << "数组 placement new\n";
        return addr;
    }

    static void operator delete[](void* ptr, void* addr) noexcept
    {
        std::cout << "数组 placement delete\n";
    }
};

int main()
{
    // 开辟一块备用内存
    alignas(Demo) char buffer[sizeof(Demo)]{};

    // 触发当前类的 placement new
    Demo* obj = new(buffer) Demo;

    // 必须手动调用析构，禁止直接 delete
    obj->~Demo();

    // 测试数组placement
    alignas(Demo) char arr_buf[sizeof(Demo) * 3]{};
    Demo* arr = new(arr_buf) Demo[3];
    for (int i = 0; i < 3; ++i)
        arr[i].~Demo();

    // 普通 new Demo 不会进入上面任何重载，使用全局默认operator new
    Demo* heap_obj = new Demo;
    delete heap_obj;

    return 0;
}
```

### nothrow new/delete

- 与普通new/delete的区别在：分配失败后不抛出异常和触发new-handler，直接返回nullptr
- 注意直接使用delete会调用正常版本，只有在拿到内存后构造对象出现异常才调用nothrow delete
- 自定义nothrow new/delete

``` cpp
#include <new>
#include <cstdlib>
#include <iostream>

class TestObj
{
public:
    int val;
    TestObj() : val(100) {}

    // 只定义 nothrow 版本 operator new，不写普通operator new
    static void* operator new(std::size_t size, const std::nothrow_t&) noexcept
    {
        std::cout << "调用类内 nothrow new，申请字节数：" << size << '\n';
        return std::malloc(size);
    }

    // 配套 nothrow 形式的 operator delete
    // 仅在：nothrow new分配成功，但构造函数抛出异常时由编译器自动调用
    static void operator delete(void* ptr, const std::nothrow_t&) noexcept
    {
        std::cout << "调用类内 nothrow delete，回收内存\n";
        if (ptr != nullptr)
        {
            std::free(ptr);
        }
    }

    // 数组版本 nothrow new / delete（按需选用）
    static void* operator new[](std::size_t size, const std::nothrow_t&) noexcept
    {
        std::cout << "调用类内数组 nothrow new\n";
        return std::malloc(size);
    }

    static void operator delete[](void* ptr, const std::nothrow_t&) noexcept
    {
        std::cout << "调用类内数组 nothrow delete\n";
        if (ptr != nullptr)
        {
            std::free(ptr);
        }
    }
};

int main()
{
    // 触发类内 nothrow new
    TestObj* p = new(std::nothrow) TestObj;
    if (p != nullptr)
    {
        std::cout << p->val << '\n';
        delete p; // 走全局默认operator delete，不会进上面的nothrow delete
    }

    // 数组形式
    TestObj* arr = new(std::nothrow) TestObj[5];
    if (arr != nullptr)
    {
        delete[] arr;
    }

    return 0;
}
```

# 常用代码

1.  流动态构造vector

``` cpp
#include <iostream>
#include <iterator>
#include <vector>

std::istream_iterator<int> in_iter(std::cin),eof_iter;
std::vector<int> vec(in_iter,eof_iter);
for(const auto &i : vec)
{
    std::cout << i << " ";
}
```

2.  流动态输出vector

``` cpp
std::ostream_iterator<int> out_iter(std::cout," ");
copy(vec.begin(),vec.end(),out_iter);
```

3.  遍历多维数组
    内置数组：

``` cpp
int arr[3][4] = {
    {1, 2, 3, 4},
    {5, 6, 7, 8},
    {9, 10, 11, 12}
};
for(auto& row : arr)
{
    for(auto& col : row)
    {
        std::cout << col << " ";
    }
    std::cout << std::endl;
}
```

vector：

``` cpp
vector<vector<int>> vec = { {10, 20}, {30, 40}, {50, 60} };
for (auto it_row = vec.begin(); it_row != vec.end(); ++it_row) {
    // it_row 推导为 vector<vector<int>>::iterator 
    for (auto it_num = it_row->begin(); it_num != it_row->end(); ++it_num){
        *it_num += 1; 
        cout << *it_num << " "; 
    } 
    cout << endl; 
}
```

4.  循环读入数据

``` cpp
int value = 0;
while (std::cin>>value)
{
    /* code */
}
```

5.  const_cast重载函数（不推荐） \^const-cast

``` cpp
const string &shorterString (const string &s1, const string &s2) {
    return s1.size() <= s2.size ()?s1:s2;
}

string &shorterString(std::string &s1, std::string &s2){
    auto &r = shorterString(const_cast<const string&>(s1),const_cast<const string&>(s2));
    return const_cast<string&>(r);
}
```

6.  统一const和非const成员函数的功能（推荐） \^const-nonconst

``` cpp
class Screen
{
public:
    // 根据对象是否是 const 重载了 display 函数
    Screen &display(std::ostream &os)
    {
        do_display(os);
        return *this;
    }
    const Screen &display(std::ostream &os) const
    {
        do_display(os);
        return *this;
    }

private:
    // 该函数负责显示 Screen 的内容
    void do_display(std::ostream &os) const { os << "contents"; }
    // 其他成员与之前的版本一致
};
```

7.  拷贝赋值运算符的合理写法 \^operator-copy

``` cpp
HasPtr &HasPtr::operator=(const HasPtr &rhs)｛ 
    auto newp = new string(*rhs.ps); // 拷贝底层 string
    delete ps; // 释放旧内存
    ps = newp; // 从右侧运算对象拷贝数据到本对象
    i = rhs.i;
    return *this; // 返回本对象
}
```

8.  copy and swap实现赋值运算符 \^copy-and-swap

``` cpp
T& operator=(T s){
    swap(*this,s);
    return *this;
}
```

9.  swap实现 \^swap

``` cpp
#include <iostream>
#include <utility>

namespace Acls
{
    class A;
    //有这个就好使了
    inline void swap(A &lhs, A &rhs) noexcept
    {
        lhs.swap(rhs);
    };
    class A
    {
    private:
        char *data;
        std::size_t len;

    public:
        void swap(A &other) noexcept{
            std::swap(data, other.data);
            std::swap(len, other.len);
        };
    };
    
}

// std通用swap特化，但是包大人说了不推荐存在兼容性隐患
namespace std
{
    template <>
    void swap(Acls::A &lhs, Acls::A &rhs) noexcept
    {
        Acls::swap(lhs,rhs);
    }
}
```

10. 容器使用函数对象完成排序 \^funcsort

``` cpp
vector<string> vec1;
sort(vec1.begin(),vec1.end(),greater<string>());
```

11. 派生类的带参构造函数 \^derive-construct

``` cpp
class Base {
public:
    Base(int a) { cout << "基类带参构造 a=" << a << endl; }
};

class Derive : public Base {
public:
    Derive(int x) : Base(x) {
        cout << "派生类构造 x=" << x << endl;
    }
};
```

12. std::move的实现 \^stdmove

``` cpp
template<typename T>
typename std::remove_reference<T>::type&& move(T&& t) noexcept
{
    return static_cast<typename std::remove_reference<T>::type&&>(t);
}
```

13. 递归可变参数模板

``` cpp
template<typename T> 
ostream& print(ostream &os, const T&t){
    return os<<t;
}
template<typename T, typename... Args> 
ostream& print(ostream &os, const T&t, const Args&... rest){
    os<<t<<",";
    return print(os,rest...);
}
```
