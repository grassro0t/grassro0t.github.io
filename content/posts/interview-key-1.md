---
title: "后端面试高频重点（一）"
slug: "interview-key-1"
date: 2026-09-27T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["设计模式", "数据结构", "算法", "C++"]
categories: ["面试八股"]
summary: "本系列内容以面试和工作高频知识点为引扩展全貌，涵盖后端几乎所有常用知识点。"
toc: true
comments: true
description: "后端面试高频重点（一）"
---

>本部分内容为个人整理，以面试和工作高频知识点为引扩展全貌，涵盖计算机软件工程方向几乎所有常用知识点，只要是在其中的都是重点内容，只是有的可能对面试意义不大，将根据面试进度推进不断补充或修改
- ⭐⭐⭐⭐⭐：极高频考点，必须掌握
- ⭐⭐⭐⭐：偶尔考到，尽量掌握
- ⭐⭐⭐：很少考到，看情况掌握
# 设计模式
- 介绍单例模式⭐⭐⭐⭐⭐
	- 私有构造函数+静态实例，全局单个实例
	- 饿汉式：建立时就创建
	- 懒汉式：用到才创建
``` cpp
    // 懒汉式单例模式
    // 线程安全
    class SingletonLazyFinal
    {
    public:
        SingletonLazyFinal(const SingletonLazyFinal &) = delete;
        SingletonLazyFinal &operator=(const SingletonLazyFinal &) = delete;

        static SingletonLazyFinal &getInstance()
        {
            // 首次调用才构造，C++11 自动线程安全
            static SingletonLazyFinal inst;
            return inst;
        }

    private:
        SingletonLazyFinal() = default;
    };
```
- 介绍工厂模式⭐⭐⭐⭐⭐
	- 单工厂单产品，抽象工厂返回抽象产品，一个具体工厂要实现一个具体产品
``` cpp
    // 抽象产品
    class Product
    {
    public:
        virtual ~Product() = default;
        virtual void show() = 0;
    };

    class ProductA : public Product
    {
    public:
        void show() override { std::cout << "Product A\n"; }
    };
    class ProductB : public Product
    {
    public:
        void show() override { std::cout << "Product B\n"; }
    };

    // 抽象工厂
    class Factory
    {
    public:
        virtual ~Factory() = default;
        virtual std::unique_ptr<Product> create() = 0;
    };

    class FactoryA : public Factory
    {
    public:
        std::unique_ptr<Product> create() override
        {
            return std::make_unique<ProductA>();
        }
    };

    class FactoryB : public Factory
    {
    public:
        std::unique_ptr<Product> create() override
        {
            return std::make_unique<ProductB>();
        }
    };

```
- 介绍抽象工厂模式
	- 单工厂多产品，只是有具体产品族继承多个抽象产品，具体工厂是实现具体产品族的
``` cpp
    // 产品1：按钮
    class Button
    {
    public:
        virtual ~Button() = default;
        virtual void render() = 0;
    };
    // 产品2：文本框
    class TextBox
    {
    public:
        virtual ~TextBox() = default;
        virtual void render() = 0;
    };

    // Windows族
    class WinButton : public Button
    {
    public:
        void render() override { std::cout << "Windows Button\n"; }
    };
    class WinTextBox : public TextBox
    {
    public:
        void render() override { std::cout << "Windows TextBox\n"; }
    };

    // Mac族
    class MacButton : public Button
    {
    public:
        void render() override { std::cout << "Mac Button\n"; }
    };
    class MacTextBox : public TextBox
    {
    public:
        void render() override { std::cout << "Mac TextBox\n"; }
    };

    // 抽象工厂
    class GUIFactory
    {
    public:
        virtual ~GUIFactory() = default;
        virtual std::unique_ptr<Button> createButton() = 0;
        virtual std::unique_ptr<TextBox> createTextBox() = 0;
    };

    class WinFactory : public GUIFactory
    {
    public:
        std::unique_ptr<Button> createButton() override
        {
            return std::make_unique<WinButton>();
        }
        std::unique_ptr<TextBox> createTextBox() override
        {
            return std::make_unique<WinTextBox>();
        }
    };

    class MacFactory : public GUIFactory
    {
    public:
        std::unique_ptr<Button> createButton() override
        {
            return std::make_unique<MacButton>();
        }
        std::unique_ptr<TextBox> createTextBox() override
        {
            return std::make_unique<MacTextBox>();
        }
    };
```
- 介绍模板方法模式⭐⭐⭐⭐⭐
	- 基类确定整体方法调用流程，具体实现不同子类不一样
``` cpp
    // 抽象父类：定义算法骨架
    class AbstractCook
    {
    public:
        virtual ~AbstractCook() = default;

        // 【模板方法】禁止子类重写，保证流程固定！
        void cook()
        {
            prepare(); // 准备材料（可变）
            fry();     // 翻炒（可变）
            plate();   // 装盘（可变）
            std::cout << "===== 菜品制作完成 =====\n\n";
        }

    protected:
        // 可变步骤：交给子类实现
        virtual void prepare() = 0;
        virtual void fry() = 0;
        virtual void plate() = 0;
    };

    // 具体子类1：番茄炒蛋
    class TomatoEgg : public AbstractCook
    {
    protected:
        void prepare() override
        {
            std::cout << "准备：番茄、鸡蛋";
        }
        void fry() override
        {
            std::cout << " → 热油，先炒鸡蛋再放番茄翻炒\n";
        }
        void plate() override
        {
            std::cout << "装盘：番茄炒蛋成品\n";
        }
    };

    // 具体子类2：青椒肉丝
    class PepperPork : public AbstractCook
    {
    protected:
        void prepare() override
        {
            std::cout << "准备：青椒、肉丝";
        }
        void fry() override
        {
            std::cout << " → 爆香肉丝，加入青椒大火翻炒\n";
        }
        void plate() override
        {
            std::cout << "装盘：青椒肉丝成品\n";
        }
    };
```
- 介绍策略模式⭐⭐⭐⭐⭐
	- 目标对象持有并运用抽象策略，不同类实现不同策略
``` cpp
// 抽象策略
class PaymentStrategy {
public:
    virtual ~PaymentStrategy() = default;
    virtual void pay(double money) = 0;
};

// 具体策略：微信支付
class WechatPay : public PaymentStrategy {
public:
    void pay(double money) override {
        std::cout << "微信支付：" << money << "元\n";
    }
};

// 具体策略：支付宝支付
class AliPay : public PaymentStrategy {
public:
    void pay(double money) override {
        std::cout << "支付宝支付：" << money << "元\n";
    }
};

// 上下文：订单
class OrderContext {
private:
    std::shared_ptr<PaymentStrategy> strategy;
public:
    void setStrategy(std::shared_ptr<PaymentStrategy> s) {
        strategy = std::move(s);
    }
    void doPay(double money) {
        if(strategy) {
            strategy->pay(money);
        } else {
            std::cout << "未设置支付策略\n";
        }
    }
};
```
- 介绍装饰器模式⭐⭐⭐⭐⭐
	- 抽象装饰器与目标继承同一个接口并持有这个抽象接口，利用多态实现这个接口的方法，动态为目标添加功能
``` cpp
class Component {
public:
    virtual void op() = 0;
};
class Concrete : public Component {
public:
    void op() override {}
};
class Decorator : public Component {
    Component* inner;
public:
    void op() override {
        before();
        inner->op(); // 一定传给内层，不会拦截
        after();
    }
};
```
- 介绍门面模式⭐⭐⭐⭐⭐
	- 一个门面类持有所有子系统类的对象，完成统一封装
``` cpp
// 子系统：灯光
class Light {
public:
    void dim() { std::cout << "灯光调暗\n"; }
    void bright() { std::cout << "灯光调亮\n"; }
};

// 子系统：投影仪
class Projector {
public:
    void on() { std::cout << "投影仪开机\n"; }
    void off() { std::cout << "相机关机\n"; }
};

// 子系统：音响
class Sound {
public:
    void on() { std::cout << "音响开启\n"; }
    void off() { std::cout << "音响关闭\n"; }
};

// 子系统：播放器
class Player {
public:
    void play() { std::cout << "开始播放电影\n"; }
    void stop() { std::cout << "停止播放\n"; }
};

// 门面：家庭影院，封装整套流程
class HomeTheaterFacade {
private:
    Light light;
    Projector projector;
    Sound sound;
    Player player;
public:
    void watchMovie() {
        std::cout << "=====开始观影=====\n";
        light.dim();
        projector.on();
        sound.on();
        player.play();
    }

    void endMovie() {
        std::cout << "=====结束观影=====\n";
        player.stop();
        sound.off();
        projector.off();
        light.bright();
    }
};
```
- 介绍代理模式⭐⭐⭐
	- 和装饰器很像，区别在装饰器一定不会拦截，而代理有可能拦截，进行权限控制
``` cpp
class Subject {
public:
    virtual void request() = 0;
};
class Real : public Subject {
public:
    void request() override { /* 真实逻辑 */ }
};
class Proxy : public Subject {
    Subject* real;
public:
    void request() override {
        if (!checkAuth()) return; // 可以拦截，不调用真实对象
        real->request();
    }
};
```
- 介绍观察者模式⭐⭐⭐⭐
	- 被观察者与观察者一对多，被观察者持有抽象观察者，提供订阅入口，观察者订阅后，被观察者一发布就通知观察者（调用观察者方法）
``` cpp
// 抽象观察者
class Observer {
public:
    virtual ~Observer() = default;
    virtual void update(float temp, float humidity) = 0;
};

// 抽象主题（被观察者）
class Subject {
protected:
    std::vector<std::weak_ptr<Observer>> observers;
public:
    virtual ~Subject() = default;
    void attach(std::shared_ptr<Observer> obs) {
        observers.push_back(obs);
    }
    void detach(std::shared_ptr<Observer> obs) {
        for(auto it = observers.begin(); it != observers.end(); ){
            auto sp = it->lock();
            if(!sp || sp == obs){
                it = observers.erase(it);
            }else{
                ++it;
            }
        }
    }
    virtual void notify() = 0;
};

// 具体主题：天气数据
class WeatherData : public Subject {
private:
    float temp{0};
    float humidity{0};
public:
    void setMeasure(float t, float h) {
        temp = t;
        humidity = h;
        notify(); // 状态变化，推送通知
    }
    void notify() override {
        for(auto& wk : observers) {
            if(auto sp = wk.lock()) {
                sp->update(temp, humidity);
            }
        }
    }
};

// 具体观察者：手机APP
class PhoneApp : public Observer {
public:
    void update(float temp, float humidity) override {
        std::cout << "[手机APP] 温度:" << temp << "℃ 湿度:" << humidity << "%\n";
    }
};

// 具体观察者：户外大屏
class ScreenDisplay : public Observer {
public:
    void update(float temp, float humidity) override {
        std::cout << "[户外大屏] 更新天气 " << temp << "℃\n";
    }
};
```
- 介绍适配器模式⭐⭐⭐
	- 适配器继承新接口，持有旧接口，完成接口转换功能
``` cpp
// 被适配者：旧圆孔充电器（已有老组件，接口不匹配）
class RoundCharger {
public:
    void roundPlug() {
        std::cout << "圆孔充电器输出电流\n";
    }
};

// 目标接口：客户端期望的TypeC接口
class TypeC {
public:
    virtual ~TypeC() = default;
    virtual void typeCPlug() = 0;
};

// 对象适配器：组合持有旧充电器，完成接口转换
class ChargerAdapter : public TypeC {
private:
    std::shared_ptr<RoundCharger> adaptee;
public:
    explicit ChargerAdapter(std::shared_ptr<RoundCharger> r) : adaptee(std::move(r)) {}
    void typeCPlug() override {
        std::cout << "适配器转换接口 → ";
        adaptee->roundPlug(); // 调用被适配者原有逻辑
    }
};
```
# 数据结构 & 算法
- 排序有哪几种？时间复杂度？空间复杂度？稳定性？⭐⭐⭐⭐⭐
	- 快排、堆、选择不稳定，其他都稳定
	- 冒泡：往后沉
	- 选择：每次在后面选最小的
	- 插入：打牌整理手牌，往前面有序数组插
	- 归并排序：二分排好后合并
	- 快速排序：找基准，小的大的放两边
	- 堆排序：大小根堆性质
	- 计数排序：遍历一直到最大数字
	- 基数排序：按位数一轮一轮排序，从低到高排序
	- 桶排序：多个区间作为桶，桶内部排序后合并

| 排序        | 最好        | 平均        | 最坏        | 空间           | 稳定   |
| --------- | --------- | --------- | --------- | ------------ | ---- |
| 冒泡排序      | O(n)      | O(n2)     | O(n2)     | O(1)         | ✅稳定  |
| 选择排序      | O(n2)     | O(n2)     | O(n2)     | O(1)         | ❌不稳定 |
| 插入排序      | O(n)      | O(n2)     | O(n2)     | O(1)         | ✅稳定  |
| 希尔排序（不考虑） | O(n)      | O(n1.3)   | O(n2)     | O(1)         | ❌不稳定 |
| 归并排序/分治   | O(nlogn)  | O(nlogn)  | O(nlogn)  | O(n)         | ✅稳定  |
| 快速排序      | O(nlogn)  | O(nlogn)  | O(n2)     | O(logn)∼O(n) | ❌不稳定 |
| 堆排序       | O(nlogn)  | O(nlogn)  | O(nlogn)  | O(1)         | ❌不稳定 |
| 计数排序      | O(n+k)    | O(n+k)    | O(n+k)    | O(n+k)       | ✅稳定  |
| 基数排序      | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k)       | ✅稳定  |
| 桶排序       | O(n)      | O(n)      | O(n2)     | O(n+m)       | ✅稳定  |
- 二叉树有哪些类型？
	- 普通二叉树
	- 满二叉树：叶子同一层
	- 完全二叉树：叶子左边全部满
	- 二叉搜索树：左<根<右，极端会退化为链表
	- 平衡二叉搜索树：左右子树高度差<=1
- 什么是一致性哈希？⭐⭐⭐
	- 解决问题：节点扩容缩容数据重新哈希大量迁移，缓存雪崩
	- 把哈希空间连成一个环
	- 把服务节点算哈希映射到环上
	- 把数据算哈希映射到环上，顺时针存在第一个遇到的节点
	- 节点扩容缩容只迁移一小段区域数据，但是有可能导致节点负载不均，解决方法：一个真实节点映射为多个虚拟节点散列到不同位置
- 手搓快排⭐⭐⭐⭐⭐
``` cpp
    vector<int> sortArray(vector<int>& nums) {
        QuickSort(nums,0,nums.size()-1);
        return nums;
    }
    void QuickSort(vector<int>& arr,int l,int r){
        if(l>=r) return;
        int rand_id = l+rand()%(r-l+1);
        swap(arr[rand_id],arr[r]);
        int middle = Partition(arr,l,r);
        QuickSort(arr,l,middle-1);
        QuickSort(arr,middle+1,r);
    }
    int Partition(vector<int> &arr, int l, int r){
        int val = arr[r];
        int i = l;
        int j = r-1;
        while(i<=j){
            while(i<=j&&arr[i]<=val) i++;
            while(i<=j&&arr[j]>=val) j--;
            if(i<=j) swap(arr[i],arr[j]);
        }
        swap(arr[i],arr[r]);
        return i;
    }
```
- 介绍布隆过滤器⭐⭐⭐
	- 就是一个位图，通过不同的哈希函数将一个数据映射到位图的几个位置，可以快速判断集合内有没有这个数据，可能有假阳性不可能有假阴性
- 介绍红黑树⭐⭐⭐⭐⭐
	- 特性：
		- 非平衡二叉搜索树
		- 节点红色黑色
		- 根节点黑色
		- 叶节点黑色
		- 红色不能相连
		- 黑高相同
	- 基本操作：左旋右旋
	- 插入：
		- 新节点红色挂到叶子上
		- 如果冲突看叔叔颜色，如果黑色可能要旋转
	- 删除（复杂）：
		- 叶子节点直接删，左右单孩子用孩子替代，孩子齐全用右子树最小节点替代
		- 红色直接删，黑色需要修复
		- 黑色需要看兄弟颜色和兄弟的孩子颜色，会有旋转
``` txt
左旋
        X                    Y
      /   \                /   \
     T1    Y     ───▶     X     T3
          / \            / \
         T2  T4         T1 T2
右旋
        Y                    X
      /   \                /   \
     X    T4    ───▶     T1    Y
    / \                        / \
   T1 T2                      T2 T4
```
- 介绍B+树⭐⭐⭐⭐
	- 特性：
		- 多路平衡搜索树
		- 子节点数=key数+1
		- m阶内部节点数：\[ceil(m/2),m\]
		- 非叶子节点只存索引，叶子节点存数据
	- 与B树区别：
		- 只有叶子有数据
		- 叶子间有链表连接
		- 顺序存储性能更高
	- 插入：
		- 找到叶子插入
		- 如果满了就向上对半分裂成两个叶子，更新链表
	- 删除：
		- 找到叶子删除
		- 如果缺了就左右兄弟借一个过来，如果兄弟也没有就和兄弟合并成一个叶子，更新链表
- 介绍跳表⭐⭐⭐
	- 特性：
		- 多层有序链表+随机层数
		- 底层链表保存全部数据，上层链表加快查询
	- 查找：从顶层开始往右，不满足条件往下
	- 插入：前驱数组记录查找过程的前驱，生成新节点随机生成层数（可能升高整体层数），每层连接前驱
	- 删除：前驱数组记录查找过程的前驱，如果有就删除（检查head降低整体层数），每层前驱跳过
- 字符串算法（todo）
	- KMP算法
	- Manacher（少用）
- 图论算法（todo）
	- DFS
	- BFS
	- Dijkstra
	- Kruskal
	- Prim
	- 拓扑排序算法
	- 并查集
# C++语言
- 5大编程范式？
	- 面向过程：按流程写代码
	- 面向对象：封装、继承、多态
	- 函数式：函数组合完成计算
	- 泛型编程：模板，复用
	- 模板元编程：模板，在编译期执行计算、类型变换
- C++是静态还是动态类型语言？⭐⭐⭐⭐⭐
	- 静态，编译阶段类型检查
- C++哪几个部分组成？
	- C语言
	- 面向对象C++
	- 模板
	- STL
- C++有哪些缺陷？哪些不建议用？⭐⭐⭐⭐
	- 缺少自动内存回收
	- 隐性性能损耗难察觉
	- 调试难度大
	- 缺少工程化工具
	- RTTI、异常、虚继承不建议用
- C++11的新特性？⭐⭐⭐⭐⭐
	- auto、decltype
	- 右值引用&&
	- 列表初始化
	- 智能指针
	- lambda表达式
	- constexpr
	- unordered_map、unordered_set
	- thread库
- 程序入口模板？
	- int main(int argc, char \*\* argv)
	- 返回值0表示成功，-1失败
- C++有哪些可调用对象？类似函数的对象？⭐⭐⭐⭐
	- function对象
	- lbamda表达式（匿名函数）
	- 函数指针
	- bind绑定函数
	- 函数类
- 可调用对象与谓词区别？⭐⭐⭐⭐
	- 谓词是可调用对象的子集，返回值必须是bool类型
- 常用函数模板
	- greater<>
	- equal_to<>
	- plus<>
- 名字查找先于类型检查什么意思？⭐⭐⭐
	- 就是内层作用域先寻找名字，同名会覆盖，无视之前名字的类型
- 声明与定义有什么区别？
	- 声明可以多次，定义可以多次，建议在第一次使用时定义
- 怎么判断变量类型？
	- 从左往右读，最接近变量名的符号是变量类型
- 变量初始化有哪些类型？⭐⭐⭐⭐⭐
	- 默认初始化
	- 值初始化()
	- 拷贝初始化=
	- 列表初始化{}
	- 零初始化：全局、静态变量
- 哪些必须初始化？⭐⭐⭐⭐⭐
	- 所有变量都建议初始化，否则局部变量默认初始化会出现不定值
	- 引用、const、auto、lambda引用捕获都必须初始化
- 哪些禁止拷贝？哪些可以移动？⭐⭐⭐⭐
	- 禁止拷贝：内置数组、unique_ptr、锁、thread、流对象、atomic、异步任务、自定义类
	- 可以移动：unique_ptr、流对象、thread、异步任务
- 深拷贝与浅拷贝区别？⭐⭐⭐⭐⭐
	- 浅拷贝：只拷贝值，指针复制共用一块内存
	- 深拷贝：除了值，指针指向的内存也拷贝一份
- 常见内置类型大小？⭐⭐⭐⭐⭐
	- bool：1B
	- char：1B，是否带符号不一定，建议用unsigned char
	- short：2B
	- int：4B
	- long long：8B
	- double：8B
- 介绍static对象？⭐⭐⭐⭐⭐
	- 生命周期都是全程，都放到数据段（.data/.bss）
	- 全局static：作用域本文件（匿名namespace替代）
	- 局部static：作用域本函数
	- 类内static：作用域本类，类实例共享，类内声明类外定义（不带static），访问控制符控制
		- 变量：常与const联用
		- 函数：常与constexpr联用，不能用const、virtual，不带this指针
	- 什么时候构造析构？全局、类都是main之前构造，局部是延迟构造
	- const static会放到.rodata段
- 字面值有哪些？字面值放在哪？
	- 字面值放在.rodata或直接内嵌.text代码段
	- 整数字面值：0x开头和普通数字
	- 二进制字面值：0b开头
	- 浮点字面值：e科学计数法和普通实数
	- 字符字面值：'a'、转义字符、宽字符
	- 字符串字面值："hello"，底层是const char*
	- 布尔字面值：true/false
	- 指针字面值：nullptr
- 不完全类型定义？哪些可以用不完全类型？
	- 有声明没定义的类型，比如类、内置数组、void、枚举、模板类
	- 指针引用、静态成员、别名可以用
- 左值引用、右值引用、const左值引用、值传递、万能引用区别？⭐⭐⭐⭐⭐
	- 引用是别名，必须初始化
	- 值传递：不可改变外部值，拷贝
	- 左值引用：可改变外部值，不接收右值
	- const 左值引用：不可改变外部值，接收右值
	- 右值引用：可改变外部值，不接收左值，本身是左值，常用于移动操作
	- const 右值引用：极少使用
	- 万能引用：模板T&&，左右都可，依靠引用折叠，可配合std::forward完美转发（还原左右属性、const属性、volatile属性）
- 引用折叠？
	- 两个引用只要有一个左值引用结果就是左值引用，只有全是右值引用结果才是右值引用
- 哪些不支持引用？
	- 引用的引用
	- 数组元素
	- 容器元素
	- 动态分配
- 左值右值区别？哪些是右值？⭐⭐⭐⭐⭐
	- 左值持久右值短暂，右值没地址（寄存器或内存中）
	- 右值：
		- 纯右：字面值、运算临时值、函数返回值、后自增自减
		- 将亡：std::move后，一定在内存
- std::move底层怎么实现的？
	- 万能引用（引用折叠）
	- std::forward还原变量属性进行完美转发
- 别名方法？⭐⭐⭐⭐⭐
	- auto：忽略顶层const和引用，函数数组只获得指针，常与范围for联用
	- decltype：获得完全类型
	- using：常用来取别名
	- typedef：老用法，很少用
- 顶层const和底层const？⭐⭐⭐⭐⭐
	- 顶层const：认为本身是常量
	- 底层const：认为指向对象是常量（实际可以不是）
- 指针引用复合？const复合？⭐⭐⭐⭐
	- 指向指针的指针：`int **p`
	- 指向指针的引用：`int *&p`
	- 指针或引用必须指向对象，引用是别名不是对象
	- 常量引用：不存在
	- 对常量的引用（简称常量引用）：`const int &p = 42;`
	- 常量指针：`int *const pp = &p;`
	- 指向常量的指针：`const int *p = &r;`
- const与constexpr的区别？⭐⭐⭐⭐
	- 常量表达式只包含整型const，非整型的const不算，包含全部的constexpr
	- const：认为是常量，没校验
	- constexpr：编译器确定的百分百的常量
- 迭代器分类？⭐⭐⭐⭐⭐
	- 按顺序：输入（istream_iterator）、输出（ostream_iterator）、前向（forward_list、无序容器）、双向（list、关联容器）、随机访问（vector、string、array、deque）
	- 按功能：一般（begin/end）、反向（rbegin/rend）、只读（cbegin/cend）、插入（back_inserter/inserter）、流（iostream那些）、移动（少用）
	- 前向迭代器和输入输出迭代器不支持反向，插入也不支持反向
- 运算符表达式求值顺序有什么要注意的？
	- 不确定就用括号
	- 求值顺序：&&、||、?:、,
- 隐式类型转换分类？⭐⭐⭐⭐⭐
	- 编译器只做一步隐式类型转换
	- 非bool转bool
	- 整型提升
	- 算数转换（有符号无符号、整型浮点型）
	- 函数、数组转指针
	- 非const转const
	- 类（运算符重载推荐explicit、转换构造函数）
	- 派生类基类转换
- 哪些时候会隐式类型转换？⭐⭐⭐
	- 初始化、拷贝
	- 传参
	- 返回
	- 运算
	- 条件判断
- 派生类和基类的转换？⭐⭐⭐⭐⭐
	- 派生类对象 = 基类对象：不允许
	- 基类对象 = 派生类对象：可以，切割
	- 派生类指针 = 基类：不可以，必须使用static_cast（不安全）或dynamic_cast（安全）
	- 基类指针 = 派生类：可以，多态基础
	- 访问这些对象会受到访问控制符的控制
- 显式类型转换（强转）分类？⭐⭐⭐⭐⭐
	- static_cast：数值、`void*`指针、父子类
	- const_cast：增删顶层、底层const和volatile属性
	- dynamic_cast：父子类（安全）
	- reinterpret_cast：底层二进制
- namespace的分类？
	- .h和.cpp都要有，.h里using namespace std必须放外面，.cpp必须放里面，#include都放外面
	- 匿名
	- 嵌套
	- 内联：新代码内联，旧代码嵌套
- using声明和指示区别？
	- using声明：单个名字
	- using指示：全部namespace
- 异常写法？哪些要声明noexcept？⭐⭐⭐
	- try、catch、finally、throw
	- noexcept同时出现在声明和定义，如果异常直接终止程序
	- 移动函数、析构函数、swap函数一般声明成noexcept
- 析构函数为什么禁止抛出异常？有异常咋办？⭐⭐⭐⭐⭐
	- 销毁对象再次抛出异常，变两个异常了
	- 销毁未完成，资源未释放
	- 解决：
		- 直接终止
		- 记录异常不抛出
		- 引起异常的操作隔离出新函数
- 所有变量函数的前缀后缀修饰符？
	- 变量：template<> extern/static/thread_local mutable constexpr/constinit const/volatile xxx
	- 函数：template friend/extern explicit virtual/static inline constexpr/consteval xxx const/volatile &/&& noexcept override/final
	- mutable与static、constexpr、const冲突
	- static函数与virtual、const、&/&&、override/final、extern冲突
	- 虚函数理论可以内联但是工程上不建议内联
- 类内声明类外定义不用写的修饰符？⭐⭐⭐⭐
	- static
	- virtual
	- override/final
	- explicit
	- friend
	- mutable（类内定义）
- 默认形参？可变形参？尾置返回？
	- 默认形参：静态绑定，位置靠后
	- 可变形参：C风格省略符、同类型initializer_list、不同类型可变参数模板（类型参数包、包扩展）
	- 尾置返回：配合auto使用
- inline函数和constexpr函数的特点？
	- inline只是请求，具体看编译器实不实现，原地展开，调试不友好
	- 隐式内联，一条return，参数和返回值都是常量表达式
- 函数重载的定义？怎么匹配到目标函数？
	- 函数名称一样，数量类型不一样，静态多态，顶层const参数视为一样，函数const和&/&&可以区分
	- 匹配：
		- 完全匹配
		- 仅顶层const、数组函数转指针
		- 整型提升
		- 算数转换
		- 底层const
		- 类
		- 可变参数列表
		- 二义性调用
- 类的作用域和定义？
	- 直到遇到作用域运算符以后才进入类的作用域中，所以返回值有时候也要用作用域运算符
	- 定义完成后要加分号
	- 编译器声明先于定义
	- 可以前向声明
- this指针特点？⭐⭐⭐⭐⭐
	- 指向自身实例的顶层const指针，非const函数自带的隐性参数
	- const函数自带的this指针应当底层也是const的，所以只有const实例能调用，非const函数调用const函数，this指针会增加底层const
	- 函数&/&&限定符可以限定this指针的左右值
- 访问控制符有哪些？继承访问控制符有哪些？⭐⭐⭐⭐⭐
	- 默认class是private，struct是public
	- 访问控制符：
		- public：用户可访问
		- protected：内部且类的友元和子类可访问
		- private：仅内部访问
	- 继承访问控制符：
		- 基类private都是不可访问的
		- public：基类public-public
		- protected：都是protected
		- private：都是private
		- using可以改变继承访问级别，但只能升高不能降低
- 类的友元可以传递或继承吗？⭐⭐⭐
	- 不行，而且只是个性质通知，不是传统的函数声明，函数声明需要在其他地方声明
- 类的覆盖和重写的区别？⭐⭐⭐⭐⭐
	- 覆盖：重写普通函数，会完全覆盖父类同名函数，使用using引入父类重载函数并单独覆盖
	- 重写：重写虚函数
- 构造函数初始化列表初始化与一般初始化区别？⭐⭐⭐⭐⭐
	- 初始化列表：真正初始化
	- 一般初始化：拷贝初始化
- 构造函数类型？
	- 默认构造函数
	- 委托构造函数
	- 转换构造函数（explicit）
	- 拷贝和移动构造函数
	- 继承构造函数（using）
- 为什么构造和析构函数禁止使用虚函数？⭐⭐⭐⭐⭐
	- 因为构造和析构过程多态失效
	- 如果真用上了子类的方法，则UB
- 拷贝成员5法则？
	- 5法则：要么一个都不自定义，要么全都自定义
	- =default使用自动合成成员
	- = delete阻止使用
	- 一般对象一般不用，RAII对象裸指针的话必须实现5个成员
	- 拷贝注意自赋值，移动注意移后源置空
	- 析构函数（noexcept），静态成员不参与析构
	- 拷贝构造函数
	- 拷贝赋值运算符
	- 移动构造函数（noexcept）
	- 移动赋值运算符（noexcept）
	- copy and swap（推荐非成员函数swap）技巧可以统一拷贝和移动赋值运算符，但是高性能情况下别用
- 为什么基类都要自定义虚析构？⭐⭐⭐⭐⭐
	- 因为有可能new一个新对象，没法自动回收资源
	- 因为自定义了析构，所以5个成员都需要自己实现
- 对象销毁情形？⭐⭐⭐⭐
	- 离开作用域
	- delete
	- 全局对象程序结束
	- 右值对象
	- 类和容器元素
	- 智能指针
- 常用运算符重载
	- <<、>>、\==、bool、<，注意explicit
- 聚合类？字面值类？RAII类？
	- 聚合类：默认public，无继承多态
	- 字面值类：至少一个constexpr构造函数，成员全是字面值或constexpr函数
	- RAII类：一般用智能指针管理底层动态内存不需要自定义拷贝移动成员，分成像unique_ptr类和像shared_ptr类，尽量不用裸指针
- 常用哪些STL？⭐⭐⭐⭐⭐
	- vector、string、array、list、unordered_set、unordered_map、stack、queue、priority_queue
- stl容器底层数据结构：⭐⭐⭐⭐⭐
	- vector：连续数组
	- deque：分块数组
	- stack：deque
	- queue：deque
	- priority_queue：vector
	- map/set：红黑树
	- unordered_map/unordered_set：哈希表
- vector与内置数组区别？⭐⭐⭐⭐⭐
	- 变长
	- vector在堆上，内置数组可堆可栈
	- 动态内存安全
- vector怎么动态扩容？⭐⭐⭐⭐⭐
	- 找一块翻倍大的堆内存
	- 原内存元素拷贝或移动过去
	- 释放原内存
- vector的resize与reserve区别？size与capacity区别？⭐⭐⭐⭐⭐
	- size：元素个数
	- capacity：底层容量，内存预分配
	- reserve：改变capacity
	- resize：改变size，不够就扩容
- vector怎么转内置数组？⭐⭐⭐⭐
	- v.data()
- vector迭代器失效怎么办？⭐⭐⭐⭐⭐
	- 使用it保存erase返回的新迭代器
- list与vector各自优势在哪？⭐⭐⭐⭐⭐
	- list：中间增删
	- vector：随机访问、遍历
- map的key有什么要求？⭐⭐⭐⭐⭐
	- key 要支持<重载，严格弱序
	- unordered_map还要求哈希函数和=重载
- 有vector为什么还要array？⭐⭐⭐⭐⭐
	- 想固定长度、栈分配，还支持STL通用接口
- push_back与emplace_back区别？⭐⭐⭐⭐⭐
	- push_back：拷贝对象
	- emplace_back：原地构造
- hash 冲突原因？怎么解决？⭐⭐⭐
	- 哈希函数结果一样
	- 解决：拉链法（哈希表+链表）、下一个空桶、扩容rehash、优化哈希函数
- 迭代器、裸指针、智能指针区别？⭐⭐⭐⭐⭐
	- 裸指针：内存地址
	- 智能指针：RAII封装裸指针，安全，自动释放
	- 迭代器：容器位置游标，底层不一定是指针
- 什么情况会导致迭代器失效？⭐⭐⭐⭐
	- 连续容器：扩容全部失效、删除后面失效
	- 链表红黑树：删除迭代器失效
	- 哈希表：rehash全失效
	- 所以严禁循环里对容器大小进行改变
- switch语句参数不能是什么类型？
	- 只能是整型（char、int、long long）、枚举、bool
- C++怎么消除循环依赖？⭐⭐⭐
	- 前置声明
	- 源文件放实现
	- 重新设计避免依赖
- 常用哪些标准库设施？
	- bitset、pair、tuple、enum
	- random随机数库理论更好实际不咋用（又是引擎又是分布太复杂了）
- STL的一些常用操作？⭐⭐⭐⭐
	- empty、size、resize、emplace、emplace_back、erase、pop_back、pop、push、top、insert
	- list：splice
	- string：substr、find
- 常用的泛型算法？⭐⭐⭐⭐
	- 泛型算法基本都是基于迭代器的
	- find、fill_n、sort、stable_sort、copy、unique、reverse、min_element、lower_bound
- IO库包括哪些类型？
	- iostream：标准IO
	- fstream：文件
	- sstream：字符流，很少用
- 怎么刷新缓冲区？⭐⭐⭐
	- flush、endl
	- unitbuf禁止缓冲
- 条件状态？
	- is.rdstate()返回位图
	- good()、fail()检查
	- 实际上直接当条件判断来使用的
- 文件模式？⭐⭐⭐⭐
	- in：读
	- out：写
	- trunc：从头
	- app：从尾
- 未格式化IO有哪些方法？
	- peek、unget、putback
	- seek、tell
- map的mapped_type和value_type？迭代器与for遍历与下标？⭐⭐⭐⭐⭐
	- mapped_type：pair<key,value>
	- value_type：value
	- 迭代器返回的是指针，mapped_type，需要->second
	- 遍历返回的是值，mapped_type，需要.second
	- 下标返回的是值，value_type，本身就是value值引用
- 面向对象的特点？⭐⭐⭐⭐⭐
	- 用指针而不是对象
	- 封装、继承、多态
- RTTI的基本概念？
	- 派生类构造基类部分以及之前视为基类，销毁派生类部分以后也视为基类
	- dynamic_cast和typeid
- 派生类构造函数怎么实现？
	- 推荐用初始化列表调用
	- 也可以使用using 基类::构造函数名，访问权限using所在位置控制，保留基类的explict和constexpr，派生类数据成员默认初始化
- 虚函数的使用方法有哪几类？
	- 模板方法模式：public继承基类，private虚函数
	- 策略模式：持有基类，public虚函数
- 虚函数与实函数区别？⭐⭐⭐⭐⭐
	- 虚函数提供接口，实函数提供实现
	- 虚函数必须提供定义（基类也是）
	- 虚函数的默认实参静态绑定
	- 关键词virtual和override
- 介绍一下纯虚函数和抽象基类？⭐⭐⭐⭐⭐
	- 纯虚函数：=0，可以不提供实现，工程上一般也会实现
	- 抽象基类：有纯虚函数没实现的类，看作接口不能创建对象
	- C继承B，C只需要补充实现B未实现的纯虚函数就行了
- 虚函数的底层原理是什么？⭐⭐⭐⭐⭐
	- 虚函数表和虚指针（8B），有虚函数就会有
	- 虚函数表在.rodata段，存储虚函数地址
	- 虚指针构造时指向虚函数表
	- 多态就是运行时通过虚指针查找真实使用的函数地址
	- 常考对象内存：vptr8+成员+字节对齐+EBO空白类优化
- 静态绑定与动态绑定区别？⭐⭐⭐⭐
	- 静态绑定：编译期确定类型
	- 动态绑定：运行期确定类型
- 派生类会继承基类的拷贝成员吗？
	- 不会，条件合适自动合成
	- 基类一般虚析构+2default拷贝+2delete移动
- C++支持多继承吗？
	- 支持，但是一般不用，容易歧义指明作用域
- 虚继承概念？⭐⭐⭐⭐⭐
	- 解决菱形继承二义性问题
	- 中间层在继承时用virtual修饰
	- 构造析构：先构造虚基类，再构造基类，再构造派生类，析构同理
	- 中间层有虚基类指针（8B）+虚基类表，继承后也会继承虚基类指针
	- 会带来更多的额外开销，不推荐使用！
- final加在哪里？有什么作用？⭐⭐⭐
	- 类名后：阻止类被继承
	- 虚函数后：阻止子类重写
- 模板编程的参数类型？⭐⭐⭐⭐
	- 类型参数、非类型参数、默认形参
- 模板的类型？⭐⭐⭐⭐
	- 函数模板：可以隐式推断
	- 类模板：模板类可以部分实例化，不能隐式推断
- typename与class区别？⭐⭐⭐⭐⭐
	- 模板编程里typename与class声明参数效果一致
	- 如果T::xxx，编译器表示分不清这是类型还是static变量，前面加typename可以告诉编译器这是类型
- 模板友元绑定类型？⭐⭐⭐
	- 无限制：所有实例可以访问
	- 一对一：仅对应类型实例可以访问
	- 多对多：所有实例可以访问
- 模板类与虚函数？
	- 模板类可以有虚函数，编译期确定类型
	- 但模板成员函数不能是虚函数（无数种可能）
- 模板支持的实参推断？⭐⭐⭐⭐⭐
	- 数组、函数转指针
	- const转换
	- 其他都不支持
- 模板重载函数匹配规则？
	- 非模板函数
	- 特化模板函数
	- 非模板函数隐式转换、普通模板函数
	- 二义性
- 标准库常用类型转换模板type_traits？
	- type_traits是模板元编程的重要基础
	- remove_reference：删除引用
	- remove_pointer：删除指针
	- remove_extend：删除数组
	- 有很多对类别、属性、类型关系这些元数据的操作
- 显式实例化与特例化区别？⭐⭐⭐⭐
	- 显式实例化：根据模板生成实例代码，<类别>
	- 特例化：每个模板的自定义实现，<>，类支持偏特化（部分特例化）
- 动态内存分配方式？⭐⭐⭐⭐
	- 裸指针new/delete分配（包括new[]/delete[]）
	- 智能指针分配
	- allocator分配
- extern C作用？⭐⭐⭐
	- 让C++调用C库或C调用C++函数
	- 关闭 C++ 名字修饰，保证链接符号一致
- volatile作用？⭐⭐⭐⭐
	- 禁止编译器对变量读写优化，禁止寄存器缓存
- volatile可以与const共用吗？
	- 可以
- new与malloc的区别？⭐⭐⭐⭐⭐
	- new是关键字，malloc是库函数
	- new构造，malloc只分配内存
	- new失败抛异常，malloc失败返回nullptr
	- new支持定位new，malloc不行
- 智能指针常用方法？
	- 创建：直接类型+{}，或者make_shared<>、make_unique<>
	- get、use_count
	- 还可以自定义删除器，不过shared_ptr放到初始化参数里，unique_ptr放到模板参数里
- unique_ptr 怎么实现独占内存？⭐⭐⭐⭐⭐
	- 禁止拷贝，支持移动
- allocator基础使用？
	- allocate分配内存
	- construct构造对象
	- destroy销毁对象
	- deallocate销毁内存
- 枚举类型？
	- 有作用域：enum class
	- 无作用域：enum
- C风格字符串与string的转换
	- string的长度是不包括'\0'的，但是string转换得到的c风格字符串的指针是包括'\0'的
	- string转c风格字符串：.c_str
	- c风格字符串转string：string()
- sizeof与strlen的区别？⭐⭐⭐⭐⭐
	- sizeof：字节数
	- strlen：字符数（一个字符可能多个字节）
- lambda表达式的结构？⭐⭐⭐⭐
	- `[]()->返回值{}`
	- 捕获列表：捕获外部变量，用于有的谓词把参数列表限定死了的情况
		- 值捕获：=
		- 引用捕获：&
	- mutable可以放到参数列表后，值捕获也能修改变量值
- bind参数绑定的概念？
	- `auto g = bind(f, a, b, _2, c, _1);`
	- `_n`占位符表示绑定后的函数参数位置
- C++20有哪些新特性？⭐⭐⭐⭐⭐
	- 原子操作支持智能指针解决ABA问题
	- ranges范围库，代替泛型算法，支持view管道过滤器和lambda投影函数
	- format格式化
	- 支持协程
	- constexpr增强，consteval限制编译期处理，编译期效率提升
	- 自动合成<=>三路运算符重载
	- concept变量，模板元编程增强
- enum hack什么意思？
	- 类内staic只能类外初始化，enum hack进行类内初始化
- 怎么做分离式编译？⭐⭐⭐⭐
	- 预定义#pragma once
	- namespace分割
- PIPML技巧什么意思？⭐⭐⭐⭐
	- 常见：就是普通类只提供接口并持有Impl类，使用Impl类完成具体实现
	- 继承：普通类设计成抽象基类提供接口，Impl类继承后具体实现
- 跨文件全局静态顺序无定义问题⭐⭐⭐
	- 全局、全局static变量如果不在一个文件初始化顺序是不一定的，使用局部static变量替代全局变量来实现规定顺序的初始化
- 什么是空悬指针？什么是野指针？什么是空指针？⭐⭐⭐⭐⭐
	- 空悬指针：指针指向已经失效的地址
	- 野指针：未初始化有随机垃圾值的指针
	- 空指针：不指向任何对象合法的空指针
- bitwise constness和logical constness概念区别？
	- bitwise constness：编译器只查看对象内存有没有被修改，允许返回指针
	- logical constness：业务对象整体有没有被修改，不允许返回指针
- 什么是奇异递归模板（CRTP）？
	- 模板元编程的重要基础
	- 派生类自身作为模板实参传递给基类模板，实现编译期静态多态
- 模板元编程是啥？举个简单实例？
	- 编译期执行计算、类型变换，把开销前置到编译阶段
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
- shared_ptr循环引用问题？⭐⭐⭐⭐⭐
	- 两个对象内部互相用shared_ptr指向对方，本身也是shared_ptr创建的，销毁后引用计数不为0（因为析构函数没有被调用，里面的指针也就没有删除）
	- 内部用weak_ptr替换，不加引用计数，lock检查
- new dll delete问题？⭐⭐⭐
	- dll有各自的堆，跨dll进行new和delete会崩溃
	- 解决：所有模块链接同一个CRT堆，或者每个dll导出释放接口，里面用智能指针
- new-handler是什么？
	- new抛出异常时会自动调用全局注册的new-handler
- 自定义new/delete运算符有哪些类型？
	- 默认找placement new和普通delete
	- 普通new/delete：
	- nothrow new/delete：参数const std::nothrow_t&，异常直接返回nullptr
	- placement new/delete：除了默认参数以外参数的new/delete运算符，默认在一块已经提前准备好的内存地址上初始化
- 容器和智能指针是线程安全的吗？⭐⭐⭐
	- STL 容器、标准智能指针本身都不是完全线程安全的，只满足最低的只读安全。