---
title: "高效C++系列-STL容器"
slug: "stl-container"
date: 2026-08-28T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["语言特性", "c++", "stl"]
categories: ["技术笔记"]
summary: "STL容器学习笔记，包括一些序列容器、关联容器、容器适配器、迭代器、可调用对象等"
toc: true
comments: true
description: "高效C++-STL容器"
---

- 容器选择：一般选vector或string，头尾插入选deque，中间插入选list，其他根据具体需要选择适配器
- 容器比较器默认前面\<后面
# 通用迭代器
<!-- -->

        vector<int> vec2;
        auto it = vec2.begin();
        auto cit = vec2.cbegin();
        auto rit = vec2.rbegin();
        auto crit = vec2.crbegin();
        rit.base();
        auto iit = inserter(vec2, it);
        auto iit2 = back_inserter(vec2);
        auto iit3 = front_inserter(vec2);
        istream_iterator<int> in_iter2(cin), eof_iter2;
        ostream_iterator<int> out_iter2(cout, " ");
        make_move_iterator(it);

# 可调用对象

- function模板

<!-- -->

    function<int()> f = []()
    { return 42; };

- 函数对象

<!-- -->

    int f()
    {
        return 42;
    }

- 函数指针

<!-- -->

    int (*pf)() = f;

- lambda表达式

<!-- -->

    int v1 = 42;
    auto f1 = [v1]()
    { return v1; };
    auto f2 = [&v1]()
    { return v1; };
    auto f3 = [=]() -> int
    { return v1; };
    auto f4 = [&]() -> int
    { return v1; };
    auto f5 = [v1]() mutable -> int
    { return ++v1; };

- 标准库函数对象

<!-- -->

    plus<int> p1;

- bind参数绑定对象

<!-- -->

    using std::placeholders::_1;
    using std::placeholders::_2;
    int a = 1, b = 2, c = 3;
    void f(int, int &, int, int, int);
    auto f2 = std::bind(f, a, std::ref(b), _2, c, _1);
    f2(7, 8); // equivalent to f(a, b, 8, c, 7)

- 实现了()运算符的类

<!-- -->

    // 自定义类，重载 ()
    class Calculator
    {
    public:
        // 重载无参 ()
        int operator()() const
        {
            return 42;
        }

        // 重载带两个 int 参数的 ()
        int operator()(int a, int b) const
        {
            return a + b;
        }
    };

# 泛型算法

    #include <numeric>
    int val=8;
    vector<int> v(4,5);
    vector<int> c;
    vector<int>::iterator it;
    auto comp = [](int a, int b) { return a < b; };

- 普通算法

<!-- -->

    // 常用算法
    auto result = find(v.cbegin(), v.cend(), val);
    int counts = count(v.cbegin(), v.cend(), val);
    int sum = accumulate(v.cbegin(), v.cend(), 0);
    auto equals = equal(v.cbegin(), v.cend(), c.cbegin());
    fill(v.begin(), v.end(), 10);
    fill_n(v.begin(), v.end(), 20);
    auto retit = copy(v.cbegin(), v.cend(), back_inserter(c));
    replace(v.begin(), v.end(), 10, 100);
    auto ret = lower_bound(v.cbegin(), v.cend(), 5);
    ret = upper_bound(v.cbegin(), v.cend(), 5);
    auto ret2 = equal_range(v.cbegin(), v.cend(), 5);
    int max_val = max({1, 2, 3, 4, 5});
    max_val = *max_element(v.cbegin(), v.cend());
    sort(v.begin(), v.end(), greater<int>());
    stable_sort(v.begin(), v.end());
    partial_sort(v.begin(), v.begin() + 2, v.end());
    std::shuffle(v.begin(), v.end(), std::mt19937{std::random_device{}()});
    reverse(v.begin(), v.end());
    // 不常用算法
    iota(v.begin(), v.end(), 0);
    int min_val = *min_element(v.cbegin(), v.cend());
    auto min_max = minmax_element(v.cbegin(), v.cend());
    nth_element(v.begin(), v.begin() + 2, v.end());
    next_permutation(v.begin(), v.end());
    auto end_unique = unique(v.begin(), v.end());
    set_union(v.cbegin(), v.cend(), c.cbegin(), c.cend(), back_inserter(c));

- 谓词算法

<!-- -->

    end_unique = unique(v.begin(), v.end(), comp);
    auto res = transform(v.cbegin(), v.cend(), back_inserter(c), comp);
    for_each(v.cbegin(), v.cend(), comp);

- if算法

<!-- -->

    find_if(v.cbegin(), v.cend(), comp);

- copy算法

<!-- -->

    replace_copy(v.cbegin(), v.cend(), back_inserter(c), 10, 100);

- is算法

<!-- -->

    is_sorted(v.cbegin(), v.cend());

# 容器通用操作

    vector<int>::iterator it;
    vector<int>::const_iterator cit;
    vector<int>::size_type sz;
    vector<int>::value_type val;
    vector<int>::reference ref = val;
    vector<int>::difference_type diff;
    vector<int> v;
    vector<int> v1(v);
    vector<int> v2(v.begin(), v.end());
    vector<int> v3{1, 2, 3};
    v2 = v1;
    v.size();
    v.max_size();
    v.empty();

# 顺序容器

## vector

    vector<int>::iterator it;
    vector<int>::const_iterator cit;
    vector<int>::size_type sz;
    vector<int>::value_type val;
    vector<int>::reference ref = val;
    vector<int>::difference_type diff;
    vector<int> v(4, 5);
    vector<int> v2;
    v.push_back(10);
    v.emplace_back(20);
    v.insert(v.begin(), 2, 8);
    v.insert(v.begin(), {1, 2, 3});
    v.emplace(v.begin(), 0);
    v.pop_back();
    v.back() = 100;
    v.erase(v.begin(),v.end());
    v.clear();
    v.at(0) = 10;
    v.capacity();
    v.shrink_to_fit();
    v.reserve(10);
    v.resize(5, 0);
    v == v2;
    v < v2;
    vector<vector<int>> vv(3, vector<int>(4, 5));// 3 rows, 4 columns, all elements initialized to 5

## string

    const char *cp = "hello";
    string s(4, '0');
    string s2("value");
    string s3 = "value";
    string s4(cp, 3);    // s4 == "hel"
    string s5(s2, 1, 3); // s5 == "alu"
    s.push_back('!');
    s.insert(s.begin(), 2, 'o');
    s.insert(s.begin(), {'1', '2', '3'});
    s.pop_back();
    s.back() = '!';
    s.erase(s.begin(), s.end());
    s.clear();
    s.at(0) = 'a';
    s.capacity();
    s.shrink_to_fit();
    s.reserve(10);
    s.resize(5, '0');
    s3 = s + s2;
    getline(cin, s);
    s.find('e', 0);
    s.rfind('e', 0);
    s.find_first_of('e', 0);
    s.find_first_not_of('e', 0);
    s.find_last_of('e', 0);
    s.find_last_not_of('e', 0);
    s.substr(0, 3);
    s.append("value");
    s.compare(s2);
    s.compare(0, 3, s2, 1, 3);
    s.assign("整体替换");
    s.replace(s.begin(), s.end(), "部分替换");
    int i = stoi(s, 0, 10);
    double d = stod(s, 0);
    s = to_string(i);
    // c类型字符串
    const char *cc = s.c_str();
    char c = '9';
    char *c1 = "test";
    char c2[] = {'c', '+', '+', '\0'};
    isalnum(c);
    strlen(c2);
    strcmp(cc, c2);
    strncpy(c1, c2, sizeof(c1) - 1);
    strncat(c1, c2, sizeof(c1) - strlen(c1) - 1);

## deque

    deque<int> dq{1, 2, 3};
    dq.push_back(4);
    dq.emplace_back(4);
    dq.push_front(0);
    dq.emplace_front(0);
    dq.insert(dq.begin(), 2, 3);      // 不建议使用
    dq.insert(dq.begin(), {1, 2, 3}); // 不建议使用
    dq.emplace(dq.begin(), 1, 2);
    dq.pop_back();
    dq.pop_front();
    dq.back();
    dq.front();
    dq.erase(dq.begin(), dq.end());
    dq.clear();
    dq.at(0);
    dq.shrink_to_fit();
    dq.resize(5, 7);

## array

- array不支持增删等修改大小的操作

<!-- -->

    #include <array>

    array<int, 5> a;
    a.back();
    a.front();
    a.data(); // 返回数组首地址
    a.at(0);
    a.fill(10);
    array<int, 5> a2(a);
    a2 = a;

## list

    #include <list>

    list<int> l(10, 42);
    l.push_back(0);
    l.emplace_back(1);
    l.push_front(2);
    l.emplace_front(3);
    l.insert(l.begin(), 2, 3);
    l.insert(l.begin(), {4, 5, 6});
    l.emplace(l.begin(), 7);
    l.pop_back();
    l.pop_front();
    l.back() = 0;
    l.front() = 0;
    l.erase(l.begin(), l.end());
    l.clear();
    l.resize(10, 7);

## forward_list

- 难以找到前驱，一般容器是迭代器前操作，本容器是迭代器后操作

<!-- -->

    #include <forward_list>

    forward_list<int> flist{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    auto it = flist.before_begin();
    auto it2 = flist.cbefore_begin();
    flist.front();
    flist.insert_after(it, 2, 8);
    flist.insert_after(it2, {1, 2, 3});
    flist.emplace_after(it, 5);
    flist.erase_after(it, flist.end()); // 删除it之后的所有元素
    flist.resize(5, 7);

# 适配器通用操作

    deque<string> d;
    stack<string, vector<string>> stringStack;
    stack<string>::size_type;
    stack<string>::value_type;
    stack<string>::reference;
    stack<string>::const_reference;
    stack<string>::container_type;
    stack<string> stack1;
    stack<string> stack2(d);
    stack1.empty();
    stack1.size();
    swap(stack1, stack2);

# 容器适配器

## stack(deque)

    #include <stack>

    stack<string> st;
    stack<string,vector<string>> vec_stk;
    st.pop();
    st.top();
    st.push("hello");
    st.emplace("hello");

## queue(deque)

    #include <queue>

    queue<string> q;
    q.pop();
    q.front();
    q.back();
    q.push("");
    q.emplace("");

## priority_queue(vector)

- 需要元素有明确定义的\<运算符

<!-- -->

    priority_queue<string> pq;
    priority_queue<string, vector<string>, greater<string>> pq_reverse; // 小顶堆
    pq.pop();
    pq.push("hello");
    pq.top();
    pq.emplace("world");

# 关联容器

- 关联容器不要用泛型算法
  \## map
- 底层使用红黑树实现

<!-- -->

    #include <map>

    map<string, string>::key_type;
    map<string, string>::mapped_type;
    map<string, string>::value_type;
    map<string, string> m = {{"a", "b"}, {"c", "d"}};
    map<string, string, greater<string>> m2;
    auto m_iter = m.begin();
    *m_iter; // pair<map<string, string>::iterator, bool>
    auto ret = m.insert(m_iter, make_pair("e", "f"));
    ret->first;
    ret->second;
    m.erase("a");
    m.erase(m.begin(), m.end());
    m.at("g") = "h"; // 下标不存在时会插入新元素
    m.find("i");
    m.count("j");
    m.clear();

## set

    #include <set>

    using key_value = set<string>::value_type;
    set<string> s = {"a", "b", "c", "d", "e"};
    set<string, greater<string>> s2;
    auto s_it = s.cbegin();
    *s_it;
    auto ret = s.insert(s_it, "f");
    auto ret2 = s.emplace(s_it, "g");
    ret2.first;  // iterator
    ret2.second; // bool，是否成功
    s.insert({"h", "i"});
    s.erase("c");
    s.erase(s.begin(), s.end());
    s.find("d");
    s.count("d");
    s.clear();

## multimap

- 可重复关键字

<!-- -->

    #include <map>

    multimap<string, string> mm = {{"a", "1"}, {"a", "2"}, {"b", "3"}};
    mm.count("a");
    mm.lower_bound("a");
    mm.upper_bound("a");
    mm.equal_range("a");

## multiset

- 可重复元素

<!-- -->

    #include <set>

    multiset<string> ms = {"a", "b", "c"};
    ms.insert("a");
    ms.count("a");
    ms.lower_bound("a");
    ms.upper_bound("a");
    ms.equal_range("a");

## 无序容器

- 推荐当作正常关联容器使用，无序，但是速度更快
- 自定义hash函数

<!-- -->

    size_t hasher(const Sales_data &s)
    {
        return hash<string>()(s.bookNo);
    };

- 自定义比较函数

<!-- -->

    bool eqOp(const Sales_data &lhs, const Sales_data &rhs){
        retrurn lhs.isbn() == rhs.isbn();
    }

- 创建

<!-- -->

    #include <unordered_set>
    #include <unordered_map>

    using SD_multiset = unordered_multiset<Sales_data, decltype(hasher) *, decltype(eqOp) *>;
    SD_multiset sd_ms;

- 桶接口

<!-- -->

    Sales_data sd;
    sd_ms.bucket_count();//桶数
    sd_ms.max_bucket_count();//最大桶数
    sd_ms.bucket_size(1);//桶大小
    sd_ms.bucket(sd);//桶索引
    sd_ms.rehash(10);//改变桶数
    sd_ms.reserve(10);//改变大小

# pair

    #include <utility>

    string s;
    size_t n;
    pair<string, size_t> pr(s, n);
    pair<string, size_t> pr2 = make_pair(s, n);
    pr.first;
    pr.second;
    pr == pr2;

# ==tuple==

    #include <tuple>

    tuple<int, int, int> t1(1, 2, 3);
    auto t2 = make_tuple(1, 2, 3);
    auto t_count = tuple_size<decltype(t1)>::value;
    using t_type = tuple_element<0, decltype(t1)>::type;
    int &t_element = get<0>(t1);
    int a, b, c;
    tie(a, b, c) = t1; // 解构

# ==bitset==

- 常用于优化时间复杂度，01背包算法中

<!-- -->

    #include <bitset>

    bitset<13> bitset1(0xbeef);
    string str("11111110011");
    bitset<32> bitset2(str, 5, 4); // 1100
    bitset2.any();
    bitset2.all();
    bitset2.count();
    bitset2.set();
    bitset2.set(2);
    bitset2.reset();
    bitset2.reset(2);
    bitset2.flip();
    bitset2.flip(2);
    bitset2.test(0);
    bitset2.to_string();
    bitset2.to_ulong();
    bitset2._Find_first();

# ==正则表达式库==

- regex_match整串全部校验、regex_search部分校验搜索、regex_replace正则替换
- string使用regex、smash、ssub_match、sregex_iterator
- `const char*`使用regex、cmatch、csub_match、cregex_iterator
- 元字符匹配：

| 符号 | 含义                                                    |
|------|---------------------------------------------------------|
| `.`  | 匹配任意单个字符（换行符 `\n` 默认不匹配）              |
| `\d` | 数字 `[0-9]`；`\D` 非数字                               |
| `\w` | 字母、数字、下划线 `[a-zA-Z0-9_]`；`\W` 相反            |
| `\s` | 空白（空格、`\t`、`\r`、`\n`、`\v`、`\f`）；`\S` 非空白 |
| `[]` | 字符集，`[1345]` 匹配其中一个数字；`[0-9a-z]` 区间      |
| `^`  | 字符串开头；`$` 字符串结尾                              |

- 数量限定：

| 量词    | 作用                               |
|---------|------------------------------------|
| `*`     | 0 次或多次                         |
| `+`     | 至少 1 次                          |
| `?`     | 0 次或 1 次；紧跟`*+/`后开启非贪婪 |
| `{n}`   | 精准 n 次                          |
| `{n,}`  | n 次及以上                         |
| `{n,m}` | n～m 次                            |

- 分组：

1.  `(xxx)`：**捕获分组**，匹配内容存入 `smatch[1]、smatch[2]`
2.  `(?:xxx)`：**非捕获分组**，只分组不保存结果，不占用下标
3.  `|`：或，`a|b` 匹配 a 或者 b

- 手机号校验示例

<!-- -->

    #include <iostream>
    #include <string>
    #include <regex>

    int main()
    {
        // 1. 正则：严格纯11位中国大陆手机号整体校验
        // ^开头 $结尾，1开头，第二位3~9，后面9位数字
        std::string strPhone = "13800138000";
        std::regex regPhone("^1[3-9]\\d{9}$");
        std::smatch result;  // 核心：smatch接收匹配结果

        // regex_match：整串完全匹配
        if (std::regex_match(strPhone, result, regPhone))
        {
            std::cout << "匹配成功，手机号：" << result[0].str() << "\n";
        }
        else
        {
            std::cout << "手机号格式非法\n";
        }

        // 2. 在一段长文本里查找手机号，用 regex_search + smatch
        std::string text = "我的电话：13912345678，备用号135-9876-5432";
        // 兼容中间横杠、空格的宽松正则
        std::regex regSearch(R"(1[3-9](?:-|\s)?\d{3}(?:-|\s)?\d{4})");

        std::smatch mat;
        std::string::const_iterator start = text.cbegin();
        std::string::const_iterator end = text.cend();

        std::cout << "\n文本中提取到的手机号：\n";
        // 循环查找所有符合规则的号码
        while (std::regex_search(start, end, mat, regSearch))
        {
            std::cout << mat.str() << "\n";
            start = mat[0].second; // 跳到本次匹配末尾，继续往后搜
        }

        return 0;
    }

# ==随机数库==

- `std::mt19937`是最常用的随机数引擎（梅森旋转算法）
- 封装成全局工具

<!-- -->

    class RandomUtil
    {
    public:
        // 获取 [min, max] 整数
        static int getInt(int min, int max)
        {
            static std::mt19937 rng(std::random_device{}());
            std::uniform_int_distribution<int> dist(min, max);
            return dist(rng);
        }

        // 获取 [min, max] 浮点数
        static double getDouble(double min, double max)
        {
            static std::mt19937 rng(std::random_device{}());
            std::uniform_real_distribution<double> dist(min, max);
            return dist(rng);
        }
    };

    int main()
    {
        for (int i = 0; i < 3; ++i)
        {
            std::cout << RandomUtil::getInt(10, 20) << " ";
        }
        std::cout << "\n" << RandomUtil::getDouble(2.0, 3.0);
        return 0;
    }

# 工具函数

    __gcd(10, 5);           // 最大公约数
    __lg(10.0);         // 对数
    __builtin_popcount(10); // 二进制中1的个数
    __builtin_ffs(10);      // 二进制中最右边的1
    __builtin_clz(10);      // 二进制中最左的1
    __builtin_ctz(10);      // 二进制中最右的0
    __builtin_parity(10);   // 二进制中1的个数的奇偶性
