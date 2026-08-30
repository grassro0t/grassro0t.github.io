---
title: "高效C++-资源管理类实例"
slug: "source-class-example"
date: 2026-08-30T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["特殊类实例", "c++", "语言特性"]
categories: ["技术笔记"]
summary: "c++资源管理类实例，包括基于裸指针+引用计数、智能指针、allocator的实现。"
toc: true
comments: true
description: "高效C++-资源管理类实例"
---

- 资源类大都不能依赖合成成员
# 裸指针资源类
- 自定义实现拷贝和拷贝赋值，既可以深拷贝也可以独占内存

<!-- -->

    class HasPtr
    {
    public:
        HasPtr(const string &s = string()) : ps(new string(s)), i(0) {}
        HasPtr(const HasPtr &p) : ps(new string(*p.ps)), i(p.i) {}
        HasPtr &operator=(const HasPtr &);
        ~HasPtr() { delete ps; }
    private:
        string *ps;
        int i;
    };
    HasPtr &HasPtr::operator=(const HasPtr &rhs)
    {
        auto newp = new string(*rhs.ps);
        delete ps;
        ps = newp;
        i = rhs.i;
        return *this;
    }

# 独占内存类（unique_ptr版）

- 必须禁用拷贝和拷贝赋值，支持移动和移动赋值

<!-- -->

    #include <iostream>
    #include <string>
    #include <memory>   // std::unique_ptr

    class HasPtr
    {
    public:
        // 默认构造，unique_ptr接管新建string
        explicit HasPtr(const std::string &s = std::string())
            : ps(std::make_unique<std::string>(s)), i(0)
        {}
        // 禁用拷贝构造、拷贝赋值（独占资源不允许浅拷贝共享）
        HasPtr(const HasPtr&) = delete;
        HasPtr& operator=(const HasPtr&) = delete;

        // 移动构造：转移unique_ptr所有权，无内存拷贝
        HasPtr(HasPtr&& other) noexcept
            : ps(std::move(other.ps)), i(other.i)
        {
            other.i = 0;
        }

        // 移动赋值
        HasPtr& operator=(HasPtr&& rhs) noexcept
        {
            if (this != &rhs)
            {
                ps = std::move(rhs.ps);
                i = rhs.i;
                rhs.i = 0;
            }
            return *this;
        }

        // 析构：无需手动写，unique_ptr离开作用域自动释放堆内存
        ~HasPtr() = default;

        // 对外简易接口测试用
        std::string getStr() const
        {
            return ps ? *ps : "";
        }
        void setStr(const std::string& s)
        {
            ps = std::make_unique<std::string>(s);
        }

    private:
        // unique_ptr独占管理string堆内存，生命周期自动管控
        std::unique_ptr<std::string> ps;
        int i;
    };

# 共享内存类（shared_ptr版）

- 所有拷贝成员基本都可以使用默认的，非常方便

<!-- -->

    #include <string>
    #include <memory>

    class HasPtr
    {
    public:
        // 构造函数，shared_ptr持有一份堆上string
        explicit HasPtr(const std::string &s = std::string())
            : ps(std::make_shared<std::string>(s)), i(0)
        {}

        // shared_ptr天然支持拷贝构造、拷贝赋值，直接使用编译器默认版本即可
        HasPtr(const HasPtr&) = default;
        HasPtr& operator=(const HasPtr&) = default;

        // 移动构造、移动赋值，做所有权快速转移，可选优化
        HasPtr(HasPtr&& other) noexcept = default;
        HasPtr& operator=(HasPtr&& other) noexcept = default;

        // 析构由shared_ptr自动处理引用计数，无需手动释放
        ~HasPtr() = default;

        // 获取当前引用计数
        long use_count() const
        {
            return ps.use_count();
        }

        // 修改字符串内容，会引发写时复制吗？不会，多个对象共享同一块内存，一改全改
        void set_content(const std::string& s)
        {
            *ps = s;
        }

        std::string get_content() const
        {
            return *ps;
        }

    private:
        std::shared_ptr<std::string> ps;
        int i;
    };

# 共享内存类（引用计数版）

    #include <string>
    #include <iostream>

    class HasPtr
    {
    public:
        // 构造：新建string + 新建引用计数器，计数初始化为1
        explicit HasPtr(const std::string &s = std::string())
            : ps(new std::string(s)), use(new std::size_t(1)), i(0)
        {}

        // 拷贝构造：共享同一份资源，引用计数+1
        HasPtr(const HasPtr &rhs)
            : ps(rhs.ps), use(rhs.use), i(rhs.i)
        {
            ++*use;
        }

        // 拷贝赋值：先递增右侧计数，再递减左侧计数（自赋值安全）
        HasPtr &operator=(const HasPtr &rhs)
        {
            // 先递增右侧引用计数，防止自赋值时提前释放
            ++*rhs.use;

            // 释放左侧原有资源
            if (--*use == 0)
            {
                delete ps;
                delete use;
            }

            // 接管右侧资源
            ps = rhs.ps;
            use = rhs.use;
            i = rhs.i;

            return *this;
        }

        // 移动构造：直接转移所有权，不增减计数（源对象不再持有资源）
        HasPtr(HasPtr &&rhs) noexcept
            : ps(rhs.ps), use(rhs.use), i(rhs.i)
        {
            rhs.ps = nullptr;
            rhs.use = nullptr;
            rhs.i = 0;
        }

        // 移动赋值
        HasPtr &operator=(HasPtr &&rhs) noexcept
        {
            if (this != &rhs)
            {
                // use用于判断指针本身是否非空。移动构造 / 移动赋值后，源对象的use会被置为nullptr，这时不能解引用。
                if (use && --*use == 0)
                {
                    delete ps;
                    delete use;
                }
                // 接管右侧资源
                ps = rhs.ps;
                use = rhs.use;
                i = rhs.i;
                // 源对象置空
                rhs.ps = nullptr;
                rhs.use = nullptr;
                rhs.i = 0;
            }
            return *this;
        }

        // 析构：引用计数-1，归0则释放string和计数器
        ~HasPtr()
        {
            if (use && --*use == 0)
            {
                delete ps;
                delete use;
            }
        }

        // 当前引用计数
        std::size_t use_count() const
        {
            return use ? *use : 0;
        }

        std::string get_content() const
        {
            return ps ? *ps : "";
        }

        void set_content(const std::string &s)
        {
            if (ps) *ps = s;
        }

    private:
        std::string *ps;       // 共享的字符串数据
        std::size_t *use;      // 共享的引用计数器（堆上分配，所有拷贝对象共用）
        int i;
    };

# 容器类（allocator版）

- 最复杂，少用，除非你要自定义容器

<!-- -->

    #include <iostream>
    #include <string>
    #include <memory>   // std::allocator
    #include <utility>  // std::move
    #include <initializer_list>

    class StrVec
    {
    public:
        // ===== 构造 / 析构 =====
        StrVec() : elements(nullptr), first_free(nullptr), cap(nullptr) {}

        // 初始化列表构造
        StrVec(std::initializer_list<std::string> il)
            : elements(nullptr), first_free(nullptr), cap(nullptr)
        {
            reserve(il.size());
            for (const auto &s : il)
                push_back(s);
        }

        // 拷贝构造：深拷贝，重新分配内存并拷贝每个元素
        StrVec(const StrVec &rhs)
        {
            auto data = alloc_n_copy(rhs.begin(), rhs.end());
            elements = data.first;
            first_free = cap = data.second;
        }

        // 移动构造：直接接管指针，不分配内存
        StrVec(StrVec &&rhs) noexcept
            : elements(rhs.elements), first_free(rhs.first_free), cap(rhs.cap)
        {
            rhs.elements = rhs.first_free = rhs.cap = nullptr;
        }

        // 拷贝赋值：copy and swap 技巧，异常安全
        StrVec &operator=(const StrVec &rhs)
        {
            auto data = alloc_n_copy(rhs.begin(), rhs.end());
            free();
            elements = data.first;
            first_free = cap = data.second;
            return *this;
        }

        // 移动赋值
        StrVec &operator=(StrVec &&rhs) noexcept
        {
            if (this != &rhs)
            {
                free();
                elements = rhs.elements;
                first_free = rhs.first_free;
                cap = rhs.cap;
                rhs.elements = rhs.first_free = rhs.cap = nullptr;
            }
            return *this;
        }

        // 析构：销毁所有元素 + 释放内存
        ~StrVec() { free(); }

        // ===== 元素操作 =====
        void push_back(const std::string &s)
        {
            chk_n_alloc();
            alloc.construct(first_free++, s);
        }

        void push_back(std::string &&s)
        {
            chk_n_alloc();
            alloc.construct(first_free++, std::move(s));
        }

        // 原地构造
        template <typename... Args>
        void emplace_back(Args &&...args)
        {
            chk_n_alloc();
            alloc.construct(first_free++, std::forward<Args>(args)...);
        }

        void pop_back()
        {
            if (size() > 0)
                alloc.destroy(--first_free);
        }

        // ===== 容量管理 =====
        std::size_t size() const { return first_free - elements; }
        std::size_t capacity() const { return cap - elements; }
        bool empty() const { return size() == 0; }

        void reserve(std::size_t n)
        {
            if (n > capacity())
                reallocate(n);
        }

        void resize(std::size_t n)
        {
            if (n > size())
            {
                while (size() < n)
                    push_back("");
            }
            else if (n < size())
            {
                while (size() > n)
                    pop_back();
            }
        }

        // ===== 迭代器 / 访问 =====
        std::string *begin() const { return elements; }
        std::string *end() const { return first_free; }

        std::string &operator[](std::size_t n) { return elements[n]; }
        const std::string &operator[](std::size_t n) const { return elements[n]; }

        std::string &front() { return *elements; }
        std::string &back() { return *(first_free - 1); }

    private:
        // ===== 工具函数 =====

        // 检查容量，不够则扩容
        void chk_n_alloc()
        {
            if (size() == capacity())
                reallocate();
        }

        // 拷贝 [b, e) 范围的元素到新分配的内存，返回首指针和尾后指针
        std::pair<std::string *, std::string *> alloc_n_copy(const std::string *b, const std::string *e)
        {
            auto data = alloc.allocate(e - b);
            return {data, std::uninitialized_copy(b, e, data)};
        }

        // 销毁所有元素并释放内存
        void free()
        {
            if (!elements) return;  // allocator不能deallocate空指针

            // 逆序销毁已构造的元素
            for (auto p = first_free; p != elements; )
                alloc.destroy(--p);

            // 释放原始内存
            alloc.deallocate(elements, cap - elements);
        }

        // 重新分配内存（默认翻倍），移动构造所有元素
        void reallocate()
        {
            auto new_cap = size() ? 2 * size() : 1;
            reallocate(new_cap);
        }

        // 按指定容量重新分配
        void reallocate(std::size_t new_cap)
        {
            auto new_data = alloc.allocate(new_cap);

            // 用移动迭代器，移动构造而非拷贝构造，提升性能
            auto dest = new_data;
            auto elem = elements;
            for (std::size_t i = 0; i != size(); ++i)
                alloc.construct(dest++, std::move(*elem++));

            free();  // 释放旧内存

            elements = new_data;
            first_free = dest;
            cap = elements + new_cap;
        }

        // ===== 成员变量 =====
        std::allocator<std::string> alloc;  // 内存分配器
        std::string *elements;   // 数组首元素
        std::string *first_free; // 第一个空闲位置
        std::string *cap;        // 容量末尾（尾后指针）
    };

# 容器类（容器版）

- 底层套个容器颗秒

<!-- -->

    #include <vector>
    #include <string>
    #include <initializer_list>

    class StrVec
    {
    public:
        StrVec() = default;
        StrVec(std::initializer_list<std::string> il) : data(il) {}

        // 拷贝/移动/析构全部用默认，vector自己管
        StrVec(const StrVec&) = default;
        StrVec(StrVec&&) noexcept = default;
        StrVec& operator=(const StrVec&) = default;
        StrVec& operator=(StrVec&&) noexcept = default;
        ~StrVec() = default;

        void push_back(const std::string& s) { data.push_back(s); }
        void push_back(std::string&& s) { data.push_back(std::move(s)); }

        template <typename... Args>
        void emplace_back(Args&&... args) { data.emplace_back(std::forward<Args>(args)...); }

        void pop_back() { data.pop_back(); }

        std::size_t size() const { return data.size(); }
        std::size_t capacity() const { return data.capacity(); }
        bool empty() const { return data.empty(); }

        void reserve(std::size_t n) { data.reserve(n); }
        void resize(std::size_t n) { data.resize(n); }

        std::string* begin() { return data.data(); }
        std::string* end() { return data.data() + data.size(); }
        const std::string* begin() const { return data.data(); }
        const std::string* end() const { return data.data() + data.size(); }

        std::string& operator[](std::size_t n) { return data[n]; }
        const std::string& operator[](std::size_t n) const { return data[n]; }

        std::string& front() { return data.front(); }
        std::string& back() { return data.back(); }

    private:
        std::vector<std::string> data;
    };
