---
title: "C++工程常用工具链-编译调试测试篇"
slug: "build-test-tool"
date: 2026-08-23
draft: false   # true=草稿，构建默认忽略
tags: ["工具", "c++", "测试"]
categories: ["技术笔记"]
summary: "一些C++工程中常用的编译调试测试工具链，包括vscode、vim、gcc、clang、cmake、doctest等。"
toc: true
comments: true
description: "C++工程常用工具链-编译调试测试篇"
---
# 远程编辑器：vscode

## 常用快捷键

- copilot自动采纳：tab
- 搜索文件：command+p
- 命令面板：command+shift+p
- 全局搜索替换：command+shift+f
- 快速注释：command+/
- 快速格式化：option+shift+f
- 侧边栏打开：command+b
- 删除当前行：command+shift+k
- 上下移动当前行：shift+option+上下
- 上下复制当前行：option+上下
- 多行光标：shift+option
  \## 常用插件
- clang-format
- cline
- trae
- cmake tools
- draw.io integration
- github actions
- markdown all in one
- one dark pro
- hex editor
- vscode mindmap
  \# 本地编辑器：vim
  \## 模式切换
- 插入模式：i
- 命令模式：esc
  \## 命令模式
- 保存退出：`:wq`
- 强制退出：`:q!`
- 全文替换：`:%s/a/b/g` a换成b
- 回到行首：0
- 回到行尾：\$
- 回到开头：gg
- 回到末尾：G
- 显示行号：`:set nu`
- 跳转行数：`nG` 跳到n行
- 删除整行：`dd`
- 复制整行：`yy`
- 查找：`/xxx`，`n`下一个，`N`上一个
- 向上翻整页：`ctrl+b`
- 向下翻半页：`ctrl+d`
  \## 插入模式
- 优先打字
  \## 配置文件 \~/.vimrc

``` conf
set number          "显示行号
set relativenumber  "相对行号
set expandtab       "tab转为空格
set tabstop=4
set shiftwidth=4
set autoindent
set ignorecase
set smartcase
syntax enable       "语法高亮
```

# 编译器：gcc

- g++默认链接上 C++ 标准库，gcc不会默认链接C++标准库，一般c++都是用g++
- 静态语言在编译阶段确认类型
- 不同编译器在编译阶段有各自优化方法，比如gnu在拷贝初始化时用直接初始化替代
- `g++ [源文件] -o [输出程序名] [参数]`
  \## 静态库与动态库
- 静态库.a：链接阶段嵌入程序，更新库需要重新编译主程序，一般比较大，发布不带
- 动态库.so：运行时加载，更新库不需要重新编译主程序，一般比较小，发布必须携带
  \## 编译流程

1.  **预处理（-E）**：展开头文件、宏
2.  **编译（-S）**：生成汇编代码
3.  **汇编（-c）**：生成目标文件 `.o`，**不链接**
4.  **链接（默认开启）**：多个 `.o` 合并为可执行文件
    \## 参数

- c++标准：-std=c++11
- 开启警告：`-Wall`，`-Werror`表示警告当错误
- 优化等级：`-On`，`-O0`调试用，`-O2`开发发布用
- 开启调试：`-g`，`-g3`携带宏定义
- 头文件搜索目录：`-I[路径]`
- ==库文件搜索目录（静态动态都一样）==：`-L[路径]`
- ==链接库（静态动态都一样）==：`-lxxx`
- 定义编译期宏：`-D[宏名称]`
  \## 生成文件类型
- 可重定位目标文件（无链接）：`g++ main.cpp -c -o main.o`
- 静态库：`ar rcs libfunc.a func.o`，静态库命名规范`libxxx.a`
- 动态库：`g++ -shared -fpic xxx.cpp -o libxxx.so`，动态库命名规范`libxxx.so`，windows是dll，`-fpic`生成位置无关代码动态库一般必须加
- 链接所有：`g++ main.o func.o -o app -L. -lfunc`
- 重要提示：==Linux 默认不会搜索当前目录的 `.so`，需要把`libfunc.so`放入系统库目录`/usr/lib`==
  \## linux系统库文件
- /usr/include
- /usr/lib
- /usr/local/include
- /usr/local/lib
- /lib
  \# 编译器：clang
- 整体报错提示比gcc更友好，代码高亮
- 基础用法：`clang++ main.cpp -o main`
- 参数（大量类似gcc）：`-std`、`-I`、`-L`、`-l`、`-Wall`、`-Wextra`、`-Werror`、`-g`、`-O0`、`-O2`
- 查看目标平台：`-print-target-triple`
- 指定目标平台（交叉编译）：`--target=aarch64-linux-gnu`
- 静态语法检查：\`--analyze
  \## clang-format
- vscode插件clang-format，设置开启「保存自动格式化」
- .clang-format文件存放代码格式化规则，统一项目代码风格，放在项目根目录中，可见[代码规范：clang-format](1.5%20部署与发行*.md#代码规范：clang-format)
  \# 调试器：gdb
- - 预处理变量：函数名`__func__`、文件名`__FILE__`、当前行号`__LINE__`、编译时间`__TIME__`、编译日期`__DATE__`
- 关闭本文件调试：`#define NODEBUG`
- ==日志调试==：推荐cerr，cout有缓冲会丢失信息需要添加flush
- ==断言调试==：头文件`<cassert>`，`assert(表达式)`
  \## 常用指令
- 开启调试：`-g`，`-O0`关闭编译器优化（因为编译器优化可能打乱堆栈）
- 调试：gdb ./main
- 设置断点：`b 行号或函数名`
- 运行：`r`
- 单步跳过：`n`
- 单步进入：`s`
- 继续运行：c
- 打印变量：`p 变量名`，查看变量类型`ptype 变量名`，实时监视`watch 变量名`
- 打印调用栈：`bt`
- 退出：`q`
  \## attach
- 不用重启进程调试，直接挂载上去，挂载瞬间进程会立即暂停
- 必须用相同的用户进行attach
- 寻找pid：`ps aux | grep ./app`
- 挂载上进程：`gdb -p <pid>`
- 继续运行：`c`
- 打印调用栈：`bt`
- 打印线程：`info threads`
- 切换线程：`thread xxx`
- 打印变量：`print var`
- 设置变量：`set var x=10`
- 解除挂载：`detach`
  \# 内存泄漏调试：valgrind
- 运行很慢，不要用来压力测试
- 开启调试：`-g`，`-O0`关闭编译器优化（因为编译器优化可能打乱堆栈）
- 一行搞定：`valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./app`
- 此工具集成了6大工具但是最常用且默认的是内存泄漏检查：`valgrind --tool=memcheck ./app`
- 打印泄漏代码行和栈和所有类型泄漏：`valgrind --leak-check=full --show-leak-kinds=all ./app`
- 追踪未初始化变量来源：`--track-origins=yes`
- 泄漏信息：

1.  **definitely lost**：确定内存泄漏 → 必须修复
2.  **indirectly lost**：跟着 definitely 一起泄漏，修好前者自动解决
3.  **possibly lost**：疑似泄漏（容器自定义分配器等常出现，按需甄别）
4.  **still reachable**：程序退出前没主动 free，但指针还持有，很多场景可忽略

- 内存泄漏原因：

1.  new、malloc忘记delete、free
2.  重复释放
3.  野指针、悬挂指针
4.  数组越界
    \# 离线调试：coredump

- 开启调试：`-g`，`-O0`关闭编译器优化（因为编译器优化可能打乱堆栈）
- 终端窗口输入`ulimit -c unlimited`开启当前终端的core文件
- 运行程序直到崩溃
- 调试：`gdb ./main core.xxx`
- 其他操作：同gdb，最重要的是`bt`、`info threads`
- 排查所有线程堆栈：`thread apply all bt`
  \# ==构建工具：make==
- `make` 是工具，读取 `Makefile` 文件，自动判断哪些文件修改了，**只重新编译改动的代码**，避免全部重编
- 构建：`make`、`make xxx`
- ==清理==：`make clean`
- ==并行编译==：`make -j$(nproc)`
- ==makefile文件一般首个规则就是最终目标文件，结尾要有clean以便重编译==
  \## 工作流程
  1. 在当前目录下找名字叫"Makefile"或"makefile"的文件
  2. 读入被include的其它Makefile
  3. 初始化文件中的变量
  4. 推导隐晦规则，并分析所有规则
  5. 为所有的目标文件创建依赖关系链

6.  找文件中的第一个目标文件（target）并把这个文件作为最终的目标文件
7.  根据触发机制递归生成最终目标文件
    \## makefile

- 在目标文件不存在或依赖文件比目标文件更新时会触发对应规则命令
- 基本格式（规则）

``` conf
目标: 依赖文件列表（空格分割）
    命令（开头必须是【Tab】，空格会报错！）
```

- order-only依赖：\|左边是普通依赖，只要修改时间更新就会重新执行，\|右边是order-only依赖，只有依赖不存在时才会新建，不会重新编译
- 环境变量

1.  CFLAGS：编译时的GCC编译器参数
2.  MAKECMDGOALS：存放你所指定的终极目标的列表
3.  CC 、CXX：当前使用的编译器
4.  MAKE：当前使用的Make工具

- 自动变量

1.  \$@：当前规则的**目标文件**
2.  \$\<：第一个依赖文件
3.  \$\^：所有全部依赖文件（去重）
4.  \$?：比目标更新的依赖文件
5.  \$()：调用自定义变量

- 变量赋值

1.  递归展开：NAME = app
2.  立即展开：TARGET := app
3.  追加赋值：CXXFLAGS += -pthread

- 伪目标：`clean / all / install` 这类不是生成文件的目标，必须声明 `.PHONY`，可以作为目标也可以作为依赖
- 语句

1.  判断：`ifeq(A,B) xxx else xxx endif`、ifneq、ifdef、ifndef
2.  循环：`for i in xxx`、`do xxx done`
3.  函数：`$(function arguments)`

- 常用函数

``` conf
wildcard：展开bash的通配符
subst：str文本替换
join：str拼接
strip：去掉开头结尾空字符
findstring：查找字符串
filter：正则过滤字符串
sort：排序
word：取单词
wordlist：取单词串
dir：取目录
notdir：取文件
words：单词个数统计
```

## 工程模版

- 工程结构

``` bash
my_project/
├── include/       # 公共头文件（库的头文件放这里）
│   └── util.h
├── src/           # 库的源码（用来编译成静态/动态库）
│   └── util.cpp
├── app/           # 主程序源码（调用我们编译的库）
│   └── main.cpp
├── build/         # 中间产物：所有 .o 目标文件（自动生成，无需手动创建）
├── lib/           # 最终输出：静态库 .a、动态库 .so（自动生成）
├── bin/           # 最终输出：可执行程序（自动生成）
└── Makefile       # 本配置文件
```

- makefile

``` bash
# ==============================================
# 1. 基础编译与目录配置
# ==============================================
CXX      := g++
STD      := -std=c++17
CXXFLAGS := $(STD) -Wall -Wextra
INCFLAGS := -I./include

# 输出目录分离配置
BUILD_DIR := build
LIB_DIR   := lib
BIN_DIR   := bin

# 库命名（最终生成 libxxx.a / libxxx.so，符合 Linux 规范）
LIB_NAME    := util
STATIC_LIB  := $(LIB_DIR)/lib$(LIB_NAME).a
SHARED_LIB  := $(LIB_DIR)/lib$(LIB_NAME).so

# 可执行程序输出路径
TARGET := $(BIN_DIR)/app

# ==============================================
# 2. 源码自动搜寻与路径转换
# ==============================================
# 自动匹配目录下所有 cpp 文件
LIB_SRCS := $(wildcard src/*.cpp)
APP_SRCS := $(wildcard app/*.cpp)

# 路径替换：把 .cpp 对应到 build 目录下的 .o 文件
LIB_OBJS := $(patsubst src/%.cpp, $(BUILD_DIR)/src_%.o, $(LIB_SRCS))
APP_OBJS := $(patsubst app/%.cpp, $(BUILD_DIR)/app_%.o, $(APP_SRCS))

# ==============================================
# 3. 伪目标声明
# ==============================================
.PHONY: all clean static shared app

# 默认目标：一键编译静态库、动态库、可执行程序
all: static shared app

# 子目标：单独编译某一类产物
static: $(STATIC_LIB)
shared: $(SHARED_LIB)
app:    $(TARGET)

# ==============================================
# 4. 输出目录自动创建
# ==============================================
# Order-only 依赖：仅目录不存在时创建，目录修改不触发重编译
$(BUILD_DIR) $(LIB_DIR) $(BIN_DIR):
    mkdir -p $@

# ==============================================
# 5. 通用编译规则：.cpp → .o（中间目标文件）
# ==============================================
# 库源码编译：加 -fpic（动态库强制要求，静态库兼容）
$(BUILD_DIR)/src_%.o: src/%.cpp | $(BUILD_DIR)
    $(CXX) $(CXXFLAGS) $(INCFLAGS) -fpic -c $< -o $@

# 主程序源码编译：不需要 -fpic
$(BUILD_DIR)/app_%.o: app/%.cpp | $(BUILD_DIR)
    $(CXX) $(CXXFLAGS) $(INCFLAGS) -c $< -o $@

# ==============================================
# 6. 静态库生成
# ==============================================
$(STATIC_LIB): $(LIB_OBJS) | $(LIB_DIR)
    ar rcs $@ $^

# ==============================================
# 7. 动态库生成
# ==============================================
$(SHARED_LIB): $(LIB_OBJS) | $(LIB_DIR)
    $(CXX) -shared $^ -o $@

# ==============================================
# 8. 可执行程序生成（链接自定义库）
# ==============================================
# 示例默认链接动态库；换静态库把 $(SHARED_LIB) 换成 $(STATIC_LIB) 即可
$(TARGET): $(APP_OBJS) $(SHARED_LIB) | $(BIN_DIR)
    $(CXX) $(APP_OBJS) -L$(LIB_DIR) -l$(LIB_NAME) -Wl,-rpath,$(abspath $(LIB_DIR)) -o $@

# ==============================================
# 9. 清理规则
# ==============================================
clean:
    rm -rf $(BUILD_DIR) $(LIB_DIR) $(BIN_DIR)
```

# ==构建工具生成框架：cmake==

- CMake不是构建工具，它是跨平台的构建工具生成器
- 目标：最小执行单元，每个可执行文件、静态库、动态库都是一个独立目标
- **每个库/程序自己的 CMakeLists** 负责"目标定义 + 链接关系"，**全部用 `target_*` 命令**（作用域限定在目标，而不是全局变量），依赖关系通过 `PUBLIC`/`PRIVATE` 精确传递
- 一份 `CMakeLists.txt` 配置文件，CMake 会自动生成对应平台的原生构建脚本：Linux就是makefile、macOS就是xcode工程
- - add_subdirectory新建子目录作用域，子目录的变量父亲不可见，父目录的变量子目录可以读
- include(xxx.cmake)在当前作用域执行脚本
  \## 作用域
- **PRIVATE**：只给当前目标自己用，不传给依赖它的目标
- **PUBLIC**：自己用，也传递给所有依赖它的目标
- **INTERFACE**：自己不用，只传递给依赖它的目标（纯头文件库常用）
  \## 常用配置
- 工程基础配置

``` cmake
# 指定最低CMake版本（必须放在第一行！）
cmake_minimum_required(VERSION 3.16)

# 项目名称，可选指定语言 C / CXX
project(demo LANGUAGES C CXX)

# 设置C++标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON) # 强制要求，不回退
set(CMAKE_CXX_EXTENSIONS OFF)       # 关闭GNU扩展，标准C++
```

- 子模块引入（注意作用域）

``` cmake
# include()不创建新作用域、不新建构建目录，只是单纯把脚本内容原地粘贴执行。
include(sub/config.cmake)

# add_subdirectory()创建独立作用域，专门用来组织子模块工程。
# 加载 ./src 下的CMakeLists.txt
add_subdirectory(src)

# 手动指定子模块构建目录（默认不需要）
add_subdirectory(thirdparty build_thirdparty)

# EXCLUDE_FROM_ALL：默认编译主项目时不编译该子目标，需要手动构建
add_subdirectory(demo EXCLUDE_FROM_ALL)
```

- 变量赋值和追加

``` cmake
# 普通变量
set(NAME "value")

# 缓存变量（GUI/多次构建保留，可外部-D覆盖，FetchContent必备）
set(XXX_BUILD_TESTS OFF CACHE BOOL "disable test")

# 条件判断
if(WIN32)
elseif(UNIX)
else()
endif()

# 打印调试信息
message(STATUS "提示信息")
message(FATAL_ERROR "致命错误，终止构建")

# 列表追加
list(APPEND SRCS a.cpp b.cpp)
```

- 依赖查找（包括FetchContent）

``` cmake
# 查找系统安装库
find_package(Protobuf REQUIRED)
find_package(gRPC REQUIRED)

# FetchContent 自动拉取第三方源码
include(FetchContent)
FetchContent_Declare(...)
FetchContent_MakeAvailable(...)
```

- 条件判断

``` cmake
if(${CMAKE_CXX_STANDARD} LESS 17)
    message(FATAL_ERROR "Require C++17 or higher")
endif()
```

- 目标构建

``` cmake
# 可执行文件
add_executable(main main.cpp)

# 静态库
add_library(mylib STATIC src/a.cpp)
# 动态库
add_library(mylib SHARED src/a.cpp)
# 接口库（仅头文件库 header-only）
add_library(mylib INTERFACE)
```

- 为目标添加宏

``` cmake
# 多个宏批量写
target_compile_definitions(app PRIVATE
  DEBUG
  MAX_CONN=1024
  ENABLE_LOG=1
)
```

- 为目标添加编译配置

``` cmake
target_compile_options(target PRIVATE -Wall -O2)
```

- 为目标引入头文件

``` cmake
target_include_directories(target PRIVATE ./include)
```

- 为目标引入链接
  - 现代cmake中target_link_libraries链接的可以是一个目标（自带配套头文件目录、宏、编译参数）

  ``` cmake
  target_link_libraries(target PRIVATE protobuf::libprotobuf)
  ```

  ## 内置变量

  ### 路径变量
- `CMAKE_SOURCE_DIR`：**项目根源码目录**（CMakeLists.txt 所在目录）
- `CMAKE_CURRENT_SOURCE_DIR`：**当前正在解析的 CMakeLists.txt 所在源码目录**
- `CMAKE_BINARY_DIR`：**顶层构建目录（build 目录）**
- `CMAKE_CURRENT_BINARY_DIR`：当前 CMakeLists.txt 对应的构建输出目录
  \### 平台变量

``` cmake
WIN32      # Windows系统（MSVC / MinGW）
UNIX       # Linux / MacOS 全部为 TRUE
APPLE      # MacOS
LINUX      # Linux
ANDROID    # Android
```

### 静态库/动态库

``` cmake
# 控制默认构建静态/动态库
BUILD_SHARED_LIBS
# OFF = 默认构建 STATIC；ON 默认构建 SHARED
```

### C++标准相关

``` cmake
CMAKE_CXX_STANDARD
CMAKE_CXX_STANDARD_REQUIRED
CMAKE_CXX_EXTENSIONS
```

## FetchContent

- cmake内置模块，在 CMake 构建阶段自动下载 / 拉取第三方源码，就地编译（有可能静态库有可能动态库有可能可执行文件）
- 引入fetchcontent

``` txt
include(FetchContent)
```

- 声明三方库

``` txt
# 1. 声明第三方库
FetchContent_Declare(
  protobuf
  GIT_REPOSITORY https://github.com/protocolbuffers/protobuf.git
  GIT_TAG        v3.21.12   # 指定tag，不要用main/head，版本不稳定
  GIT_SHALLOW    TRUE        # 只拉取单个commit，加速克隆（推荐）
)
```

- 下载并编译引入三方库

``` txt
# 2. 拉取并编译引入
FetchContent_MakeAvailable(protobuf)
```

- 自动下载源码到`build/_deps/xxx-src/`，自动编译后生成目标库和可执行文件都在`build/_deps/xxx-build`
- 直接链接到目标库

``` txt
target_link_libraries(app PRIVATE spdlog::spdlog)
```

## 工程模版

- ==FetchContent==：CMake 内置模块，构建时自动 git clone 第三方库源码到构建目录
- ==GenerateExportHeader==：CMake 内置模块，自动生成 `utils_export.h`，实际使用中，头文件添加MYPROJECT_UTILS_EXPORT标记的函数和类才能在动态库符号表中找到
- 使用方法

``` bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug   # 配置(-DWARNINGS_AS_ERRORS=ON让警告直接变错误)
cmake --build build -j                    # 编译
./build/app/myapp                         # 运行
ctest --test-dir build --output-on-failure # 测试
cmake --install build --prefix "${PWD}/install" # 安装
```

- 工程结构

``` bash
cmake_test/
├── CMakeLists.txt                    # 根构建（含 CMAKE_POLICY_VERSION_MINIMUM 兼容 CMake 4.x）
├── README.md                         # 完整文档
├── .clang-format                     # 代码风格
├── .gitignore
├── cmake/
│   ├── CompilerWarnings.cmake        # 严格警告集（-Wall -Wextra -Wconversion -Wshadow...）
│   ├── Dependencies.cmake            # FetchContent：fmt 11.1.4 + doctest v2.4.11
│   └── RPath.cmake                   # @loader_path / $ORIGIN RPATH 配置
├── libs/
│   ├── core/    (STATIC)             # libmyproject_core.a
│   │   ├── include/myproject/core/   #   calculator.h, config.h
│   │   ├──src/                      #   calculator.cpp, config.cpp
│   │   └── CMakeLists.txt
│   └── utils/   (SHARED)             # libmyproject_utils.dylib
│       ├── include/myproject/utils/  #   string_utils.h, logger.h（含导出宏）
│       ├── src/                      #   string_utils.cpp, logger.cpp
│       └── CMakeLists.txt
├── app/                              # myapp 可执行文件
│   ├── main.cpp
│   └── CMakeLists.txt
└── tests/                            # 3 个 doctest 测试（CTest 注册）
    └── CMakeLists.txt
```

- [CMakeLists](../code/cmake_test/CMakeLists.txt)：控制全局设置和组装cmake子模块
- [RPath](../code/cmake_test/cmake/RPath.cmake)：开发和安装环境使用不同的库文件搜索目录（动态库）
- [CompilerWarnings](../code/cmake_test/cmake/CompilerWarnings.cmake)：警告升级为错误，支持各个编译规则调用
- [Dependencies](../code/cmake_test/cmake/Dependencies.cmake)：FetchContent 在线拉取第三方库源码，配合 `SYSTEM TRUE` 隔离第三方库警告
- libs/core/[CMakeLists](../code/cmake_test/libs/core/CMakeLists.txt)：编译静态库，源码封闭，头文件开放
- libs/utils/[CMakeLists](../code/cmake_test/libs/utils/CMakeLists.txt)：编译动态库，源码封闭，头文件开放，配合GenerateExportHeader控制对外符号表接口
- app/[CMakeLists](../code/cmake_test/app/CMakeLists.txt)：编译主程序并链接两个库
- tests/[CMakeLists](../code/cmake_test/tests/CMakeLists.txt)：负责使用doctest工具做单元测试
  \# 单元测试代码框架：doctest
  \## 使用方式

1.  测试脚本链接doctest三方库和待测程序（或者库）
2.  测试cpp脚本必须声明（多个测试源文件时，只能一个文件定义 `DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN`，其余文件只 include，不能定义宏）

``` cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN #定义这个宏后框架自动生成main
#include <doctest/doctest.h>
```

3.  编译测试脚本
4.  使用add_test将测试脚本生成的可执行文件注册到ctest中
    \## 断言系统
    \### CHECK_xxx

- 失败打印信息，**继续执行当前用例**
- 布尔

``` cpp
CHECK(cond);
CHECK_FALSE(cond);
```

- 整型比较

``` cpp
CHECK_EQ(a,b); // ==
CHECK_NE(a,b); // !=
CHECK_LT(a,b); // <
CHECK_GT(a,b); // >
CHECK_LE(a,b); // <=
CHECK_GE(a,b); // <=
```

- 浮点比较

``` cpp
// 自定义误差
CHECK(res == doctest::Approx(0.3).epsilon(1e-8));
```

- 异常检测

``` cpp
// 任意异常抛出即成功
CHECK_THROWS(expr);
// 精确匹配异常类型（你关注的 CHECK_THROWS_AS）
CHECK_THROWS_AS(expr, std::invalid_argument);
// 异常类型 + 匹配异常描述字符串
CHECK_THROWS_WITH(expr, "错误提示文本");
// 要求不抛出任何异常
CHECK_NOTHROW(expr);
```

- 错误提示

``` cpp
CHECK_MESSAGE(value > 0, "数值非法，value = " << value);
```

- 跳过测试

``` cpp
FAIL("不应该执行到此分支");
FAIL_MESSAGE("错误码：" << code);

SKIP();
SKIP_MESSAGE("当前平台不支持，跳过");
```

### REQUIRE_xxx

- 失败立刻终止当前 TEST_CASE
- 基本都有对应check版本的require版本
  \## 测试语法
- 基本测试：

``` cpp
TEST_CASE("基础加法测试") {
    int a = 2;
    int b = 3;
    CHECK(a + b == 5); //失败继续执行打印错误
    CHECK_FALSE(1 + 1 == 3);
    CHECK_MESSAGE(a > 0, "a不能小于0，当前值：" << a);
    REQUIRE(ptr != nullptr); //失败立刻终止当前测试程序
    REQUIRE_MESSAGE(file.open(), "打开文件失败");
}
```

- 分支测试（每个分支都会执行一遍后测试其他分支的TEST_CASE）

``` cpp
TEST_CASE("测试abs") {
    int x;
    SECTION("正数") {
        x = 5;
        CHECK(abs(x) == 5);
    }
    SECTION("负数") {
        x = -5;
        CHECK(abs(x) == 5);
    }
    SECTION("零") {
        x = 0;
        CHECK(abs(x) == 0);
    }
}
```

- 分组测试（也类似分支，但支持过滤执行：./test --filter="数学运算"）

``` cpp
TEST_SUITE("数学工具") {
    TEST_CASE("加法") {
        CHECK(1+2==3);
    }
    TEST_CASE("乘法") {
        CHECK(2*3==6);
    }
}
```

- 测试夹具（多个用例需要重复初始化 / 释放资源）

``` cpp
struct FixtureDemo {
    std::vector<int> vec;
    FixtureDemo(){
        vec.push_back(1); // setup，每个用例执行前调用
    }
    ~FixtureDemo(){
        // teardown，每个用例结束自动调用
    }
};

TEST_CASE_FIXTURE(FixtureDemo, "带夹具的vector测试"){
    CHECK_EQ(vec[0],1);
}
```

## 命令行调用

``` bash
# 列出所有测试用例，不执行
./test --list-test-cases

# 过滤：只运行名称/标签匹配
./test --filter="[math]"
# 排除标签
./test --filter="! [slow]"

# 详细输出
./test -s
# 关闭颜色
./test --no-colors
# windows失败后暂停
./test --pause
```

## 前后钩子

``` cpp
DOCTEST_GLOBAL_SETUP() {
    // 全部测试启动前执行一次
}
DOCTEST_GLOBAL_TEARDOWN() {
    // 全部测试跑完执行一次
}
```

## 常用宏

``` cpp
// 关闭异常支持
#define DOCTEST_CONFIG_NO_EXCEPTIONS
// 关闭耗时统计
#define DOCTEST_CONFIG_NO_TIMERS
// 缩短编译时间，关闭部分扩展功能
#define DOCTEST_CONFIG_SUPER_FAST_ASSERTS
```

## cmake集成

1.  FetchContent 在线拉取doctest源码
2.  测试脚本链接doctest三方库和待测程序（或者库）并编译
3.  开启ctest：include(CTest)或者enable_testing()
4.  使用add_test将测试脚本生成的可执行文件注册到ctest中
    \# 接口测试：curl

- 底层 libcurl 库Linux/macOS 自带，用来发请求、调试接口、测试服务
- 语法：`curl [options] URL`
  \## 常用参数
- `-X GET/POST/PUT/DELETE`：指定 HTTP 方法
- `-H "key:value"`：设置请求 Header，可多次使用
- `-d 'json字符串'`：POST 请求 body 数据；默认`Content‑Type:application/x‑www‑form‑urlencoded`
- `‑‑json '{}'`：发送 json，自动设置Content‑Type: application/json（新版 curl）
- `-o file` ：把响应输出保存到文件
- `-O`：把 url 文件名保存成本地文件
- `-v`：verbose，打印完整请求响应细节（调试神器）
- `-i`：输出响应头 + 响应体
- `-I`：只发 HEAD 请求，只拿响应头，不拿 body
- `-L`：跟随 301/302 重定向
- `-k`：跳过 HTTPS 证书校验（测试环境用，生产不要）
- `-b "k=v"`：发送 Cookie
- `-c cookie.txt`：把返回 Cookie 保存到文件
- `‑‑connect‑timeout 3`：连接超时 3 秒
- `‑‑max‑time 10`：总请求最大耗时 10 秒
- `-s`：静默模式，关闭进度条、下载统计、错误进度输出
  \## 常用指令
- get请求，看响应头和返回内容

``` bash
curl -i http://127.0.0.1:8080/hello
```

- post请求，发送json

``` bash
curl -X POST --json '{"name":"test","age":20}' http://127.0.0.1:8080/api/user
```

- post请求，表单提交

``` bash
curl -X POST -d "username=admin&password=123456" http://127.0.0.1:8080/login
```

- 自定义header携带（鉴权）

``` bash
curl -H "Authorization: Bearer eyxxx" http://127.0.0.1:8080/api/protected
```

- 发送cookie

``` bash
curl -b "sessionid=abc123" http://127.0.0.1:8080/home
```

- 保存结果

``` bash
curl -o out.json http://127.0.0.1:8080/api/list
```

- 跟随重定向

``` bash
curl -L http://127.0.0.1:8080/redirect
```

- https忽略证书校验

``` bash
curl -k https://127.0.0.1:8443/hello
```

- 跟踪完整过程

``` bash
curl -v http://127.0.0.1:8080/hello
```
