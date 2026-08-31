---
title: "计算机基础系列-计算机组成原理"
slug: "computer-organization"
date: 2026-08-31T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["计算机组成原理", "c++", "并发编程", "计算机体系结构"]
categories: ["技术笔记"]
summary: "计算机组成原理，包括一些硬件上的设计和基础设施，以及上一层的操作系统级的设计，还有一些并发编程的相关知识，Damn it，这部分内容非常庞杂，而且互相串联，需要长期的实践才能理解透彻。"
toc: true
comments: true
description: "计算机组成原理"
---

| 层级 | 名称         | 类型   | 说明                       |
|------|--------------|--------|----------------------------|
| 6    | 应用语言层   | 虚拟机 | 面向用户的应用程序         |
| 5    | 高级语言层   | 虚拟机 | 高级编程语言，编译器翻译   |
| 4    | 汇编语言层   | 虚拟机 | 汇编助记符，汇编器翻译     |
| 3    | 操作系统层   | 虚拟机 | 系统调用，软硬件过渡       |
| 2    | 机器语言层   | 实机器 | ISA 二进制指令，软硬件接口 |
| 1    | 微程序机器层 | 实机器 | CPU 内部微指令             |
| 0    | 数字逻辑层   | 实机器 | 逻辑门、硬件电路           |

# 计算机数据

- 整数、浮点数、字节、字

- 字长：
  - 地址字长：虚拟地址的大小
  - 机器字长：cpu一次能处理的大小
  - 存储字长：每个存储单元能存储的大小
  - 总线字长：总线一次传输的数据量
  - 程序字长：编译器决定

- 小端大端：小端是最低字节在低地址中，大端是低字节在高地址中，操作系统确定，linux默认小端序

- ASCII 字符，和大小端序完全无关，因为ASCII 是单字节编码，1 字节一个字符
  \## 数据类型
  \### 整型

- 尽量不使用无符号数，因为C语言会在运算中默认将有符号数转为无符号数

- 无符号数：![](../computer-organization/bbaae1373fa824afeed30d909d6292285686b2a9.png)

- 有符号数：
  - 原码：![计算机组成原理_CSAPP](../computer-organization/2dd3c2de604bf3219e0a104c7a66026f0e1c840d.png)

  - 反码：正数不变，负数符号位以外取反![计算机组成原理_CSAPP](../computer-organization/e221dd4c7fd3dbce70b3bb8c821d94141773d917.png)

  - 补码：正数不变，负数反码+1

  - <figure>
    <img
    src="../computer-organization/f640aba1507733abe516217a5ecdb15a4a4486d9.png"
    alt="计算机组成原理_CSAPP" />
    <figcaption aria-hidden="true">计算机组成原理_CSAPP</figcaption>
    </figure>

- 类型转换：先进行位扩展，再进行有符号和无符号转换

- 位扩展：
  - 无符号数：零扩展
  - 有符号数：符号扩展

- 截断：无符号数相加丢弃高位

- 溢出：有符号数相加数据错误

- 有符号转无符号：![计算机组成原理_CSAPP\|373](../computer-organization/b8d49bcbe4441e20118feff7e94a34712ae8b1d9.png)

- <figure>
  <img
  src="../computer-organization/4cd9b51104fc5dafb7cd6ac2bf667d5a6699177a.png"
  alt="计算机组成原理_CSAPP|604" />
  <figcaption aria-hidden="true">计算机组成原理_CSAPP|604</figcaption>
  </figure>

- 无符号转有符号：![计算机组成原理_CSAPP\|434](../computer-organization/4d89656c77b2d42ae1ff7caf139c432150446493.png)

- ![计算机组成原理_CSAPP\|605](../computer-organization/74fdc6a3a5eefcdf5c1678e2eff432cabfda30c1.png)
  \### 浮点型

- 二进制小数：0.xxx，小数在后面补0，整数在前面补0

- 浮点表示：![计算机组成原理_CSAPP](../computer-organization/a19970189f5b45cf6e9750250fda06f82c964467.png)
  - S：符号 1位
  - M：尾数 二进制小数小数部分 n位
  - E：阶码 无符号数，对尾数进行加权 k位

- 计算机中的浮点数：
  - S：符号
  - Frac：组成尾数的值，Frac超出范围舍入
  - Exp：组成阶码的值，EXP超出范围溢出
  - `Bias=2^(k-1)-1`
  - 规格化值：阶码不全为0和1，exp是阶码指示的值，E=exp-Bias，尾数表示1.XXXX，专门用于表示大范围的数
  - 非规格化值：阶码全为0，E=1- Bias，尾数表示0.XXXX，专门用于表示正负0以及趋近于0的数
  - 特殊值：inF和NaN
    ![计算机组成原理_CSAPP](../computer-organization/b11d826cfc43f2b0395b8717f973d11bb01da210.png)
    ![计算机组成原理_CSAPP](../computer-organization/5ad29181fff379a64c2c00d222ca2a66b7394991.png)
    ![计算机组成原理_CSAPP](../computer-organization/5db171ce9f9d76d23e8d2aac4aaa26e5e7f6fdae.png)

- 二进制转浮点数：
  1.  计算Bias
  2.  计算exp和frac
  3.  计算E和M
  4.  计算V
      ![计算机组成原理_CSAPP](../computer-organization/a5ef4440d1d5fb51ca200cf907e0fb32098ac11a.png)

- 舍入方式：
  - 向偶数舍入=四舍五入+看最后一位是否偶数，奇数进偶数舍
    ![计算机组成原理_CSAPP](../computer-organization/fa06c60de3504714ac62a5a1d88dff036b7d74c1.png)
    ![计算机组成原理_CSAPP](../computer-organization/77ac14e857a57e5a1470e8649f18dbbea43e73db.png)

- 浮点数运算：可交换，不可结合（溢出和不精确舍入），不可分配（NAN），满足单调性

- 整数浮点转换：
  - float/double转换成int：double会出现舍入和溢出
  - int或float转换为double：没问题
  - int转换为float：出现舍入
  - double转换为float：出现舍入和溢出
    \## 基础运算

- 位运算：
  - 移位运算：
    - 逻辑右移：左侧补0
    - 算数右移：左侧补符号位

- 逻辑运算：逻辑运算有提前终止的特性

- 算数运算：

- 加法：
  - 无符号：![](../computer-organization/72fbdf6dfcb4514632fa85abaf93ac762e139b6a.png)
    - 溢出：![340](../computer-organization/c9785ab8b70709ed204df6a276670c0e393f9714.png)
  - 有符号：![](../computer-organization/e817ad37e78ea5929d3799b04678bce2cd6d66aa.png)
    - 溢出：![328](../computer-organization/c03816196f0aade510087ad72288b15e608857bf.png)

- 乘法：
  - 无符号：![](../computer-organization/4ef00384da008f7673ecda78f87bf110bbe81f21.png)
  - 有符号：![](../computer-organization/b1ea3f06bfe936590c11ea4e059d043ff2e09c14.png)
    \## X86汇编

- 默认机器字长64位，汇编使用的地址也是虚拟地址

- 指令集：
  - 复杂指令集（CISC）：X86
  - 精简指令集（RISC）：ARM

- 基础设施：
  - 程序计数器（PC）
  - 寄存器（Register）
  - 状态码（CC）

- 伪指令：以.开头的行都是指导汇编器和链接器工作的伪指令

- 数据类型：
  ![计算机组成原理_CSAPP](../computer-organization/9644d2a92b3f76cd74e1c26afdd8b70a8ff66c27.png)

- 操作数
  - Imm：立即数
  - R\[\]：某个寄存器的内容
  - M\[\]：某个内存的内容
    ![计算机组成原理_CSAPP](../computer-organization/1574731ca22140093c20a5e5e51d3233f5d6c0ba.png)
    \### 整型指令

- 寄存器：
  ![计算机组成原理_CSAPP](../computer-organization/c659400dfb51079835352f4fba701d6e808de925.png)

- 数据传送指令MOV
  - 两个操作数不能同时为内存地址，必须用一个寄存器进行中转
  - 寄存器大小一定要和指令最后一个字符指定的大小匹配
  - 较小的源值复制到较大的目的，MOVZ是将目的中剩余的字节填充0，MOVS是将目的剩余的字节填充符号位的值
    ![计算机组成原理_CSAPP](../computer-organization/dde4fc1371a3f4c3b42438b1380cd3058538b99f.png)
    ![计算机组成原理_CSAPP](../computer-organization/11a5a67b0975a23c70a596337f24b52b3a68a442.png)

- 入栈出栈PUSH/POP
  ![计算机组成原理_CSAPP](../computer-organization/815739904c94b75e99951780120cb4c7569c3fa4.png)

- 算数和逻辑运算指令
  - 一元操作符中，操作数可以是寄存器也可以是内存地址。
  - 二元操作符中，第一个操作数可以是立即数、寄存器或内存地址；第二个操作数可以是寄存器或内存地址。
  - 移位操作中，第一个操作数可以是立即数或放在寄存器%cl中，第二个操作数可以是寄存器或内存位置。
    ![计算机组成原理_CSAPP](../computer-organization/8a5f18e0843868d94d19a17ab7760a5f98e0d709.png)
    ![计算机组成原理_CSAPP](../computer-organization/a1e45091163b33a78e0d121da51337086da49ab7.png)
    ![计算机组成原理_CSAPP](../computer-organization/dd82ce20365e5807bdc6c75048e11075c02767b7.png)

- 设置条件码的指令CMP/TEST
  - CMP S1, S2：用来比较S1和S2，根据S2-S1的结果来设置条件码。
  - TEST S1, S2：用来测试S1和S2，根据S1 & S2的结果来设置条件码

- 获取条件码结果的指令SET
  ![计算机组成原理_CSAPP](../computer-organization/fba22aa23b11efeb9150bac43c2bd8a41af42d59.png)

- 跳转指令JMP/JX
  - 编码方式一般是PC-relative的，也就是跳转目标地址减去跳转指令下一条指令的地址的差。
    ![计算机组成原理_CSAPP](../computer-organization/230060dee9f31b4a55a8d5aeb78a0246e580b3ce.png)

- 条件传送指令CMOV
  - 条件控制：直接跳转，分支预测
  - 条件传送：不会先判断跳转，而是先将两个分支的结果进行计算，将结果分别保存在两个寄存器中，然后再通过条件传送指令CMOV将正确结果传送到输出的寄存器中。
    ![计算机组成原理_CSAPP](../computer-organization/3f1cb43ea4ce1add8fe1f2efaadc99efa965c414.png)

- 循环指令：

- do-while循环直接跳转

- while循环

- for循环：转化为while
  ![](../computer-organization/d8ca693abc95aa1e95b9a379c7c2ce9adab99515.png)
  \### 浮点指令

- SIMD：一条指令能够操作多个数据，是对CPU指令的扩展，用来进行小数据的并行操作

- 在浮点运算中，指令被分成了标量指令（Scalar Operations）和SIMD指令，在指令中分别用s和p表示

- 寄存器：
  ![计算机组成原理_CSAPP](../computer-organization/d6948027651b570f8f17896999ea88c3e37f8302.png)

- 数据传送指令VMOV：
  ![计算机组成原理_CSAPP](../computer-organization/7c1537057a6df976e9081068aaaaac90d160a15a.png)

- 浮点转换指令：
  ![计算机组成原理_CSAPP](../computer-organization/345c2a6f54a3b41048d6f14ccfc70131c222d4b8.png)
  ![计算机组成原理_CSAPP](../computer-organization/ba22526978cb48ed2fc039c61ef11e7c74cff836.png)

``` txt
# Conversion from single to double precision
vunpcklps %xmm0, %xmm0, %xmm0 #replicate first vector element
vcvtps2pd %xmm0, %xmm0 #convert two vector elements to double
# Conversion from double to single precision
vmovddup %xmm0, %xmm0 #replicate first vector element
vcvtpd2psx %xmm0, %xmm0 #convert two vector elements to single
```

- 浮点运算指令：
  ![计算机组成原理_CSAPP](../computer-organization/6162ccb9711114013f66f0eb231f27914d173a6e.png)
  ![计算机组成原理_CSAPP](../computer-organization/7dc7d092d2f68c4a8fe373553f24621ae41e06d0.png)
- 浮点比较指令：
  ![计算机组成原理_CSAPP](../computer-organization/7277bec2c9d69d6100e795a4f946c07ef0db2d87.png)
  \### 数据结构
- 数组：参见[数组](计算机组成原理_CSAPP笔记.md#数组：)
- 结构体：数据对齐，从大到小声明
  - 结构体大小是结构体中最大对象大小的倍数
  - 任何K字节的基本对象的地址必须是K的倍数
- 联合体：用于节省内存比如二叉树节点
  ![计算机组成原理_CSAPP](../computer-organization/0110ba9d355d3f2142743583582aacfc250f1e57.png)
  \### 函数
- 关键：控制传递（能中断和恢复执行）+数据传递（传参与返回值）+内存分配（局部变量分配和回收）
- 函数栈是向下增长的，rsp的值只会越来越小
- 调用函数A，被调函数B
- rsp：永远指向**栈顶**（栈的最低地址）
- rbp：栈帧的**基址（栈的最高地址）**
- 栈帧（从高到低）：
  - A参数构造区：保存多余函数参数
  - A返回地址：调用B后的下一条指令地址
  - B被保存寄存器：旧rbp、旧的被调用者寄存器
  - B局部变量：局部变量、数组或结构、调用者寄存器
    ![计算机组成原理_CSAPP\|372](../computer-organization/4900097e8b6561b08c8d517bce2221ab38c5b991.png)
    ![计算机组成原理_CSAPP](../computer-organization/9927ba2b816d3f12186e86de2e2e892bae685195.png)
    ![计算机组成原理_CSAPP](../computer-organization/adc52df6a262af46a77ec2529936b0f67150fec6.png)
- 调用流程：

1.  如果 A 后续还要用到调用者寄存器（`rax rdi rsi rdx rcx r8 r9 r10 r11`），A 自己压栈保存
2.  前 6 个参数放到寄存器 `rdi,rsi,rdx,rcx,r8,r9`；**多于 6 个的参数压入栈**
3.  call B：把 call 的下一条指令地址压入栈，然后跳转到 B 函数入口
4.  B函数序言prologue

``` asm
push  %rbp        ; 把A的rbp压栈，保存老栈帧基址
mov   %rsp, %rbp  ; rbp = rsp，建立B自己的栈帧基址
push %rbx         ; 被调用者寄存器必须压栈
push %r12
push %r13
sub   $N, %rsp    ; rsp -= N，向下扩展栈，开辟空间存局部变量
```

5.  B函数执行：局部变量压栈，寄存器运算
6.  B函数尾声epilogue

``` asm
pop  %r13
pop  %r12
pop  %rbx
mov   %rbp, %rsp  ; rsp直接恢复到rbp，局部变量全部丢弃，回收栈空间
pop   %rbp        ; 弹出旧rbp，恢复A函数的栈帧基址
ret               ; 从栈弹出返回地址，赋值给rip，回到A函数继续执行，等于pop %rip
```

7.  回到 A，A 把之前自己压栈的调用者寄存器 pop 回来
    \### 缓冲区溢出

- 栈整体的生长方向（rsp）：向低地址，但是C 语言数组内存增长方向：向高地址
- 超出数组的边界对栈中的其他数据进行修改，一直溢出直到覆盖函数返回地址，通过返回地址跳转到攻击代码，隐患函数包括strcat、strcpy、gets等
- 防御：
  - 不使用具有缓冲溢出的函数
  - 限制可执行代码区域：栈、堆内存标记为不可执行
  - 地址空间布局随机化：找不到攻击代码入口，但nop seld空指令可以滑到对应入口
  - 栈破坏检测：在返回地址和数组间插入一个随机值Canary，返回前检查是否被修改
- ROP攻击：
  - 溢出到返回地址写多个地址，使用现有的可执行代码的片段拼凑出攻击代码，每段代码必须以ret结尾，返回返回地址接着拼接下一段
    ![计算机组成原理_CSAPP\|654](../computer-organization/21391984e39bcb9b6283efc355df517c42421754.png)
    \# 计算机体系结构（实机器）
    ![](../computer-organization/b4f88fac945abb8abe42e481a9b2918d2b15fa5f.png)
- 老式PCI架构：
  - CPU：
    - **ALU 运算器**：算术运算、逻辑运算
    - **CU 控制器**：取指令、译码、产生控制信号；处理异常、中断
    - 寄存器组：
      - 通用寄存器：运算临时数据
      - **PC 程序计数器**：下一条指令内存地址
      - **PSW 程序状态字**：条件标志、CPU 特权级（用户态 / 内核态）
    - **Cache L1/L2/L3**：CPU 高速缓存，缓解 CPU‑内存速度差距
    - MMU 内存管理单元：地址转换
  - 内存：保存内核数据、用户进程数据（包括页表）、页缓存
  - 总线：
    - 系统总线
    - 内存总线
    - IO总线
  - IO桥（北桥）：连接三类总线，有内存控制器、IO控制器
  - IO适配器/控制器：控制器是I/O设备或主板的芯片组，适配器是插在主板卡槽的卡
  - IO设备：外部设备
- 总线功能类型：
  - 数据总线：传输数据
  - 地址总线：传输地址
  - 控制总线：允许读线、允许写线、时钟同步
- 北桥南桥：
  - 北桥：IO桥，负责高速IO通信
  - 南桥：负责低速IO通信
- 现代PCIE架构：
  - 系统总线：取消
    - CPU与高速IO：PCIe 点对点链路（RC）
    - CPU与低速IO：DMI高速点对点串行链路
  - 内存总线：取消，DDR5 物理链路直连内存
  - IO总线：PCIE树形点对点链路
  - **I/O 桥（北桥）**：取消，Root Complex (RC，根复合体)，**集成在 CPU 内部**
  - PCIe Switch（南桥）：用来扩展更多 PCIe 端口
  - Endpoint（EP 终端设备）：就是挂在 PCIe 上的外设
  - PCI‑to‑PCIe 桥：兼容老式 PCI 设备
    ![](../computer-organization/99e87e8aefe9e58abc1368e89f3bac3ecc8b3eca.png)
    \## CPU
    ![](../computer-organization/6a94297500a8fa1ef03793c72ab347648a08dc98.png)
    \### 程序编译流程
- ELF文件格式：Linux 下\*\*目标文件、可执行文件、共享库、coerdump统一的文件格式
- .a文件：不是ELF 文件，是归档文件 (archive)，是把多个可重定位 ELF 目标文件 `.o` 打包压缩到一个文件里

``` txt
libxxx.a
├─ 头部归档信息
├─ file1.o （ELF可重定位目标文件）
├─ file2.o （ELF可重定位目标文件）
└─ 索引符号表(__.SYMDEF) //记录每个符号属于哪一个`.o`，加快链接查找
```

- 预处理：替换#include和#define，生成.i文件
- 编译：翻译成汇编，生成.s文件（汇编）
- 汇编：翻译成二进制，生成.o文件（可重定位）
- 链接：重定位链接生成目标，生成.exe文件
- 反汇编：二进制转汇编，`gcc:objdump -d/gdb:disassemble`
  ![](../computer-organization/134b951db606ef80982e2941dd08a071c472e5fe.png)
  \#### 可重定位目标文件（.o）
- 由各种不同的二进制代码和数据节（Section）组成，在编译时与其他可重定位目标文件合并起来，得到一个可执行目标文件
- 结构：
  - ELF头（ELF header）：包含生成该目标文件的系统的字大小和字节顺序、ELF头的大小、目标文件类型、机器类型、节头部表的文件偏移，以及节头部表中条目的大小和数目。
  - .text： 已编译程序的机器代码
  - .rodata： 只读数据，比如跳转表等等
  - .data： 保存**已初始化的全局变量和静态变量**
  - .bss： 保存**未初始化的静态变量**，以及**被初始化为0的全局变量和静态变量**，.bss不占据实际的空间，只是一个占位符，因为未初始化变量不需要占据任何实际的磁盘空间，运行时，再在内存中分配这些变量
  - ==.symtab==： 符号表，存放在程序中定义和引用的函数和变量的符号信息。
  - .rel.text：.text节的重定位信息，可执行目标文件中需要修改的指令地址
  - .rel.data： .data节的重定位信息，合并后的可执行目标文件中需要修改的指针数据的地址
  - .debug：调试符号表，其条目是程序中定义的局部变量和类型定义，程序汇总定义和引用的全局变量，以及原始的C源文件
  - .line： 原始C源程序中的行号和.text节中机器指令之间的映射
  - ==.strtab==： 字符串表，包括.symtab和.debug节中的符号表，以及节头部中的节名字
  - ==节头部表==（Section Header Table）：给出不同节的大小和位置等其他信息，决定整个文件的虚拟地址
    ![计算机组成原理_CSAPP\|424](../computer-organization/00bb7d44493e4b09b53fa7e7d2dd8bbaed16e340.png)
    \##### ELF头
    ![计算机组成原理_CSAPP\|461](../computer-organization/bf471e6312958f7c653b4353abc4b98cc5d51f59.png)
    \##### 符号表
- 符号类型：
  - 全局链接器符号：在当前可重定位目标模块中定义，并能被其他模块引用的符号。对应于非静态的函数和全局变量。
    - 强符号：非静态函数和已初始化的全局变量。
    - 弱符号：未初始化的全局变量。
  - 局部链接器符号：只在当前可重定位目标模块定义和引用的符号。对应于静态的函数和静态变量。
  - 外部链接器符号：在别的可重定位目标模块中定义，并被当前模块引用的符号。属于全局链接器符号。
- 符号结构：
  - name：保存符号的名字，是.strtab的字节偏移量
  - type：说明该符号的类型，是函数、变量还是数据节等等
  - binding：说明该符号是局部还是全局的
  - value：对于可重定位目标文件而言，是定义该符号的节到该符号的偏移量（比如函数就是在.text中，初始化的变量在.data，未初始化的变量在.bss中）；对于可执行目标文件而言，是绝对运行形式地址。
  - size：是符号的值的字节数目。（通过value和size就能获得该符号的值）
  - section：说明该符号保存在哪个节中，是节头部表中的偏移量。
    ![计算机组成原理_CSAPP](../computer-organization/7414db9d030944d6e0743b0a04aefedc8d1f5e40.png)
- 伪节（Pseudosection）：无法通过节头部表进行索引的数据节，在符号表中定义，没有section字段，只存在于重定位目标文件，在可执行目标文件中不存在
- 伪节类型：
  - ABS：不该被重定位的符号
  - UNDEF：未定义的符号，即在当前可重定位目标文件中引用，但在别的地方定义的符号
  - COMMON：表示未被分配位置的**未初始化的全局变量**。此时value给出对齐要求，size给出最小的大小。
    ![计算机组成原理_CSAPP](../computer-organization/aefc2869d5c2fc61ddaeba7fef2110c88503cff5.png)
    \##### 重定位条目
- 汇编器先用占位符来占领位置，然后对地址未知的符号产生一个重定位条目
- 代码的重定位条目会保存在.rel.text节中，已初始化数据的重定位条目会保存在rel.data.节中
- 结构：
- Offset：要修改符号引用的内存地址
- Type：重定位的类型
- Symbol：符号表的索引值，表示引用的符号，可以通过该符号获得真实的内存地址
- Addend：有符号常数，有些重定位需要使用这个参数来修改引用位置
  ![计算机组成原理_CSAPP](../computer-organization/d58750bf093bf0813fdd65cec9e4ea119cf1499b.png)
  \##### 节头部表
  ![计算机组成原理_CSAPP](../computer-organization/f5bf5d611a2a3b5791b6f71d2a2c59a636bf9bf4.png)
  \#### 共享目标文件（.so）
- 特殊类型的可重定位目标文件，可以在加载时或运行时被动态地加载进内存并链接
- 动态库的代码段（.text、.rodata）可以多进程共享，不同进程映射到各自虚拟地址，物理内存只有一份
- 动态库的数据段（.data、.bss）不能共享每个进程独立，不同进程映射到各自虚拟地址，物理内存只有一份，但写时复制
  \#### 可执行目标文件（.exe）
- 可被加载器直接复制到内存中执行
- 加载：
  - 用户态发起 execve
  - 内核读取 ELF 文件，将可执行文件中的数据段和代码段复制到对应的内存位置
  - 销毁原有虚拟地址空间
  - 映射 PT_LOAD 段到虚拟地址空间
  - 创建用户栈
  - 判断静态链接 / 动态链接，动态链接把动态链接器 `ld‑linux‑so`（本身也是 ELF 共享库）也映射到进程虚拟地址
  - 切换回用户态，运行动态链接器 ld‑linux‑so，将各个`.so`映射到进程虚拟地址空间（会应用地址空间布局随机化）
  - 跳转到用户程序入口_start
- 结构：在这里会分成代码段、数据段和不加载段
  - ELF头：多了一个程序的入口点（Entry Point）即当程序运行时要执行的第一条指令的地址
  - 段头部表（Segment Header Table）：包括页大小、虚拟地址内存段（节）、段大小，合作决定整个文件的虚拟地址
  - .init：定义了一个小函数_init，程序的初始化代码会调用
    ![计算机组成原理_CSAPP](../computer-organization/0ea254112b26a325d97d3010829cbd0659d6fb95.png)
    \##### 段头部表
    ![计算机组成原理_CSAPP](../computer-organization/4c364473fafcd5a50e923c4366a2c37303ff067f.png)
    \##### 节头部表
- Address不为0说明链接器已为所有数据节分配了虚拟地址
  ![计算机组成原理_CSAPP](../computer-organization/a11cc7b72833c1ab8340359f98700e6cc6ec3fa6.png)
  \#### 链接器
- 类型：
  - 静态链接：编译阶段，把**所有目标文件.o + 静态库.a**在编译链接时全部合并，把库代码**复制拷贝进可执行文件**
  - 动态链接：
    - 加载时（常用）：程序加载到内存的时候，操作系统动态链接器把需要的共享库映射到shared library，符号解析、重定位
    - 运行时：程序**已经跑起来之后，代码主动调用 API，手动加载共享库**，比如`dlopen() dlsym() dlclose()`
- 任务：
  - **符号解析**：确认所有符号的定位
    - 局部链接器符号解析：编译器只允许每个可重定位目标文件中每个局部链接器符号只有一个定义。
    - 全局链接器符号解析：编译器可能会碰到不在当前文件中定义的符号，则会假设该符号是在别的文件中定义的，就会在重定位表中产生该符号的条目，让链接器去解决。
    - 冲突处理：全局有冲突
      - 不允许有多个同名的强符号，如果存在，则链接器会报错
      - 如果有一个强符号和多个弱符号同名，则符号选择强符号的定义
      - 如果有多个弱符号同名，符号就随机选择一个弱符号的定义
  - **重定位**：确认所有符号和指令的地址
    - 流程：
      - 所有目标模块中相同类型的节合并成同一类型的新的聚合节
      - 将运行时内存地址赋给新的聚合节，以及got、plt对应的地址
      - 使重定位条目指向正确的运行时内存地址
        \#### 地址无关代码（动态库）
- 问题：
  - 多个程序共享动态库时，对应的动态库的虚拟地址不可能都一样没法写死
  - 但可执行目标文件中的代码段是不可写的，无法通过动态链接器对可执行目标文件的代码段进行修改
  - 虽然代码是不可修改的，但变量是数据是可以修改的
- 思想：数据段与代码段的距离总是保持不变的。所以我们可以让处于代码段的plt函数通过距离常量来访问处于数据段中对应的got中保存的地址
- 过程链接表（Procedure Linkage Table，PLT）：在可执行目标文件的代码段新建一个.plt节，然后对所有引用了共享库中的函数都在该数据节中创建一个新函数xxx@plt，然后将代码中调用地址替换成call xxx@plt
- 全局偏移量表（Global Offset Table，GOT）： 在可执行目标文件中的数据段新建了一个数据节.got，用来保存共享库函数的地址，动态链接器获取到共享库函数当前地址就更新这里
- 延迟绑定（Lazy Binding）：第一次调用共享库的函数时才调用ld‑linux‑so 解析符号，回填 GOT 表，后续直接走GOT 表
  \#### 库打桩
- 允许截胡共享库，替换成自己的代码
- 参见[库打桩（Library Interpositioning）](计算机组成原理_CSAPP笔记.md#库打桩（Library%20Interpositioning）：)
  \### ALU
- FLAG寄存器：
  ZF：零标志
  CF：进位标志
  SF：符号标志
  OF：溢出标志
- 支持带进位的加法、带借位的减法、取反、位左移右移
- 乘法：左移k位相当于乘上了2\^k，对于任意整数K，我们可以先对计算关于2幂次的展开，然后根据位向量进行移位并相加
- 除法：算数右移k位相当于除以了2\^k，为了保证向0舍入（有符号数不满足，**小于零的移位操作是远离0进行舍入的**），有符号数需要添加一个偏移量：`(x<0 ? x+(1<<k)-1 : x) >> k`
- 现代CPU直接在硬件层面设计了乘除法，可以直接给ALU除法指令
  ![650](../computer-organization/dc92b93ec4b4b31a7451b383e45cfc91948e9b0c.png)
- 1位加法器
  ![225](../computer-organization/85a81c663bf1eed7935c5afa6cc611a98d0016a8.png)
- 半加器
  ![](../computer-organization/4722e705613f1d29c3f48f0df2fab79c58121aeb.png)
  ![计算机组成原理_CSAPP\|224](../computer-organization/c0ed3b12968930fc485c012d388faedc87b0474e.webp)
- 加法器
  ![计算机组成原理_CSAPP\|669](../computer-organization/11cc77487f16a2fbb285e610ac9081553fd99db5.webp)
- 8位加法器
  ![计算机组成原理_CSAPP\|666](../computer-organization/089d83d7b2eb4442a40b33f9918c96e9ca673532.jpeg)
  \### CU
  \#### CPU控制全流程
- 取指（Fetch）：按照PC获取指令放到IR中
  1.  从指令中提取出指令指示符字节，并且确定出指令代码（**icode**）和指令功能（**ifun**）
  2.  从指令中确定两个寄存器标识符rA和rB
  3.  从指令中确定常数**ValC**
  4.  确定下一条指令的地址**valP**
- 译码（Decode）：CU硬件电路判断指令类型
  1.  从寄存器文件中读取rA和rB的值**valA**和**valB**
  2.  对于push和pop指令，译码阶段还会从寄存器文件中读取%rsp的值
- 执行（Execute）：数据地址控制总线开关，进行数据操作
  1.  ALU计算运算结果，得到结果**valE**，设置条件码的值
  2.  条件传送和跳转指令会根据ifun来确定条件码组合，确定是否跳转或传送。
  3.  计算内存引用的有效地址
  4.  移动栈指针
- 访存（Memory）：写入内存或从内存读取数据**valM**
- 写回（Write Back）：将结果写入寄存器文件中。
- 更新PC（PC Update）：将PC更新为valP，使其指向下一条指令。
  ![计算机组成原理_CSAPP\|330](../computer-organization/239c093a64f1925385f26b01d0af17a812e3f398.png)
  ![计算机组成原理_CSAPP](../computer-organization/8bc3f578ab0b1fed0ca00fe209a732b085e5123c.webp)
  ![](../computer-organization/323852f991c328e9992ca37df1a6dce9d23254b7.png)
  ![计算机组成原理_CSAPP\|665](../computer-organization/9667d96b54d84a1fef86c7ca6752144dee6caae3.png)
  ![计算机组成原理_CSAPP\|669](../computer-organization/ad19e4df25248a1c38bf7d9b0770272c760ff11d.png)
  ![计算机组成原理_CSAPP\|670](../computer-organization/958f04f936810c5d5322861d048da4e2617e4ea2.png)
  ![667](../computer-organization/62ffe2850cdd8cfd3089b63bd114c6e1f18c35b9.png)
  \#### 指令流水线
- 非流水线：
  ![计算机组成原理_CSAPP\|408](../computer-organization/6e0fe53def11031425dfa94d96fcb8a85e49918c.png)
- 流水线：
  ![计算机组成原理_CSAPP\|431](../computer-organization/62497b2b914110883314aa31a67c8ae30606a3a2.png)
- 暂停与气泡
  - 暂停（stall）：是行为/控制信号，流水线故意停下，不再向前推进
  - 气泡（bubble）：是停顿时产生的NOP伪指令，什么也不做，占流水线阶段，不产生结果
- 不一致划分：时钟周期受到最慢的组合逻辑的限制，导致时钟周期太慢
  ![计算机组成原理_CSAPP\|427](../computer-organization/e534d448afee675677d7823afac165dba4096c72.png)
- 流水线过深收益下降：过深的流水线可以加快时钟周期，但会增大延迟并扩大寄存器时延的影响，意味着更深的流水线会更依赖寄存器时延的性能
  ![计算机组成原理_CSAPP\|514](../computer-organization/4baa0fdf10b5611fa97bfe13bcedc2084b2933ad.png)
- 超标量处理器：有多个硬件，一个时钟周期可以完成多个指令
  ![计算机组成原理_CSAPP\|465](../computer-organization/e4ac70eb64bf0f699b9e43820b928d8139197237.webp)
- 多核心处理器：多个CPU核心
- 现代CPU：超标量处理器+乱序执行
  \##### 流水线冒险：
- 取指（Fetch）-译码（Decode）-执行（Execute）-访存（Memory）-写回（Write Back）-更新PC（PC Update）
- 数据冒险：下一条指令用到这条指令计算出来的结果，
  - RAW 写后读：A指令在WB阶段才写回寄存器，但B指令D阶段需要读对应寄存器
  - WAW 写后写：两条指令先后写同一个寄存器（乱序才有）
  - WAR 读后写：A指令D阶段需要读对应寄存器，B指令在WB阶段才写回寄存器（乱序才有）
  - 解决方法：
    - 旁路转发：可以把（E-\>M、M-\>WB）的结果直接旁路送给下一个指令的E，不需要等待写回
    - 流水线暂停：如果M-\>WB结果旁路送过去还是够不到，下一个指令直接暂停一个周期就可以了
    - 指令重排：重排顺序，无关指令插到中间
- 控制冒险：遇到分支、跳转、异常、系统调用，**下一条指令地址不能立刻确定**
  - A指令要到EX阶段才能知道是否跳转，但B指令F阶段就要取指令
  - 解决方法：
    - 分支预测：猜分支是否跳转、猜目标地址，按猜测地址预取指令，**预测错误：执行 Flush 冲刷流水线**，把流水线里面已经预取的错误指令全部丢弃，重新从正确地址取指令
    - 流水线暂停：下一个指令直接暂停直到知道结果
- **结构冒险**：多条指令**同一时钟周期争抢同一个硬件资源**
  - 比如只有一个缓存没有指令和数据分开缓存、或者只有一个ALU同时2个指令计算
  - 解决方法：
    - 复制硬件资源
    - 流水线暂停：等资源空闲再继续
    - 指令重排：重排顺序，无关指令插到中间
      \##### 流水线指标
- cpe：计算每个元素需要的周期数
- 延迟：完成单独一个运算所需的时钟周期总数
- 发射时间：采用流水线时，两个连续的同类型运算之间需要的最小时钟周期数
- 容量：能够执行该运算的功能单元数量
- 吞吐量：对于一个容量为C，发射时间为I的操作而言，其吞吐量为C/I
- 延迟界限：当指令存在数据相关时，指令的执行必须严格顺序执行，就会限制了处理器指令级并行的能力
- 吞吐量界限：处理器功能单元的原始计算能力，是程序性能的终极限制，需要考虑系统中的所有的功能单元
  \### 寄存器组
- 类型：
  - 通用寄存器 GPR
  - 程序计数器 PC / RIP 指令指针寄存器
  - 栈指针寄存器 SP/RSP
  - 状态寄存器（EFLAGS / RFLAGS）
  - 段寄存器（cs ds ss es fs gs）：现代 x86 开启分页后主要用于权限
  - MMU / 页表相关控制寄存器：CR3
  - 异常 / 中断相关寄存器
  - TLB、MMU 内部寄存器
  - 浮点 / 向量寄存器：浮点数、SIMD 向量运算
- 时钟寄存器（寄存器组）：指令寄存器IR、程序计数器PC、条件代码CC、程序状态Stat
  ![计算机组成原理_CSAPP](../computer-organization/21d460fc44f2cf53792814322e7aef0f1a2f31d1.png)
- 随机访问寄存器（内存）：虚拟内存、寄存器文件（通用寄存器）
- 通用寄存器：运算临时数据
  - %rsp：栈指针
  - %rbp：帧指针
  - %rdi %rsi %rdx %rcx %r8 %r9：函数参数
  - %rax：函数返回值
    ![计算机组成原理_CSAPP](../computer-organization/c659400dfb51079835352f4fba701d6e808de925.png)
- 指令寄存器（IR）：当前指令
- 指令地址寄存器（PS/%rip）:下一条指令内存地址
- **PC 程序计数器**：下一条指令内存地址
- **PSW 程序状态字**：
  - 条件标志CC/FLAG：参见[ALU](5.1%20计算机组成原理.md#ALU)
  - CPU 特权级（用户态 / 内核态）：Ring0内核态、Ring3用户态
    \### 时钟
- 时钟周期：CPU完成一次指令执行的时间
- 时钟速度：1秒内完成的指令个数
- 超频：修改时钟速度
  \### Cache L1/L2/L3
- 从性能而言，SRAM\>DRAM\>SSD\>磁盘，而从每字节造价而言，SRAM\>DRAM\>SSD\>磁盘
- 块（Block）：每一层存储器都会被划分成连续的数据对象组块，越低层块越大
- 程序的局部性：
  - 时间局部性 Temporal Locality：刚访问过的内存位置，在不久的将来很可能再次被访问
  - 空间局部性 Spatial Locality：如果访问某一个内存地址，那么它相邻附近的内存地址，很快也会被访问
- 存储器系统：
  - 寄存器：能在0个时钟周期内访问
  - 高速缓存存储器（Cache Memory）：能在4\~75个时钟周期内访问
  - 主存储器（Main Memory）：需要上百个时钟周期访问
  - 磁盘存储：数百万乃至千万级的时钟周期
    ![](../computer-organization/686a132bb4b71022daa8c8a8eae9403f0872818b.png)
    ![计算机组成原理_CSAPP](../computer-organization/bf902338d98ad9fdf387b34383c5e7f5873db442.png)
- 缓存命中（Cache Hit）：想要的数据块已经在缓存中
- 缓存不命中（Cache Miss）：
  - 冲突不命中（Conflict Miss）：即使 Cache 整体还有很多空闲位置，但是两个块争抢同一个 Cache 行，互相把对方挤出去，产生 miss
  - 强制性不命中（Compulsory Miss）或冷不命中（Cold Miss）：第一次访问这个内存块，Cache 里面从来没有这块数据
  - 容量不命中（Capacity Miss）：工作集（Working Set）大小超过缓存大小，缓存太小
  - 一致性不命中 Coherence Miss：别的 CPU 修改了这个 Cache 行对应的内存数据；当前核自己 Cache 里的副本已经过期无效
- 缓存替换：
  - 牺牲块（Victim Block）：满了以后要替换（Replacing）或驱逐（Evicting）的某个块
  - 替换策略（Replacement Policy）：
    - 随机替换策略
    - 最不常使用（LFU）策略
    - 最近最少被使用（LRU）替换策略
  - 写回策略：牺牲块写回时检查脏位，如果是脏的，被替换行会将数据写回主存，否则丢弃
- 命中率=命中数量/总数量
- 命中时间（Hit Time）： 从高速缓存传输一个字到CPU的时间
- 不命中处罚（Miss Penalty）：当缓存不命中时，要从下一层的存储结构中传输对应块到当前层中，需要额外的时间（不包含命中时间）
  ![计算机组成原理_CSAPP](../computer-organization/5e7e22aec19dd881b483d13a223a55029b3a0b21.png)
  \#### Cache结构
- 标记位和组索引位能够唯一的表示缓存行，组索引放地址中间是为了让连续的内存块尽可能分散在各个高速缓存组中
- 高速缓存(S, E, B, m)，容量C为所有块的大小之和
- 有效位和标记位中间一般有脏位，用来记录缓存中保存的副本和真实的数据是否一致
- 抖动（Thrash）：高速缓存反复地加载和驱逐相同的高速缓存块的组。
  ![计算机组成原理_CSAPP](../computer-organization/136751411252261a4cac0bada1e89bbf6fb58863.png)
- 直接映射高速缓存（Direct-mapped Cache）：E=1
  - 硬件简单；存在大量**冲突不命中**和抖动
    ![计算机组成原理_CSAPP](../computer-organization/da275651b00aec6f538b9b0b080756cae27ee863.png)
- E路组相联高速缓存（Set Associative Cache）：1\<E\<C/B，S\>1，Cache使用
  ![计算机组成原理_CSAPP](../computer-organization/8988b0e01a3ef6ac0fb23db212165d3cca60b21a.png)
- 全相联高速缓存（Full Associative Cache）：E=C/B ， S=1，不常用早期TLB使用
  - **没有冲突不命中**；但是硬件比较器多，成本高
    ![计算机组成原理_CSAPP](../computer-organization/11ab69d738ad1de08564441682038e9a75c3207a.png)
    \#### 真实的三级缓存
- 每个CPU核心有独立的L1、L2缓存，共享L3缓存，L2、L3都是通用缓存
- i-cache：只保存指令的高速缓存
- d-cache：只保存程序数据的高速缓存
- Unified Cache：即能保存指令，也能保存程序数据的高速缓存
  ![计算机组成原理_CSAPP](../computer-organization/f7c2bcee261a92646dd689d508d4e00a68018cec.png)
  ![计算机组成原理_CSAPP](../computer-organization/fedf528d183e1fa2ce2665b0846ee25bb2cb7504.png)
  \### MMU
  ![](../computer-organization/afbcfe39d9835f770c798de760261ed0bb7922ba.png)
- 页命中
  ![计算机组成原理_CSAPP](../computer-organization/e7d09e97a1bdd786784caa76c9d6f5ddc4f326a7.png)
- 页未命中
  ![计算机组成原理_CSAPP](../computer-organization/60bcc5c6767d9f01e75d9bf7756a16f7cebc9001.png)
  \### CPU架构设计
- 硬件控制语言（Hardware Control Language，HCL）：只表达硬件设计的控制部分。
- 硬件描述语言（Hardware Description Language，HDL）：可以用来描述硬件结构，是一种文本语言，类似于编程语言，包括Verilog和VHDL。
- 指令集体系结构=状态单元+指令集编码+编程规范和异常处理
  \#### 状态单元
  ![计算机组成原理_CSAPP](../computer-organization/535adbbc75a06cacebb759eaf997e5f4c117e9f0.png)
- 程序状态值：
  ![计算机组成原理_CSAPP](../computer-organization/0fbeeef33952f88a69744e48676d73e5b9341862.png)
  \#### 指令集编码
- Dest：绝对地址（8字节）
- 伪指令：
  - .pos：设置绝对地址
  - .align：字节对齐
    ![计算机组成原理_CSAPP](../computer-organization/37a1b2b51f7ac749d1b715321d957fb2889d37c7.png)
    ![计算机组成原理_CSAPP](../computer-organization/4ec9d6b8459ceeaab7388aa130b1d1469fb07ad3.png)
    ![计算机组成原理_CSAPP](../computer-organization/8c47f83a6bb1d9d29def0d629232aa289f69047a.png)
    \#### 指令集实现
    ![计算机组成原理_CSAPP](../computer-organization/b1dca8efc1f6a843faeef81856f2eca21211d1ba.png)
    ![计算机组成原理_CSAPP](../computer-organization/cd0059283405aeb4c4c80f5c3cd742ecebf05a06.png)
    ![计算机组成原理_CSAPP](../computer-organization/3b6daf6acb1f6701c731a643bf7851651c963bf3.png)
    \#### 硬件架构
- SEQ：参见[SEQ硬件结构](计算机组成原理_CSAPP笔记.md#SEQ硬件结构：)
- SEQ+：参见[SEQ+硬件结构](计算机组成原理_CSAPP笔记.md#SEQ+硬件结构：)
- PIPE-：参见[PIPE-硬件结构](计算机组成原理_CSAPP笔记.md#PIPE-硬件结构：)
- PIPE：参见[PIPE硬件结构](计算机组成原理_CSAPP笔记.md#PIPE硬件结构：)
  \### 性能分析
- cpi：执行一条指令所需的平均时钟周期
  - lp表示加载/使用冒险插入bubble的平均数，mp表示预测错误插入bubble的平均数，rp表示ret指令插入bubble的平均数
    ![计算机组成原理_CSAPP](../computer-organization/f55172c57b946147ec1b41d540d708b7cbd392a7.png)
    ![计算机组成原理_CSAPP](../computer-organization/14a54258eafc9311cf44d776cbfc3ece94dde063.png)
    ![计算机组成原理_CSAPP](../computer-organization/9ff4a8c965e1d609bd36c95e22db831f1a015c22.png)
    \## 内存
- RAM：随机读写，易失，断电丢数据，主存使用DRAM，CPU寄存器、Cache使用SRAM
  - SRAM：靠双稳态电路保存 bit；**不需要刷新**，通电数据就保持住
  - DRAM：**1 晶体管 + 电容**，用电容上面电荷保存数据；电容电荷会漏电，**必须周期性刷新（约几 ms 刷新一次），否则数据丢失**
- ROM：只读（现代可以电擦写），非易失，断电保存，BIOS使用EEPROM，SSD 固态硬盘、U盘使用NAND‑Flash
  - MROM：光刻固化内容，**完全不可改写**
  - PROM：一次性烧录，**烧完就不能擦除改写**
  - EPROM：紫外线照射芯片窗口整体全部擦除，不能单字节擦除
  - EEPROM：电信号擦除，不需要紫外线，可以按字节擦除改写
  - Flash ROM：按块（扇区）擦除，不能单字节擦除
- 内存控制器（Memory Controller）：通过addr引脚和data引脚将控制DRAM芯片数据的传入和传出
- 内部行缓冲区：内存控制器发送行地址（Row Access Strobe，RAS）到DRAM芯片时会将行复制到DRAM芯片的内部行缓冲区
- 优化：参见[随机访问存储器（Random-Access Memory，RAM）](计算机组成原理_CSAPP笔记.md#随机访问存储器（Random-Access%20Memory，RAM）)
  ![计算机组成原理_CSAPP](../computer-organization/798a88abb10a6983f68fb4416853e5ee1c083188.png)
  ![计算机组成原理_CSAPP](../computer-organization/68886e25eef0d0991f5527d4573f007dd75de041.png)
  \### 底层实现（SRAM）
- 1位锁存器
  ![](../computer-organization/fec4d719db0b9e7b889160778686359b98cbc199.png)
  ![](../computer-organization/2636927a4994e1f53adf98c8a73fa7c21afc9202.png)
- AND-OR锁存器
  - 上方的是SET输入，下方的是RESET输入
  - SET=1 RESET=0 OUTPUT=1
  - SET=1 RESET=1 OUTPUT=0
  - SET=0 RESET=1 OUTPUT=0
  - SET=0 RESET=0 OUTPUT=之前放入的内容
    ![469](../computer-organization/b8edbed9efbe06dc57e261ba4ef8d3a2cf77c480.png)
- 门锁
  - WRITE_ENABLE=0 =\> SET=0 RESET=0 DATA_OUTPUT=之前放入的内容
  - WRITE_ENABLE=1 DATA_INPUT=1 =\> SET=1 RESET=0 DATA_OUTPUT=1
  - WRITE_ENABLE=1 DATA_INPUT=0 =\> SET=0 RESET=1 DATA_OUTPUT=0
    ![](../computer-organization/e04995aa87595879769e3684dec180b4920eea93.png)
    ![](../computer-organization/8c64931bb7eee20728a27da142e4a9ed441142a4.png)
- 8位寄存器：
  - 线路数：1+8\*2=17
    ![](../computer-organization/2ff1e207568288113111f63160fc6d1980bb42f9.png)
- 256位寄存器
  - 门锁矩阵：256 = 16 \* 16
  - 线路数：1+1+1+16+16=35
  - 地址：4（行）+4（列）=8位 因此需要2个4位的多路复用器
    ![585](../computer-organization/1958cf027a95f71346c6e3742573a0654cdc3d8e.png)
    ![583](../computer-organization/e34cd5be80b1c4b1def4bb8873ab1b64cc924a72.png)
    ![373](../computer-organization/e151857629120433ef2444302d259a639b13b967.png)
- 256位8位寄存器
  ![](../computer-organization/c15a879f7412a1c1326675156b09d5030cd17959.png)
  ![](../computer-organization/660712a210e4db40eabb1676e7ca3e86d9b74da1.png)
  \## 内存控制器
  \## IO设备和控制器
  \### IO控制器
  \### 磁盘
  参见[磁盘（机械硬盘）](5.1%20计算机组成原理.md#磁盘（机械硬盘）)
  \### 固态硬盘
- 闪存翻译层（Flash Translation Layer）：将对逻辑块的请求翻译成对底层物理设备的访问。
  ![计算机组成原理_CSAPP](../computer-organization/183de447a2d79e33c3f56db280ef96b807df8ea5.png)
  \# 操作系统（虚拟机）
- 资源：每个 CPU、内存和磁盘都是系统的资源
- 操作系统：将物理（physical）资源（如处理器、内存或磁盘）转换为更通用、更强大且更易于使用的虚拟形式
- 标准库：操作系统会提供几百个系统调用（system call），让应用程序调用
  \## 虚拟化
- 历史：批处理-多道批-现代操作系统
- 机器状态：程序运行时环境参数，可以读取或更新
- 程序是静态的，进程是动态的有生命周期
  \### 虚拟化CPU
- 核心：进程+时间片轮转
  \#### 进程
- 祖宗进程：**init/systemd(PID=1)**
- 僵尸进程：子进程先结束，退出状态存放在PCB块未回收
- 孤儿进程：父进程退出，子进程还在运行，会被祖宗进程收养
- 守护进程：长期运行的程序，通常用来提供服务
- shell进程：以用户身份来运行程序的应用程序，在Linux中的默认shell叫做bash，shell的后台作业需要等待前台作业完毕
- 进程组：由一个正整数进程组ID来标识，每个进程组包含一个或多个进程，而每个进程都只属于一个进程组，默认父进程和子进程属于同一个进程组
- 系统调用：
  - fork：创建子进程，写时复制，父进程返回子进程 PID，子进程返回 0
  - wait/waitpid：父进程**回收子进程**，阻塞等待子进程结束，优先使用waitpid
  - exec：加载新程序替换当前进程，不创建新 PID
  - kill：给进程发信号
  - pipe：创建匿名管道，父子进程间通信
    \##### fork

  ``` c
  pid_t fork(void);
  ```
- 共享：
  - 物理页帧（DRAM）
  - `sighand_struct`信号处理动作表
  - 文件对象`file`
  - 内核虚拟地址空间
- 复制：
  - **task_struct 进程描述符**：全新 task_struct，pid 不同
  - `mm_struct`：拷贝一份，虚拟地址空间描述符复制
  - 所有 vm_area_struct (VMA) 全部复制一份
  - 文件描述符表 (fd 数组)
  - 寄存器上下文、setjmp 的 jmp_buf、信号阻塞掩码`blocked(sigset_t)`
  - 用户栈、堆、数据段、代码段的**页表 PTE 复制**
    \##### execve

  ``` c
  int execve(const char *pathname, char *const argv[], char *const envp[]);
  ```
- 打开可执行文件
- 销毁当前进程整个用户虚拟地址空间（vm_area_struct、旧页表）
- 文件描述符 fd不变
- 创建一批全新 VMA
- 页表全部重置
- 重置信号处理，信号阻塞掩码 blocked 保留不变
- 指令寄存器跳转到新程序入口点，开始执行新程序
  \##### waitpid
- `wait`等价`waitpid(-1, NULL, 0)`，阻塞等待任意子进程
- pid
  1.  `pid > 0`：只等待这个指定 pid 的子进程
  2.  `pid = 0`：等待**调用进程所在进程组**的任意子进程
  3.  `pid = -1`：等待**任意子进程**，等价 wait ()
  4.  `pid < -1`：取绝对值，等待该进程组内任意子进程
- options
  - `0`：阻塞调用，直到子进程状态变化
  - `WNOHANG`：非阻塞，没有子进程退出立刻返回 0，**不挂起进程**
  - `WUNTRACED`：也捕获被暂停 (stop) 的子进程
  - `WCONTINUED`：捕获 SIGCONT 恢复运行的子进程
- wstatus：输出参数，子进程退出状态
  - `WIFEXITED(wstatus)`：子进程正常 exit 退出 → true
    - `WEXITSTATUS(wstatus)`：拿到子进程 exit 返回码 (0‑255)
  - `WIFSIGNALED(wstatus)`：子进程被信号杀死
    - `WTERMSIG(wstatus)`：获取杀死进程的信号编号
  - `WIFSTOPPED(wstatus)`：子进程被信号暂停 (SIGSTOP)
    - `WSTOPSIG(wstatus)`：暂停信号
  - `WIFCONTINUED(wstatus)`：子进程收到 SIGCONT 恢复运行

  ``` c
  pid_t waitpid(pid_t pid, int *wstatus, int options);
  ```

  ##### 进程控制块PCB
- 进程控制块（PCB）：存储关于进程信息的结构
  - 进程标识符 PID
  - 程序计数器 PC（下一条要执行指令地址）、栈指针、帧指针
  - 寄存器现场
  - 进程状态
  - 内存地址空间、打开文件、优先级
- task_struct：Linux的PCB
  - **pid_t pid**：进程 ID
  - **pid_t tgid**：线程组 ID，主线程 pid，代表进程 ID
  - `struct task_struct *parent`：父进程指针
  - 链表成员：把所有进程串成双向链表
  - **进程状态 state**：运行、可中断睡眠、不可中断睡眠、停止、僵尸 ZOMBIE
  - 退出码 exit_code：僵尸进程保留，供 wait 读取
  - \*\*mm_struct \*mm\*\*：虚拟地址空间描述符；用户进程不为空；内核线程 mm=NULL
  - `struct files_struct *files`：打开的文件描述符表
  - `struct signal_struct *signal`：信号相关
  - 寄存器上下文：线程保存的寄存器集合，上下文切换时保存 / 恢复
  - 调度相关：优先级、时间片
- mm_struct虚拟地址空间结构：

``` c
struct mm_struct {
    // 1.管理所有VMA虚拟内存区域
    struct vm_area_struct *mmap;   // VMA双向链表头，按虚拟地址升序排列
    struct rb_root mm_rb;          // VMA红黑树根，快速按地址查找VMA

    // 2.页表根
    pgd_t *pgd;                    // 该进程页全局目录PGD的地址，进程切换装载到CR3寄存器

    // 3.地址空间各段边界（虚拟地址）
    unsigned long start_code, end_code;    // 代码段 .text
    unsigned long start_data, end_data;    // 数据段 .data
    unsigned long start_brk, brk;          // 堆：start_brk堆起始；brk是堆当前边界（brk()系统调用修改brk）
    unsigned long start_stack;             // 用户栈起始
    unsigned long arg_start, arg_end;      // 命令行参数argv区间
    unsigned long env_start, env_end;      // 环境变量env区间

    unsigned long mmap_base;               // mmap匿名/文件映射区起始地址

    //4.引用计数
    atomic_t mm_users;    // 用户数：多少个task(线程)正在使用这个mm
    atomic_t mm_count;    // mm_struct本身引用计数（mm_users>=mm_count）

    //5.统计
    unsigned long total_vm;    //总虚拟页数
    unsigned long rss;         //驻留物理页数量

    //锁
    struct rw_semaphore mmap_sem; //操作VMA/mmap时的读写信号量
};
```

- vm_area_struct用户虚拟地址空间结构：
  - vm_start、vm_end、vm_page_prot常被用于缺页异常判断PTE类型

  ``` c
  struct vm_area_struct {
    // 1.地址范围
    unsigned long vm_start;    // VMA起始虚拟地址【包含】
    unsigned long vm_end;      // VMA结束虚拟地址【不包含】

    // 2.组织形式：双向链表 + 红黑树（两种索引同时存在）
    struct vm_area_struct *vm_next, *vm_prev; //双向链表，按虚拟地址升序排列
    struct rb_node vm_rb;                    //红黑树节点，快速查找某虚拟地址属于哪个VMA

    // 3.归属
    struct mm_struct *vm_mm;  // 归属哪一个进程的地址空间mm_struct

    // 4.权限标志
    pgprot_t vm_page_prot;    //页表保护位，最终写到硬件PTE
    unsigned long vm_flags;   // VMA标志：VM_READ VM_WRITE VM_EXEC VM_SHARED VM_PRIVATE

    //5.后端存储（后备存储backing store）
    struct file * vm_file;    //映射的文件；匿名映射(malloc/堆)为NULL
    unsigned long vm_pgoff;   //文件映射：以页为单位的文件偏移；匿名映射为0

    //6.操作回调
    const struct vm_operations_struct *vm_ops; //缺页、页面关闭回调函数指针

    //7.匿名映射反向映射（写时复制COW关键）
    struct anon_vma *anon_vma;
  };
  ```

  <figure>
  <img
  src="../computer-organization/ca90e0e8d16fa71d01b149eee5e9effda66d5a63.png"
  alt="计算机组成原理_CSAPP" />
  <figcaption aria-hidden="true">计算机组成原理_CSAPP</figcaption>
  </figure>
- files_struct已打开的文件描述符表：
  - `fd_array`：文件指针数组，数组下标就是文件描述符 fd，数组项指向`struct file`
  - `max_fds`：fd 数组的最大容量，数组不足时动态扩容
  - `next_fd`：分配新 fd，用来查找最小可用文件描述符
  - 进程内引用计数：记录该 files_struct 被多少进程共享（fork 后父子进程共享）
- signal_struct线程组共享信号处理函数表：
  - `nr_threads`：线程组内线程总数量
  - `thread_head`：线程链表头，把本组所有线程串起来
  - `shared_pending`：**线程组共享未决信号**，发给整个进程的信号
  - `wait_chldexit`：等待子进程退出的等待队列，供`wait/waitpid`使用
  - `group_exit_code`：线程组退出码，僵尸进程保存退出状态
  - `group_exit_task`：线程组整体退出时负责处理信号的线程
  - `rlim[]`：进程资源限制（最大文件数、栈大小等）
  - `sigcnt`：引用计数
    \##### 进程地址空间（按顺序）：
- 现代64位机器由于多级页表寻址只使用48（9+9+9+9+12）位虚拟地址
- 32位：用户3GB+内核1GB；64位：用户128TB+内核128TB
- 内核空间：
  - 线性直接映射区(linear mapping)：kmalloc 从这里分配；虚拟地址和物理地址固定偏移
  - vmemmap 页描述符数组：struct page 数组，每一个物理页对应一个struct page
  - vmalloc动态映射区：vmalloc() / ioremap()；虚拟连续，物理可以不连续，mm_struct在这
  - 固定映射fixmap区：映射硬件寄存器、临时映射，编译期固定虚拟地址
  - 内核镜像本身 .text/.data/.bss：内核代码、内核全局变量
  - 内核栈：每个进程有独立内核栈，task_struct在这（有mm_struct指针）
- **命令行参数、环境变量**：栈的上方，高地址处
- **栈 stack**：局部变量、函数参数、返回地址
- **共享区 shared Libraray**：
  - 匿名映射：大内存，内存通过mmap映射到此区域
  - 文件映射：磁盘文件通过mmap映射到此区域
  - 动态链接库.so
- **堆 heap**：
  - brk：小内存，修改堆顶地址
- \*\*数据段：可读可写
  - .bss：未初始化全局、静态变量；程序加载时系统自动清零
  - .data：已初始化全局 / 静态变量
- \*\*代码段：只读可执行，多进程可共享
  - .init：main之前运行的初始化指令
  - .rodata：const常量、字面值
  - .text：可执行代码
- 地址空洞：0-0x400000
  ![](../computer-organization/b6fd43dd650dc30a7f1f69f77dd51e186c288b79.png)
  \##### 进程状态转移：
- 新建：程序运行或`fork()`（内核态）
  - 分配PID
  - 创建初始化PCB
  - 复制虚拟地址空间（写时复制）
  - 复制文件描述符表
  - 复制进程上下文
  - 设置子进程返回值
- 就绪：PCB挂入就绪队列
- 运行：执行进程指令
  - 子进程调用exec：替换程序继续运行
  - 时间片用完/高优先级抢占：-\>就绪
  - 等待IO、sleep、等待资源：-\>阻塞
  - 调用exit、崩溃、收到kill终止信号：-\>终止
- 阻塞：等待某事件完成（读磁盘、管道读、sleep等）
  - 终止：进程执行结束，用户代码不再运行
  - PCB还在：变成僵尸进程，退出状态保存在PCB
  - wait回收：进程消失
  - 父进程终止：变成孤儿进程，被init/systemd收养

  ```
  stateDiagram-v2
    New : 新建
    Ready : 就绪
    Running : 运行
    Blocked : 阻塞
    Terminated : 终止

    New --> Ready : 收容 admit
    Ready --> Running : 调度
    Running --> Ready : 时间片到 / 抢占
    Running --> Blocked : 等待事件
    Blocked --> Ready : 事件完成
    Running --> Terminated : 正常结束 / 异常终止
  ```

  #### 受限直接执行（LDE）
- 不受限直接执行的危害：用户进程直接使用CPU，但是容易执行危险指令破坏系统，如果进程死循环不调用系统调用那么进程无法进行状态转移直接卡死
- 陷入（系统调用）：会将程序计数器、标志和寄存器推送到内核栈上
- 硬件基础：（cpu内部有专门模式位控制）
  - 用户态：禁止特权指令，IO操作、进程创建、内存映射全部没法干
  - 内核态：操作系统内核，全部指令都可以
- LDE主动执行：
  - 内核开机设置好陷阱表trap table，对应内核处理函数
  - 用户态使用系统调用（read/fork/open）触发trap命令
  - 硬件执行上下文切换，切换到内核态
  - 根据陷阱表跳转内核处理函数，内核完成后调用return-from-trap命令返回
  - 硬件执行上下文切换，切换到用户态
    ![](../computer-organization/1c30d3391f788caade5b3ff9714f989a13a4715e.png)
- LDE被动执行：
  - 内核开机设置好陷阱表trap table，对应内核处理函数，还设置硬件定时器，每过十几ms触发时钟中断
  - 时钟中断触发，硬件执行上下文切换，切换到内核态
  - 按照时钟陷阱进入陷阱表跳转内核处理函数，进行进程切换，完成后调用return-from-trap命令返回
  - 硬件执行上下文切换，切换到用户态
    ![](../computer-organization/941bd8d3ddfd527c78e72ca9711f798fee6aec22.png)
- LDE完整协议：
  - **内核态准备**：创建 PCB，加载程序，**设置陷阱表、启动硬件定时器**
  - `return‑from‑trap`：硬件把 CPU 切**用户模式**，进程直接在 CPU 运行
  - 运行中两种情况：
    - 进程要干系统操作：执行 trap → 进内核处理 → 返回用户态继续跑
    - 时间片到期 / 异常：定时器中断 / 异常，硬件强制切内核，OS 调度切换进程
  - 进程 exit 终止，内核回收资源
- 内核态-用户态切换（模式切换）：
  - 内核栈与用户栈：
    - 用户栈：用户级局部变量、函数参数、返回地址，每个进程有独立用户栈；线程也各自拥有用户栈
    - 内核栈：在内核虚拟地址，每个进程有独立内核栈；线程也各自拥有内核栈
  - 切换流程：
    - 接收到trap指令
    - 保存当前用户部分硬件上下文到当前线程**内核栈**
    - CPU标志位切换为内核态，栈切换为内核栈（通过修改栈指针）
    - 查找陷阱表，跳转系统调用入口，在内核栈上跑内核代码
- 进程切换（内核态）：
  - 进程上下文不小，切换开销巨大，不要频繁切换
  - 进程上下文：
    - 硬件上下文（PCB/task_struct）：寄存器、程序计数器PC、栈指针RSP、状态寄存器（标志位）
    - 内核上下文（PCB/task_struct）：PID、状态、优先级、信号掩码、已打开文件表、内存映射、==页表基址 CR3==
    - 用户上下文：用户虚拟地址空间（通过CR3切换）
  - 触发时机：
    - 时钟中断
    - 进程阻塞：系统调用sleep/wait主动放弃CPU、IO阻塞
    - 信号、异常
  - 切换流程：
    - 硬件触发，A从用户态切换为内核态
    - 内核把A的完整硬件和内核上下文全部保存到A的PCB中
    - 调度器运行，选择下一个进程B
    - 从B的PCB将所有内容恢复到当前硬件和内核上下文，载入CR3寄存器，修改虚拟地址映射切换用户上下文（TLB刷新）
    - 硬件指令，B从内核态切换为用户态
      \#### 进程调度
- 从**就绪队列**挑选进程运行
- 发生调度的情况：
  - 时钟中断
  - 进程阻塞：系统调用sleep/wait主动放弃CPU、IO阻塞
  - 进程终止
  - 阻塞唤醒
- 调度模型（单处理器）：
  - 非抢占：等进程主动放弃
    - FIFO 先来先服务：按到达顺序排队；先来先跑
    - SJF 短作业优先：优先选运行时间最短的进程，长任务饥饿
    - 优先级调度：调度器总是选**就绪队列中优先级最高**进程运行，==饥饿==
  - 抢占：时间片轮转
    - RR 时间片轮转：每个就绪进程分配相同**时间片**，时间片耗尽，放回就绪队列尾部，调度下一个
    - MLFQ 多级反馈队列：按优先级选取进程轮换，新进程进入最高优先级队列，用完时间片优先级下降，时间片内释放CPU则优先级不变，==老化==（经过一段时间S巫毒常量后所有工作重新加入最高优先级队列）
    - PS 比例份额算法：进程按比例获得一定份额的彩票，系统定时抽奖，抽中彩票的那个进程运行
- 调度指标：
  - 周转时间=完成时间-到达时间
  - 响应时间=启动时间-到达时间
  - 吞吐量：单位时间完成多少进程
  - CPU利用率：CPU 忙的时间占比
- 多处理器调度
  - **==CPU 缓存亲和性==**：进程在 CPU‑A 运行，数据在 A 的缓存；如果迁移到 CPU‑B，缓存全部失效，性能暴跌
  - SQMS 单队列多处理器调度：所有工作放入一个单独队列，每个CPU都从这个队列取工作执行，会有锁竞争
  - MQMS 多队列多处理器调度：不同CPU有不同的调度队列，工作根据不同策略放入不同队列，支持==工作迁移==（防止负载不均），会缓存不亲和
- Linux的进程调度策略
  - 一般任务SCHED_OTHER，实时任务SCHED_RR，后台计算SCHED_BATCH
  - 优先级两套体系：一般进程nice值，实时进程rt_priority，实时进程优先级全部高于一般进程
  - SCHED_OTHER：CFS完全公平调度
    - `vruntime`记录进程已经获得 CPU 的**加权虚拟时间**，越低，优先级越高，新进程初始值不是0而是当前最小值
    - nice值（-20～+19）决定权重，nice 越小，权重越大，vruntime 涨得慢
    - CFS 红黑树：用红黑树组织就绪队列，永远选 vruntime 最小的任务上 CPU 运行
    - target_latency：一个调度周期，队列里所有就绪任务尽量轮流跑一遍的总时间，每个任务理论分得时间 = `target_latency × (本任务权重 / 总权重)`，任务非常多时间片就会很小哦
    - 多核行为：每个CPU有自己的CFS 红黑树，只调度自己队列中的任务，定期检查负载，负载差距大进行工作迁移，vruntime校正保证跨核公平
  - SCHED_FIFO：SCHED_FIFO 任务就绪，立刻抢占所有普通 CFS 进程
  - SCHED_RR：优先级高于普通 CFS 进程，和 FIFO 几乎一样，**增加固定时间片**
  - SCHED_BATCH：基于CFS，nice值自动变高
  - SCHED_IDLE：基于CFS，只有非常空闲才运行
    \### 虚拟化内存
- 核心：内存映射（虚拟地址-\>物理地址）
  ![计算机组成原理_CSAPP](../computer-organization/e13570a920d2329e53b6f8fadd061933e8948ef6.png)
- 地址空间：
  - 物理地址空间（Physical Address Space）：硬件真实地址，对应DRAM物理页帧
  - 虚拟地址空间（Virtual Address Space）：逻辑地址，mm_struct管理
    - 用户虚拟地址空间：vm_area_struct管理
    - 内核虚拟地址空间：所有进程共享同一套内核虚拟地址布局
- 内存管理单元（Memory Management Unit，MMU）：CPU上的用于地址翻译的硬件设备
- 优点：
  - 为每个进程提供一致的虚拟地址空间
  - 进程空间隔离
  - 简化链接
  - 实时加载，效率很高
  - 简化共享
  - 简化动态内存分配
    \#### 虚拟地址空间
- 系统调用：
  - brk：修改堆边界
  - mmap：建立虚拟内存区域并映射（可以关联或不关联文件）
- api接口：
  - malloc：申请堆动态内存，惰性分配只有真正使用才分配物理页
    - 用户态缓存：空闲链表查找有没有可用空闲块有就直接用
    - 小内存：brk/sbrk申请
    - 大内存：mmap申请
  - free：
    - 小内存：标记为空闲块，缓存起来（空闲链表之类的数据结构），除非堆顶一大片连续空闲，才申请sbrk收缩
    - 大内存：munmap申请，直接回收
  - sbrk：封装brk系统调用，堆大小控制
  - calloc、realloc等：都是基于malloc实现的一些堆分配功能
    ![219](../computer-organization/e34d9b72cfdd1eee04bfd14d2f9ef084f405bc24.png)
    \#### 地址转换
- 就是把虚拟地址映射到对应物理地址，页表也在内存上，先找页表再找真实物理地址
- 重定位：程序运行时需要把代码里的逻辑地址修正成真实内存地址
  - 静态重定位：程序装入时将内部逻辑地址转成偏移量附加在程序运行的物理基址上，运行时地址不变，所以必须占用连续物理内存且不能移动
  - 动态重定位（依赖MMU）：装入时进入虚拟地址，具体地址转换完全由MMU、基址寄存器、界限寄存器确定
    \##### 分块（malloc等）：
- 用户请求N字节的内存，实际上是寻找N+head_size大小的空闲块，malloc返回指针指向有效载荷的开头
- 序言块（Prologue Block）：堆的起始位置有一个8字节的头部和8字节的脚部的已分配块。
- 碎片
  - 内部碎片：分配给进程的内存块，比程序实际申请的要大；块内部多出的空间浪费，分页、malloc会有
  - 外部碎片：内存里有大量空闲总字节，但是空闲内存是一堆**不连续的小碎块**，分段、malloc会有
- 空闲块分割与合并：
  - 分割：当空闲块比用户申请的尺寸更大，把一块大空闲块切为两块
  - 合并：释放内存时，如果**物理地址相邻的前后 chunk 也是空闲**，把多个相邻空闲块合并成一个大空闲块
    - 立即合并：释放时就合并
    - 延迟合并：使用时才合并
- 空闲链表结构：
  - 分离空闲链表 Segregated Free List（small bins /large bins）：把空闲块，按照块大小分成多条独立链表。相同 / 相近大小的空闲块挂在同一条链表
  - 伙伴系统 Buddy System（分页）：内存块大小永远是 2 的幂次方个页，按照块大小分成多条独立链表，内存对半切分成两块这两块互为**伙伴**，如果没有就找更大的链表空闲页切一半，剩下一半的放回对应链表，只有==互为伙伴才允许合并==
- 块入链策略：
  - 后进先出（LIFO）策略：将释放的已分配块放到空闲链表开始的地方，速度快
  - 地址顺序策略：释放一个块需要遍历空闲链表，保证链表中每个空闲块的地址都小于它后继的地址。内存利用率高
- 块出链策略：
  - 首次适应 First‑Fit：链表头部开始遍历，**找到第一个能够满足 size 的空闲块**就分配
  - 循环首次适应 Next‑Fit（循环适配）：**记住上一次分配结束的位置，从该位置向后开始查找**，链表循环
  - 最佳适应 Best‑Fit：挑选最小的、能满足 size 的空闲块
  - 最坏适应 Worst‑Fit：**挑选最大的那块空闲块**进行分配
- 块结构：双向链表
  - prev_size：保存**前一个 chunk 的大小**
  - size = 当前整个 chunk 总字节数
  - 标志位：
    - **P bit(bit0)**：PREV_INUSE，前一块是否被占用。1 = 占用；0 = 空闲。
    - **M bit(bit1)**：IS_MMAPPED，1 代表这块是 mmap 大块，不走 heap。
    - **A bit(bit2)**：NON_MAIN_ARENA，是否非主 arena。
  - `fd` (forward)：**本 bin 链表的下一个空闲 chunk 指针**
  - `bk` (backward)：**本 bin 链表的上一个空闲 chunk 指针**
  - fd_nextsize：链表的下一个空闲 chunk大小
  - bk_nextsize：链表的上一个空闲 chunk大小
    ![](../computer-organization/a166a340a0a18ea257ff9594f9fb33107f267a97.png)
- malloc空闲链表（bins）：
  - fastbin 快速空闲链表（\<64B）：单向链表，只有 fd 指针，不支持分割合并
  - unsorted bin 未排序 bin（free 释放的**非 fastbin 块**）：双向循环链表，不按大小排序，支持分割合并
  - small bins 小块链表（\<512B）：多条双向循环链表，相同大小的 chunk 同一条链表，不支持分割支持合并
  - large bins 大块链表（\>512B）：类似small bins，空闲块更大，支持分割合并
  - Top chunk：找不到合适空闲块，只有它可以收缩归还内核，支持分割不支持合并
- 垃圾收集器：由分配器检测哪些块已不被应用程序使用，就自动释放这些块
  - 垃圾收集器将内存视为一个有向可达图，不可达的就是垃圾，Mark&Sweep算法建立在malloc包的基础上，使得C和C++就有垃圾收集的能力，但是垃圾收集能力不强
    \##### 分段：
- 将虚拟地址分段，每个进程维护一张段表（代码、堆、栈），段表包括段基址、段长、权限位
- 程序逻辑地址=段号+段内偏移
- 段号查表获得基址，偏移不能超过段长否则==越界异常==
- 物理地址=段基址+段内偏移
  \##### 分页：
- 将空间分割成固定长度的分片，虚拟空间和物理空间都按一样长度分页
  \###### 页表
- 页：固定大小，常见4K
- MMU 依靠**页表**将虚拟地址VA翻译成**物理地址PA**，每个进程维护一张页表（==页表基址寄存器CR3==），切换进程时切换CR3相当于把整个虚拟地址映射全部给换了
- ==用户每个进程一个独立页表，内核所有进程共享一套内核页表==
- VP（虚拟地址）=VPN（虚拟页号）+VPO（页内偏移）
- PP（物理地址）=PPN（物理页号）+PPO（页内偏移）
- VPO = PPO， VPN = 页表项序号
- 虚拟页状态：
  - 已缓存：已保存在物理页中的已分配的虚拟页，P=1，PFN=内存地址
  - 未缓存：还未保存在物理页中的已分配的虚拟页，P=0，PFN=磁盘地址
  - 未分配：没有任何数据和它关联，P=0，PFN=NULL
- 访问虚拟页行为：
  - 已缓存：页命中
  - 未缓存：缺页异常
    - 内存分配物理页帧
    - 磁盘这一页读入物理内存，触发置换策略
    - 修改PTE，有效位置 1，填入 PPN
    - iretq 返回，**重新执行故障指令**
  - 未分配：缺页异常
    - 内核判断：该虚拟页**从未分配磁盘存储**，属于非法访问
    - 直接向进程发送`SIGSEGV`段错误信号，进程终止，**不会做页面加载**
- `malloc/mmap`之后，虚拟页由**未分配 → 未缓存**
- 页表项（PTE）：序号表示虚拟页号
  - **bit63 NX(XD) No‑Execute**：禁止执行位。置 1 页面禁止执行指令，防御代码注入，32 位没有该位。
  - **bit62‑52 Reserved 保留位**：硬件保留，必须填 0。
  - **bit51‑12 ==PFN== 物理页号**：物理页编号；物理页基址 = `PFN << 12`，4KB 页低 12 位是页内偏移，不存储在 PTE。
  - **bit11‑9 PAT**：页面属性表，控制 CPU 缓存访问类型。
  - **bit8 G Global 全局页**：全局页标记；进程切换刷新 TLB 时，该 TLB 缓存项不会被清空，多用于内核共享页面。
  - **bit7 D Dirty 脏位**：硬件自动置 1；页面发生写操作置位。页面换出 swap 时，D=1 代表需要回写磁盘，操作系统可软件清零。
  - **bit6 A Accessed 访问位**：硬件自动置 1；页面被读 / 写访问后置位。页面置换算法用来判断页面活跃度，操作系统软件清零。
  - **bit5 PCD**：Cache‑Disable，关闭 CPU 缓存，MMIO 设备内存使用。
  - **bit4 PWT**：Page‑Write‑Through，写透缓存模式。
  - **bit3 U/S User/Supervisor**：用户 / 内核权限。1 = 用户态可访问；0 = 仅内核态访问。
  - **bit2 R/W Read/Write**：读写权限。1 可读可写；0 只读。
  - **bit0 ==P== Present 存在位**：核心标志。1 虚拟页映射有效；0 访问触发缺页异常。

  ``` txt
  bit63           bit51‑12                 bit11‑0
  ┌────┬──────────────────────┬─────────────────────┐
  │NX  │     PFN物理页框号      │ 标志位(PAT G D A …P) │
  └────┴──────────────────────┴─────────────────────┘
  ```
- 分段+分页：段页表（段基址+页数），每个段进行分页，虚拟地址=段号+VPN+VPO，段基址对应页表基址地址，VPN查PPN，缺点是仍然基于分段，还是有外部碎片
- 反向页表：按物理页号 PFN 做索引，整机只有一张全局表，不按进程分表，问题在如果TLB没命中需要整张反向页表搜索匹配，速度超慢，有些算法用哈希或散列表加速查找
  \#### 翻译后备缓冲器TLB
- TLB现代都是组相连缓存
- CPU 内部高速缓存，缓存最近用过的页表项
- ==进程切换时，需要刷新 TLB==，防止旧进程页表缓存干扰
- 缓存替换策略：
  - LRU：最久远的
  - Random：随机，比LRU要好
    ![计算机组成原理_CSAPP](../computer-organization/a2c1bb147b57e813b8d50665783917050f03eaff.png)
    ![计算机组成原理_CSAPP](../computer-organization/2482c1f9b28cb883510d04d503a02de7d5a30feb.png)
- 页命中
  ![计算机组成原理_CSAPP](../computer-organization/41bf0320e8c0426f719baa3b5ab2aa17248ffd2d.png)
- 页不命中
  ![计算机组成原理_CSAPP](../computer-organization/9abc73d80cc8be9010d3a5f36ebddda0f68e55e4.png)
  \#### 多级页表
- 优势：页表过大，其中很多空置的时候可以压缩页表空间，但是寻址时间会延长
- PTE增加有效位：1表示有子页表，0表示无子页表
- CR3：最顶层页表的真实物理地址
- VPN被切成4级：PML4、PDPT、PD、PT
- 下一级物理基址=本级物理基址+本级偏移量，如果中途出现任何一个PTE是空（物理基址无效）说明这块虚拟内存没分配触发缺页异常
  ![](../computer-organization/dce6e220c22426b8d8d14bf825b8c5e694671e49.png)
- 真实多级页表项：
  ![计算机组成原理_CSAPP](../computer-organization/5501d9bc7b7395b8988d1687f7f7573dcbd19a90.png)
  ![计算机组成原理_CSAPP](../computer-organization/96e065c32666e00e9ef05cb9986bd64c7f23f241.png)
  \#### 页错误异常
- 页表检测异常触发#PF页错误异常，CPU 压入栈错误码，交由 OS 异常处理函数
- 缺页异常：Present=0
  - 未映射（非法虚拟地址）
  - 未分配（没分配物理页）
  - 已换出（被内存交换到磁盘）
  - mmap文件映射缺页（没分配物理页）
- 权限违规：
  - 写只读
  - 用户态访问内核页
  - nx禁止执行
- 缺页全流程：
  - 硬件翻译：MMU 翻译虚拟地址，PTE 有效位为 0；CPU 触发 Fault 故障；把出错指令地址压入内核栈；跳转到内核缺页异常处理入口
  - 内核缺页处理程序启动：查找 VMA（`vm_area_struct`），判断该虚拟地址是否属于进程合法 VMA 区间
  - 分支判断：
    - 未分配：给进程发送`SIGSEGV`段错误信号，进程终止
    - 未缓存：空闲物理页帧或页面置换获得物理页，磁盘上的页面数据读入物理页，修改页表项 PTE，TLB 刷新，`iretq`回到用户态
      \#### 内存交换
- 物理内存不足时，把内存中暂时不用的页换出到磁盘
- 写时复制：
  - 修改时触发保护故障，异常处理程序在物理内存中制造一个副本，然后让进程的PTE指向新的副本，处理程序返回从原指令运行
- 内存映射
  - 系统调用mmap将磁盘文件或swap映射到vm_area_struct
  - 类型：
    - 文件映射（file‑backed mapping）：合法打开文件
    - 匿名映射 anonymous mapping：Swap 交换分区
  - 共享：
    - 私有映射MAP_PRIVATE：读共享，写写时复制
    - 共享映射MAP_SHARED：读写共享
  - 流程：
    - 用户调用`mmap()`进内核
    - 内核创建`vm_area_struct(VMA)`
    - vm_area_struct设置这段虚拟空间参数
    - **不会分配物理内存，不会读磁盘**
    - 回到用户态
    - 用户访问这片虚拟地址 → **触发缺页异常**
- 交换空间：
  - 以页大小与交换空间交换
  - PTE Present (P) 位判断页在内存还是磁盘
- 页交换策略：
  - 检测：
    - 内核发现少于LW低水位个页可用时，页守护进程运行，开始根据牺牲策略释放页，直到到达HW高水位停止。
    - 如果进程发现没有空闲页，也会通知页守护进程进行页释放，自身挂起等待。
  - 策略：
    - FIFO 先进先出：按页面装入内存的时间排序，最先进入的页面最先换出
    - LRU 最近最少使用：淘汰**距离现在最久没有被访问**的页
    - LFU 最不经常使用：淘汰访问次数最少的页
    - Clock 时钟算法（使用Accessed 访问位）：
      - 把所有物理页排成循环链表，有一个时钟指针
      - 指针扫描页面
        - 如果 A=1：刚刚被访问，把 A 清零，指针下移
        - 如果 A=0：很久没访问，选中此页，进行换出
      - 指针循环转圈
- ==抖动==：系统大部分时间都在做页面换入换出，几乎没有时间执行用户程序，磁盘狂转，CPU 利用率低，页面集合（工作集）大于分配给它的物理内存，根本解决方法是扩大物理内存
  \#### 地址翻译全流程（MMU+TLB+多级页表+L1Cache）
- 高速缓存的CI+CO=VPO=PPO，导致VPO可先直达高速缓存选择好对应的缓存组，加速地址翻译过程。
  ![计算机组成原理_CSAPP](../computer-organization/e942510ab7552a4169516bcd5b06ceb93fba4f9a.png)
  \## 并发
- 并发：**多个任务交替轮流执行**，宏观看起来同时跑
- 并行：多个任务真正同时一起执行
- 饥饿：线程一直等待永远无法推进
- 共享变量：全局变量和局部静态变量是共享的
- 竞争：线程执行时序不同结果不同
- 临界区：导致竞争的代码，一般是访问共享资源的代码，临界区同一时刻只允许一个线程执行
  ![计算机组成原理_CSAPP\|420](../computer-organization/4c401904c864be985c1a459f86339ebf91b388b3.png)
- 互斥：解决竞争，保证临界区代码多线程不同时执行
- 同步：解决临界区代码多线程执行顺序
- 并发实现：
  - 多进程：进程间通信、多内核，多进程，内核调度
  - 多线程：线程间通信，单进程，内核调度
  - 事件：IO多路复用，单进程，用户调度
    \### 线程
- 进程是资源的容器（地址空间、文件描述符、信号、内存、页表），线程是执行流，CPU 调度的最小单位，一个进程至少拥有 1 个主线程
- ==线程之间不是父子关系而是对等关系==
- ==内核栈==：与task_struct对应，所以每个进程或线程都有一个独立的内核栈，才能顺利实现模式切换和上下文切换
- 线程模型：M个内核线程和N个用户线程，常用的线程库LinuxThreads和NPTL都是M:N=1:1
- 线程类型：
  - 可结合：线程终止时，它的内存资源不会释放，需要被其他线程回收
  - 分离的：线程会在它自己终止时由系统自动释放资源
- 线程共享资源：
  - 虚拟地址空间（代码段、堆、全局变量）
  - 文件描述符表
  - 用户 ID、组 ID
  - 信号处理函数
  - 页表、VMA
- 线程控制块（TCB）：
  - CPU 寄存器上下文：通用寄存器、程序计数器 PC、栈指针 SP
  - 线程栈信息：用户栈地址、栈大小
  - 线程 ID (TID)：内核线程 ID
  - 线程局部存储 TLS：线程私有全局变量
  - 信号屏蔽字（信号掩码）：决定屏蔽哪些信号
  - 线程状态
  - 调度相关
  - 返回值、退出状态
- PCB 可以指向该进程所有 TCB 链表
- 系统调用：
  - ==clone==：进程、线程都由 clone 创建，线程可以选择共享的资源
  - gettid：获取内核线程 ID (TID)
  - tgkill：向**指定线程发送信号**
  - exit_group：终止整个进程，所有线程全部退出
  - futex：实现用户态的同步原语，用户态快速互斥，线程阻塞
- 线程地址空间（按顺序）：
  - 与进程相比，只有栈不同，每个线程有自己的独立栈
    - 主线程栈：进程的栈段
    - 子线程栈：mmap匿名映射区
  - 多线程访问堆需要加锁保护
    ![632](../computer-organization/7557439f31e8c11dd7d95b8dee29e937cd9937a7.png)
- 线程状态转移：Linux 下线程就是 LWP 轻量级进程，内核`task_struct`，线程状态与进程状态完全同一套
- 线程切换（内核态）：
  - 线程上下文：
    - 硬件上下文：通用寄存器、PC 程序计数器、SP 栈指针、段寄存器、标志寄存器
    - 内核上下文：独立内核栈
    - 用户上下文：线程状态、调度相关、信号屏蔽字、TID、TLS
  - 触发时机：
    - 时钟中断
    - 线程阻塞：futex_wait、sleep、IO
    - 优先级线程抢占
    - 线程调用 sched_yield 让出 CPU
  - 切换流程（同进程）：
    - A从用户态切换为内核态
    - 内核把A的完整上下文全部保存到A的TCB中，修改线程 A 状态
    - 调度器运行，选择下一个进程B
    - 从B的TCB将所有内容恢复到当前上下文（CR3 不变，页表不变，TLB 不需要刷新）
    - 硬件指令，B从内核态切换为用户态
- 跨进程切换：与同进程相比，上下文变多了，而且要修改CR3基础器和刷新TLB缓存
- 线程调度：Linux 内核调度对象是`task_struct`，因此策略与进程一致
  \### NPTL库（pthread）
- 老LinuxThreads库有一个管理线程，专门用于管理其他工作线程的线程，包括接收进程终止信号、阻塞主线程、等待工作线程结束、回收线程栈内存等，但是增加了额外开销，也不能利用多核，这个库逐渐被废弃了
- NPTL库：
  - 线程不再是进程
  - 线程管理工作内核完成
  - 同一进程的线程可以在不同核心上
  - 线程同步内核自动完成
- pthread API
  - `pthread_t *thread`：输出，线程句柄
  - `attr`：线程属性pthread_attr_t，传 NULL 使用默认属性
  - `start_routine`：线程入口函数，签名固定 `void* func(void*)`
  - `arg`：传给入口函数的参数
  - state：取消状态
  - type：取消类型
    \`\`\` c
    // 创建线程
    int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
    void *(*start_routine)(void*), void *arg);

// 等待线程结束，回收资源，获取返回值
int pthread_join(pthread_t thread, void \*\*retval);

// 分离线程：线程退出自动释放资源，不能join
int pthread_detach(pthread_t thread);

// 线程主动退出
void pthread_exit(void \*retval);

// 获取本线程pthread_t（用户态ID，不是内核TID）
pthread_t pthread_self(void);

// 判断两个pthread_t是否相等
int pthread_equal(pthread_t t1, pthread_t t2);

// 发送取消请求（只是发请求，不代表线程马上退出）
int pthread_cancel(pthread_t thread);

// 设置本线程是否允许被取消
int pthread_setcancelstate(int state, int *oldstate);
// 设置取消类型：延迟取消 / 异步取消
int pthread_setcanceltype(int type, int *oldtype);

// 手动设置取消点；没有系统调用时主动检测取消标记
void pthread_testcancel(void);

// 两种state
PTHREAD_CANCEL_ENABLE // 默认：允许被取消
PTHREAD_CANCEL_DISABLE // 屏蔽取消；收到取消请求，挂起，不退出
// 两种type
PTHREAD_CANCEL_DEFERRED // 默认：延迟取消，必须到达【取消点】才退出
PTHREAD_CANCEL_ASYNCHRONOUS// 异步取消：随时可以终止线程（极少用，危险）

    - 线程属性pthread_attr_t设置
    ``` c
    // 初始化属性对象，填充系统默认值
    int pthread_attr_init(pthread_attr_t *attr);
    // 销毁属性对象，释放内部资源
    int pthread_attr_destroy(pthread_attr_t *attr);

    // ---------- 分离状态 detachstate 最常用 ----------
    // PTHREAD_CREATE_JOINABLE(默认) / PTHREAD_CREATE_DETACHED
    int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
    int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);

    // ---------- 线程栈大小 stacksize ----------
    // stacksize：字节，必须 >= PTHREAD_STACK_MIN，页对齐；Linux默认8MB
    int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
    int pthread_attr_getstacksize(const pthread_attr_t *attr, size_t *stacksize);

    // 极少用：用户自己提供栈内存空间(mmap分配)
    int pthread_attr_setstack(pthread_attr_t *attr, void *stackaddr, size_t stacksize);
    int pthread_attr_getstack(const pthread_attr_t *attr, void **stackaddr, size_t *stacksize);

    // ---------- 调度继承 inheritsched ----------
    // PTHREAD_INHERIT_SCHED(默认继承父线程) / PTHREAD_EXPLICIT_SCHED 使用attr内调度参数
    int pthread_attr_setinheritsched(pthread_attr_t *attr, int inheritsched);
    int pthread_attr_getinheritsched(const pthread_attr_t *attr, int *inheritsched);

    // ---------- 调度策略 schedpolicy ----------
    // SCHED_OTHER(分时默认)、SCHED_FIFO、SCHED_RR(实时需要root)
    int pthread_attr_setschedpolicy(pthread_attr_t *attr, int policy);
    int pthread_attr_getschedpolicy(const pthread_attr_t *attr, int *policy);

    // ---------- 调度优先级 schedparam ----------
    int pthread_attr_setschedparam(pthread_attr_t *attr, const struct sched_param *param);
    int pthread_attr_getschedparam(const pthread_attr_t *attr, struct sched_param *param);

    // ---------- 线程作用域 scope ----------
    // Linux NPTL仅支持 PTHREAD_SCOPE_SYSTEM；PTHREAD_SCOPE_PROCESS不支持
    int pthread_attr_setscope(pthread_attr_t *attr, int scope);
    int pthread_attr_getscope(const pthread_attr_t *attr, int *scope);

### 互斥与同步

- 互斥的实现：锁（自旋、互斥）、信号量、原子操作、无锁编程
- 同步的实现：锁（自旋、读写）、条件变量、信号量、管道（socket）、屏障
  \#### 生产者消费者模型
- 生产者线程生产数据放入队列，消费者线程从队列取出数据消费
- 互斥：多个线程不能同时读写队列
- 同步：消费者等待生产者放数据，生产者等待消费者取数据
- 经典方案：互斥锁+2 \* 条件变量
- 方案中为什么要使用while？
  - 虚假唤醒：没有任何人调用 signal /broadcast，wait 也可以正常返回（底层BUG）
  - 多消费者同时唤醒：另一个人已经抢走了条件变量，等到我就啥也没有了
  - 广播唤醒：理由同上
- 条件变量版本：
- 生产者：

``` cpp
pthread_mutex_lock(&mtx);

// 队列满，等待队列有空位，while处理虚假唤醒
while(queue.size == N)
{
    pthread_cond_wait(&not_full, &mtx);
}

// 生产，放入队列
queue.push(data);

// 通知：队列现在不为空，唤醒消费者
pthread_cond_signal(&not_empty);

pthread_mutex_unlock(&mtx);
```

- 消费者：

``` cpp
pthread_mutex_lock(&mtx);

//队列空，等待数据到来
while(queue.size == 0)
{
    pthread_cond_wait(&not_empty, &mtx);
}

//取出数据
data = queue.pop();

//通知：队列现在有空位，唤醒生产者
pthread_cond_signal(&not_full);

pthread_mutex_unlock(&mtx);

//消费数据（耗时操作放在锁外面！）
consume(data);
```

- 信号量版本
- 生产者：

``` cpp
sem_wait(&empty);
sem_wait(&mutex_sem);
queue.push(data);
sem_post(&mutex_sem);
sem_post(&full);
```

- 消费者：

``` cpp
sem_wait(&full);
sem_wait(&mutex_sem);
data = queue.pop();
sem_post(&mutex_sem);
sem_post(&empty);
```

#### 读者-写者模型（读写锁）

- 一组共享资源，存在一类只读取共享数据的线程，和只修改共享数据的线程，读者可以并发读，写者只能独占写
- 饥饿问题：读者源源不断，写者永远得不到执行机会；或写者频繁，读者永远得不到机会。
- 读者优先：读者在读，后续读者可以继续读，写者要全部退出才能写，写者饥饿
- 写者优先：写者在等待时，阻止后续读者进入，写者优先抢占，读者饥饿
- 公平排队：全局排队锁，谁先来抢到谁先用
- 工程实现：读写锁`std::shared_mutex`，**标准没有规定是读者优先还是写者优先**，由操作系统库实现决定

``` cpp
std::shared_mutex rw_mtx;

//读者
rw_mtx.lock_shared();
//读共享数据
rw_mtx.unlock_shared();

//写者
rw_mtx.lock();
//修改共享数据
rw_mtx.unlock();
```

#### 锁（互斥）

- 用于解决多线程情景下**临界区**竞争，保证多线程互斥访问共享资源
- 锁与中断：只有内核自旋锁会搭配关中断，普通 mutex 互斥锁、用户态 pthread 自旋锁，加锁不会关闭中断
  \##### 自旋锁
- 拿不到锁，线程**原地循环忙等**，不放弃 CPU，仅适用短临界区
- 特点：不切换线程无上下文切换开销，但CPU空转
- ==自旋锁严禁调用 sleep、futex 阻塞操作，不然别的线程无法拿到同一把自旋锁了，不能递归加锁，不然自己把自己锁死了==
- 硬件原子指令：
- TAS Test‑and‑Set 测试并设置

``` cpp
// lock变量：0空闲，1占用，以下都是原子操作不可分割
bool TestAndSet(int *lock)
{
    old = *lock;
    *lock = 1;     //强制把锁置为1
    return old;
}

while( TestAndSet(&lock) == 1 ); //一直循环，直到返回0（空闲）
//临界区
lock = 0; //解锁
```

- CAS Compare‑and‑Swap 比较交换（主流）

``` cpp
// 原子执行
int CAS(int *addr, int expect, int newval)
{
    old = *addr;
    if(*addr == expect)
        *addr = newval;
    return old;
}

// lock=0空闲；lock=1占用
while( CAS(&lock, 0, 1) != 0 );
//临界区
lock = 0; //解锁
```

- TTAS Test‑Test‑And‑Set 测试‑测试‑设置

``` cpp
//第一步：普通读，循环探测锁是否空闲（只读，不写内存）
while( lock == 1 ) ;
//锁看起来空闲，再执行原子TAS去抢锁
while( TestAndSet(&lock) == 1 );
```

- 实现方式：
  - 应用级：用户态自旋锁，pthread库支持
  - 内核级：spin_lock、spin_unlock系统调用
    \##### 互斥锁
- 谁 lock，谁必须 unlock；不能 A 加锁 B 解锁
- 拿不到锁，线程进入内核阻塞休眠，让出 CPU，适用长临界区
- 特点：有上下文切换开销，CPU利用率高
- 实现方式：
  - 应用级：用户态互斥锁，pthread库支持
  - 内核级：mutex_init、mutex_lock、mutex_unlock系统调用（底层futex）
- POSIX API：
  - 数据结构：`pthread_mutex_t mutex;`
  - `attr`：传`NULL`使用默认属性。
    \`\`\` c
    // 静态初始化，只能用于共享变量
    pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
    // 动态初始化，支持局部变量
    int pthread_mutex_init(pthread_mutex_t *restrict mutex,
    const pthread_mutexattr_t *restrict attr);

// 加锁：拿不到锁就阻塞
int pthread_mutex_lock(pthread_mutex_t *mutex);
// 非阻塞加锁：拿不到直接返回EBUSY，不阻塞
int pthread_mutex_trylock(pthread_mutex_t *mutex);
// 解锁：必须是持有锁的线程调用
int pthread_mutex_unlock(pthread_mutex_t *mutex);
// 销毁互斥锁
int pthread_mutex_destroy(pthread_mutex_t *mutex);

    - 互斥锁属性 pthread_mutexattr_t
        - type：
            - `PTHREAD_MUTEX_NORMAL` 普通锁，同一线程重复 lock：死锁；非持有者 unlock：未定义行为
            - `PTHREAD_MUTEX_ERRORCHECK` 检错锁，重复 lock 返回`EDEADLK`；非持有者 unlock 返回`EPERM`；解锁未上锁锁返回 EPERM
            - `PTHREAD_MUTEX_RECURSIVE` 递归锁，同一线程可多次 lock，内部维护引用计数；lock N 次，必须 unlock N 次才释放
        - pshared：
            - `PTHREAD_PROCESS_PRIVATE`（默认）：仅本进程内多线程使用。mutex 放在普通内存。
            - `PTHREAD_PROCESS_SHARED`：多进程之间共享锁
        - robust：
            - `PTHREAD_MUTEX_STALLED` 默认：持有锁的进程崩溃，锁永久卡死，后续 lock 永远阻塞。
            - `PTHREAD_MUTEX_ROBUST`健壮锁：持有锁进程崩溃，锁标记为损坏，返回 **`EOWNERDEAD`**，需要调用`pthread_mutex_consistent(mutex);`修复才可以继续使用
        - protocol：
            - `PTHREAD_PRIO_NONE` 默认，无优先级保护
            - `PTHREAD_PRIO_INHERIT` 优先级继承
            - `PTHREAD_PRIO_PROTECT` 优先级保护
    ``` c
    pthread_mutexattr_t attr;
    pthread_mutex_t mutex;

    // 1.初始化属性对象
    int pthread_mutexattr_init(pthread_mutexattr_t *attr);

    // 2.设置各类属性
    // 锁类型
    int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type);
    int pthread_mutexattr_gettype(const pthread_mutexattr_t *attr, int *type);
    // 进程共享属性
    int pthread_mutexattr_setpshared(pthread_mutexattr_t *attr, int pshared);
    int pthread_mutexattr_getpshared(const pthread_mutexattr_t *attr, int *pshared);
    // 鲁棒性属性
    int pthread_mutexattr_setrobust(pthread_mutexattr_t *attr, int robust);
    int pthread_mutexattr_getrobust(const pthread_mutexattr_t *attr, int *robust);
    // 优先级保护
    int pthread_mutexattr_setprotocol(pthread_mutexattr_t *attr, int protocol);

    // 3.用属性初始化mutex
    pthread_mutex_init(&mutex, &attr);

    // 4.属性对象可以立即销毁，不影响mutex
    int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);

##### 基于锁的数据结构

- 并发计数器：[并发计数器](操作系统导论笔记.md#并发计数器：)
- 并发链表：[并发链表](操作系统导论笔记.md#并发链表：)
- 并发队列：[并发队列](操作系统导论笔记.md#并发队列：)
- 并发散列表：[并发散列表](操作系统导论笔记.md#并发散列表：)
  \#### 信号量
- 操作系统提供的同步原语，核心是**引用计数 + 等待线程队列**
- 基础操作：
  - P：wait，申请资源，计数-1，计数为0线程阻塞
  - V：post，释放资源，计数+1，计数\>0唤醒一个阻塞线程
- 系统调用（System V）
  - 当 key 传入 **`IPC_PRIVATE`**，直接创建一个匿名IPC 对象，**没有对外的 key**；别的进程无法通过 key 打开它

  ``` c
  int semget(key_t key, int nsems, int semflg); //创建获取信号量集
  int semop(int semid, struct sembuf *sops, unsigned nsops); //P、V 操作（加减信号量值）
  int semctl(int semid, int semnum, int cmd, ...); //控制信号量集（初始化、删除、获取状态）
  ```
- POSIX api：
  - 分为**无名信号量 (内存信号量)**、**有名信号量 (文件形式)**
  - POSIX 信号量头文件 `<semaphore.h>`，编译链接需要 `-pthread`
  - pshared:
    - 0：线程间同步，sem放在普通进程内存
    - 非0：进程间同步，sem必须放在共享内存shmem
  - value：信号量初始计数
  - oflag: O_CREAT \| O_EXCL，文件打开标志位参见[文件模式](../2.c++语言/C++primer笔记.md#==文件模式：==)
  - mode: 文件权限 0644
  - value: O_CREAT时指定初始值
    \`\`\` cpp
    // 初始化信号量
    int sem_init(sem_t \*sem, int pshared, unsigned int value);

// 无名信号量
// P操作：等待(减1，资源不足阻塞)
int sem_wait(sem_t *sem);
// P非阻塞：资源不够直接返回-1，errno=EAGAIN
int sem_trywait(sem_t *sem);
// 带超时P，超时返回-1 errno=ETIMEDOUT
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
// V操作：释放(加1，唤醒等待者)
int sem_post(sem_t *sem);
// 获取当前信号量计数值
int sem_getvalue(sem_t *sem, int *sval);
// 销毁无名信号量
int sem_destroy(sem_t *sem);

// 有名信号量
// 创建/打开有名信号量
sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value);
// 关闭当前进程的信号量句柄(不删除内核对象)
int sem_close(sem_t *sem);
// 删除内核中的有名信号量对象，其他进程仍打开则等全部close后才真正销毁
int sem_unlink(const char *name);

    - 类型：
        - 二元信号量：sem初始值为1，实现互斥功能，类似互斥锁（但是信号量支持跨线程，容易别的线程替你释放了很搞笑，所以平时别用）
        - 计数信号量：sem初始值为0，实现同步功能，类似条件变量，自带计数器不会信号丢失
    #### 原子操作
    - 原子操作是一系列硬件/微指令层面的命令无法被CPU打断（要么不做要么做完），往往用汇编语言实现，对应的就是x86 `lock cmpxchg`，ARM `ldrex/strex`
    - 应用：无锁数据结构、多线程计数器、线程间事件标记
    - std::atomic是 C++ 标准库的模板类，对原子操作做了上层封装，头文件`<atomic>`
    - `std::atomic<T>` 把类型 T 包装成原子类型
    - api：
    ``` cpp
    #include <atomic>

    std::atomic<int> a{0};

    // 1. 原子写 store
    a.store(10);
    a.store(20, std::memory_order_release);

    // 2. 原子读 load
    int v = a.load();
    int v2 = a.load(std::memory_order_acquire);

    // 也可以直接运算符简写（默认memory_order_seq_cst）
    a = 100;
    int x = a;

    // 返回旧值
    int old = a.fetch_add(1);   // 原子 cnt +=1，返回旧值
    int old2 = a.fetch_sub(2);  // 原子 cnt -=2

    // 位操作
    a.fetch_or(0x1);
    a.fetch_and(~0x2);
    a.fetch_xor(0x4);

- 判断：

``` cpp
std::atomic<int> a{0};
int expected = 0;

// 判断
// 成功返回true，如果不等：expected 会被赋值为 a 的当前真实值，返回false
if(a.compare_exchange_strong(expected, 100)){
    // compare_exchange_strong适合不循环，单次判断场景
};

// ✅标准循环CAS模板 weak
int exp = a.load();
while (!a.compare_exchange_weak(exp, exp + 1))
{
    // compare_exchange_weak必须放在while循环中使用
}
```

##### ==CAS==

- CAS是硬件原子指令，全函数不会被线程打断

``` cpp
bool CAS(内存地址, 预期值expected, 新值desired)
{
    if( *addr == expected )
    {
        *addr = desired;
        return true; // 修改成功：期间没有其他线程改动
    }
    return false;  // 修改失败：已经被别人改了
}
```

- 循环CAS（自旋锁）-对应`std::atomic::compare_exchange_weak`

``` cpp
int exp = atomic_var.load();
while( !CAS(&atomic_var, exp, exp+1) )
{
    //失败时exp会更新为内存最新值
}
```

- 单次CAS（只尝试一次）-对应`std::atomic::compare_exchange_strong`
- ==ABA 问题==：
  - ABA 是**CAS 无锁编程特有的经典 bug**，CAS**只检查变量的值，看不到中间发生过的状态变化**，变量从 A → B → 又改回 A；CAS 看到值依旧是 A，认为期间没有改动，执行修改
  - 数值场景 ABA 一般危害不大；**指针场景会直接内存崩溃**，是高危 bug
  - 解决方法：C++20已经支持使用智能指针了，可以解决这个问题
    \`\`\` cpp
    std::atomic<int> num{A};
    //T1
    int exp = num.load();
    //T1被切走

//T2: num = B; num = A;

//T1恢复，执行CAS
CAS(&num, exp, X);
//值依旧A，CAS成功，但中间发生过修改

    - 活锁问题：类似死锁状态，但是多个线程同时执行 CAS，互相抢占，所有人一直失败循环重试，线程不会阻塞，CPU 跑满，但谁都无法完成工作。
    ##### ==内存序==
    - CPU、编译器为提升性能会做**指令重排**，多线程会造成一个线程看到错乱的变量读写顺序，出现诡异 bug，内存序就是阻止这种越界重排
    ``` cpp
    int data = 0;
    std::atomic<bool> ready{false};

    //线程A
    data = 42;
    ready.store(true);

    //线程B
    if(ready.load()){
        // 理论：ready为true，data应该等于42
        // 但重排可能发生：ready.store先执行，data=42后执行
        // 线程B读到ready=true，但data还是0！
        std::cout << data;
    }

- 内存序类型

``` cpp
std::memory_order_seq_cst    // 顺序一致性，默认，所有原子操作，全局所有 CPU 看到同一个全局执行顺序（主流）
std::memory_order_acquire    // 获取，用于读
std::memory_order_release    // 释放，用于写
std::memory_order_acq_rel    // 获取+释放，用于读‑改‑写，生产者写用 release；消费者读用 acquire
std::memory_order_relaxed    // 仅原子，无同步约束
```

#### 无锁编程

- 只是不用互斥锁，用CAS实现自旋锁，可以在中断、信号处理函数中使用
- 无锁数据结构：
  - 无锁队列：（boost::lockfree）
    - **SPSC 单生产者‑单消费者**：1 个线程写、1 个线程读，实现简单，几乎没有 ABA 风险。
    - **MPSC 多生产者‑单消费者**：多个线程入队，一个线程出队；主流。
    - **MPMC 多生产者‑多消费者**：多线程同时入队出队，实现极其复杂，手写极易出错（直接用成熟库，绝不自己造轮子）
      \##### SPSC
- 环形数组结构，原子下标：`write_idx`（写位置），`read_idx`（读位置）
- 不需要 CAS；只依靠原子 load/store + 合适内存序 (`acquire/release`) 即可
- **不存在 ABA、活锁**，性能很高
- 伪代码：

``` cpp
//入队：生产者
if(队列未满)
{
    buf[write_idx] = item;
    write_idx.store( (write_idx+1)%size , release );
}

//出队：消费者
if(read_idx != write_idx.load(acquire))
{
    auto val = buf[read_idx];
    read_idx = (read_idx+1)%size;
    return val;
}
```

##### MPSC

- 大量后端代码使用，网络、日志、任务队列
- 多个线程并发入队（push），依靠 CAS 竞争尾部，只有一个线程做 pop 出队，`head/tail`两个原子指针
- **严重 ABA 风险**：节点释放后内存地址复用，CAS 被骗
- 风险指针：线程正在访问的节点标记为 "正在使用"；节点逻辑删除之后，要等到没有任何线程引用，才真正释放内存，避免地址复用
  \#### 条件变量
- `pthread_cond_t`，本身没有互斥能力，需要mutex保护

``` cpp
//消费者
pthread_mutex_lock(&mtx);
while(队列空)
    pthread_cond_wait(&cond, &mtx); //同步等待事件
取数据
pthread_mutex_unlock(&mtx);

//生产者
pthread_mutex_lock(&mtx);
放入数据
pthread_cond_signal(&cond); //通知事件发生
pthread_mutex_unlock(&mtx);
```

- 覆盖条件问题：
  - `pthread_cond_signal`只按照等待队列顺序唤醒一个等待线程，如果唤醒的进程条件不满足重新wait，这个signal已经消失了，以后的所有等待唤醒的即使满足要求也无法唤醒
  - 修复方案：`Pthread_cond_broadcast`唤醒所有等待进程，所有唤醒的都等着拿（==惊群效应==）
- 信号丢失问题：
  - 先post通知，wait线程没起来，post信号丢失了
  - 修复方案：设置一个全局的计数器记录发送的post信号的次数，或者用信号量
- POSIX API

``` c
// 静态初始化（全局 /static）
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
// 动态初始化（带属性，推荐）
int pthread_cond_init(pthread_cond_t *restrict cond,
                      const pthread_condattr_t *restrict attr);

// 等待：释放mutex，阻塞；被唤醒后重新持有mutex返回
int pthread_cond_wait(pthread_cond_t *restrict cond,
                      pthread_mutex_t *restrict mutex);
// 限时等待，绝对时间 abstime(系统CLOCK_REALTIME)
int pthread_cond_timedwait(pthread_cond_t *restrict cond,
                           pthread_mutex_t *restrict mutex,
                           const struct timespec *restrict abstime);
// 唤醒至少1个等待线程
int pthread_cond_signal(pthread_cond_t *cond);
// 唤醒全部等待线程
int pthread_cond_broadcast(pthread_cond_t *cond);
// 销毁条件变量
int pthread_cond_destroy(pthread_cond_t *cond);
```

- 条件变量属性
  - pshared：`PTHREAD_PROCESS_SHARED`：条件变量放共享内存，多进程可用。
  - clock：默认`CLOCK_REALTIME`；可以改为`CLOCK_MONOTONIC`不受系统时间回跳影响。
    \`\`\` c
    // 属性对象生命周期
    int pthread_condattr_init(pthread_condattr_t *attr);
    int pthread_condattr_destroy(pthread_condattr_t *attr);

// 设置进程共享属性：PTHREAD_PROCESS_PRIVATE / PTHREAD_PROCESS_SHARED
int pthread_condattr_setpshared(pthread_condattr_t *attr, int pshared);
int pthread_condattr_getpshared(const pthread_condattr_t *attr, int \*pshared);

// 设置时钟，用于pthread_cond_timedwait
int pthread_condattr_setclock(pthread_condattr_t *attr, clockid_t clock_id);
int pthread_condattr_getclock(const pthread_condattr_t *attr, clockid_t \*clock_id);

    #### 屏障
    - 等待指定数量线程全部到达屏障点，才统一放行继续往下执行
    - C++20：`std::barrier`、`std::latch`；POSIX：`pthread_barrier_t`
    - 类型：
        - latch（门闩）：一次性，不可重置，等待所有工作线程执行完毕
        - barrier（栅栏）：可复用，当第 N 个线程到达，触发阶段完成，所有阻塞的线程全部唤醒放行，自动重置计数，进入下一轮
    - latch：
    ``` cpp
    #include <thread>
    #include <vector>
    #include <latch>
    #include <iostream>

    int main()
    {
        const int N = 4;
        std::latch done(N);   // 需要4个线程count_down

        std::vector<std::thread> ths;
        for (int i = 0; i < N; ++i)
        {
            ths.emplace_back([&, i](){
                std::cout << "thread " << i << " work\n";
                done.count_down(); // 完成，计数减1
            });
        }

        done.wait(); // 主线程阻塞，等全部4个线程完成
        std::cout << "all thread finished\n";

        for(auto& t : ths) t.join();
        return 0;
    }

- barrier：

``` cpp
#include <thread>
#include <vector>
#include <barrier>
#include <iostream>

int main()
{
    const int N = 3;
    // 3个线程参与；最后到达的线程执行回调
    std::barrier bar(N, [](){
        std::cout << "---- round finish ----\n";
    });

    auto work = [&](int id){
        for(int round = 0; round < 2; ++round)
        {
            std::cout << "thread:" << id << " round:" << round << " step1\n";
            bar.arrive_and_wait(); // 全部线程到齐，才往下执行

            std::cout << "thread:" << id << " round:" << round << " step2\n";
            bar.arrive_and_wait(); // 第二轮同步点
        }
    };

    std::vector<std::thread> ths;
    for(int i=0;i<N;i++) ths.emplace_back(work, i);
    for(auto& t : ths) t.join();
    return 0;
}
```

#### 管道

参见[匿名管道 Pipe（有血缘关系，半双工）](5.2%20并发编程.md#匿名管道%20Pipe（有血缘关系，半双工）)
\#### 线程安全函数
- 一个函数，**多个线程并发调用它，不管调度顺序，输出结果永远正确**
![计算机组成原理_CSAPP\|590](../computer-organization/8cedc9bb43e99f6a99a6396d24b3be4071df24c8.png)
- 线程安全但不可重入函数（弱线程安全）：内部用锁保护共享变量，但信号中断嵌套调用会死锁
- 可重入函数（强线程安全）：不使用全局 /static 变量；全部使用栈上局部变量和参数（引用传递的参数不能是共享数据）
\### 基于事件的并发
参见[网络IO模型](../6.%20网络编程/网络IO模型.md)
\### 常见并发问题
- 数据竞争：共享变量无互斥保护（互斥锁、原子操作解决）
- 活锁：互相持有锁永不解开，一起自旋，类似死锁
- 饥饿：线程一直等待永远无法推进（先来先服务、随机算法、公平锁）
- ABA 问题：[CAS](5.1%20计算机组成原理.md#==CAS==)
- 伪共享：多个线程修改**不同变量，但这些变量落在同一个 CPU 缓存行**，修改一个变量就会让整个缓存行失效，多核反复缓存同步，性能暴跌（`alignas(64)`做缓存行对齐）
- 指令重排：[原子操作](5.1%20计算机组成原理.md#原子操作)，volatile 只阻止编译器优化，不阻止 CPU 乱序
- 优先级反转：解决：优先级继承、优先级天花板协议

``` txt
高优先级：等待锁
中优先级：占用CPU疯狂跑
低优先级：持有锁，得不到CPU无法释放
```

- 锁粒度问题：粒度过大性能差，粒度过小开销大还容易出bug
- 虚假唤醒：[条件变量](5.1%20计算机组成原理.md#条件变量)
  \#### 死锁
- 互相持有锁永不解开，一起阻塞
- 必要条件：
  - 互斥资源
  - 占有且等待
  - 不可剥夺
  - 互相等待
- 解决：
  - 预防：破坏四个条件（统一顺序按相同顺序获取锁、设置超时、锁打包在一起原子获取）
  - 避免：运行时拒绝危险申请（银行家算法）
  - 检测恢复：允许发生，检测后恢复
  - 忽略：不检测不处理，崩溃就重启（实际做法哈哈哈）
- 死锁检测：构建资源分配图
  - 点：线程结点 T、资源结点 R
  - 边：分配边 R→T、请求边 T→R
  - 出现环路表示发生死锁
  - 操作系统维护资源分配图，定时扫描图找环
- 恢复策略：
  - 终止环路内部分 / 全部死锁线程，释放资源（常用）
  - 剥夺资源，强制把某些线程资源抢过来（基本不可能）
- 死锁检测工具：
  - glibc 内置死锁检测（pthread）：环境变量开启`LD_PRELOAD=/usr/lib/libpthread.so.0`和`PTHREAD_DEBUG=1`，编译开启 `-g`
  - valgrind‑helgrind：`valgrind --tool=helgrind ./your_program`
  - clang‑thread‑sanitizer (推荐)：编译时添加参数`g++ main.cpp -fsanitize=thread -g -o app`
    \### 并行编程
- 并行化：利用多核将线程计算结果保存在全局数组中
- 并行指标：
- 加速比：![计算机组成原理_CSAPP](../computer-organization/f87178790ce87f06f8922c7e298cca85eff833da.png)，T1是单核执行时间，Tp是多核执行时间
- 效率：![计算机组成原理_CSAPP](../computer-organization/b6ca8eb1cf0225e0ce2989458c2b7f53ced4e21e.png)，p是核心数
  \## 持久性
- 核心：文件系统
  \### IO设备
  ![计算机操作系统_OSTEP\|615](../computer-organization/6771db3059ed46e57995eaa009de7e83f20e4895.png)
- 组成：
  - IO总线：系统总线连接 CPU‑内存‑控制器
  - IO设备：物理硬件，键盘、磁盘、网卡，速度慢、种类多
    - 字符设备：按字节流传输，键盘、鼠标、串口、终端、声卡、管道
    - 块设备：按块 (512B/4K)传输、机械硬盘、固态硬盘、U盘、光盘
  - 设备控制器：设备板上芯片，寄存器 + 缓存；屏蔽设备硬件细节，接收 CPU 命令，控制硬件，上报状态
    - 控制寄存器：CPU 写，下发命令
    - 状态寄存器：CPU 读，获取设备状态
    - 数据寄存器：供数据读写
  - 操作系统IO软件栈：中断处理程序→设备驱动→设备独立层→用户层库
- 访问设备控制器的方式：
  - 直接访问：有独立的IO地址空间，使用in/out内置指令访问寄存器
  - MMIO内存映射：硬件BIOS初始化的时候，就给设备控制器映射一段物理地址，相当于机器整体物理地址变长了，普通mov指令就可以访问寄存器
- IO控制方式：
  - CPU轮询：不停读状态寄存器，循环判断设备是否完成，CPU利用率低，只适合低速设备
  - 中断驱动：设备完成后发送**硬件中断**信号，CPU 保存现场，进入中断处理程序，大量数据反复触发中断效率低，只适合小数据IO传输
  - DMA直接存储器访问：DMA控制器接管内存与设备间的数据传输，任务完成后才向CPU发送一次中断，大量IO可以大幅提高效率，适合大数据IO传输
  - IO通道访问：独立硬件处理器自带指令集，自己控制IO，CPU只下发任务，通道全部独立完成
- 软件分层：
  - 中断处理程序：硬件中断触发，硬件相关处理，向上通知驱动
  - 设备驱动程序：in/out等特权指令，读写寄存器，实现设备具体操作
  - 文件系统：对设备的逻辑抽象，对上层提供统一接口 open/read/write
  - 用户层：库函数、虚拟设备
    ![661](../computer-organization/20a619efceb9cd354555e1518f283a31b6fdebd0.png)
- IO库类型：

| 名称 | 层次 | 句柄 | 缓冲情况 | 典型应用 | 异步信号安全 | 主要接口方法 |
|----|----|----|----|----|----|----|
| Unix‑IO<br><br>系统调用 IO | 内核系统调用层 | int fd 文件描述符 | 无用户态缓冲区；内核 Page Cache 缓存磁盘数据 | 磁盘文件、socket、管道，底层基础接口 | ✅异步信号安全 | `open`、`close`、`read`、`write`、`lseek` |
| 标准 IO stdio<br><br>C 标准库 IO | 用户态 C 库，封装 Unix‑IO | FILE\* 文件流 | 自带用户态缓冲区：全缓冲 / 行缓冲 / 无缓冲 | 普通本地文件，格式化输入输出；**不适合信号处理、网络** | ❌不是异步信号安全 | `fopen`、`fclose`、`fread`、`fwrite`、`printf`、`scanf`、`fgets` |
| RIO 健壮 IO<br><br>CSAPP 示例库 | 用户态辅助库，封装 Unix‑IO | int fd | rio_readn：无用户缓冲<br><br>rio_readlineb：内部行缓冲 | 网络编程；自动处理短计数、EINTR 信号打断 | ✅可在信号 handler 调用（底层调用 read/write） | `rio_readn`、`rio_writen`、`rio_readlineb` |

#### 中断驱动IO（字符流设备）

- 全流程：
  - 用户进程发起 read 系统调用，切换内核态，进入设备驱动程序
  - 驱动使用out指令写设备控制寄存器，下发启动和读取命令，阻塞当前进程等待IO完成，CPU发生进程调度运行别的进程
  - 设备采集数据，把数据放入数据寄存器，向中断控制器发送中断信号IRQ
  - CPU收到中断信号，保存当前进程上下文，进入中断处理程序
  - 中断处理程序：
    - `in`指令读设备的数据寄存器，把数据读到 CPU 内部寄存器
    - 将 CPU 寄存器的数据拷贝到内核缓冲区
    - 清除设备中断标志，给 8259 发送 EOI 中断结束
    - 唤醒之前被阻塞的用户进程
    - 恢复被打断的程序上下文
  - 回到原来进程继续执行
  - 根据操作系统策略，发生进程调度，驱动把内核缓冲区的数据，拷贝到**用户进程的用户空间缓冲区**，read 系统调用返回
    \#### DMA（块设备）
- DMA端口寄存器：
  - **内存起始地址寄存器**：数据要写到内存哪个物理地址
  - **设备地址寄存器**：I/O 设备地址
  - **传输字节 / 计数寄存器**：一共要传多少字节
  - **控制寄存器**：方向、启动等命令
- 全流程：
  - 用户进程发起 read 系统调用，切换内核态，进入设备驱动程序
  - 驱动程序使用out指令配置DMA 控制器寄存器参数，然后CPU干别的事去了
  - DMA 向总线仲裁器申请系统总线，拿到总线
  - DMA 控制器在 **I/O 设备 ↔ 主存** 之间直接传输数据，每传输一个单元，内部计数寄存器自动减 1
  - 计数寄存器减到 0，DMA 控制器释放总线，还给 CPU，向 CPU 发出**硬件中断**
  - CPU 收到中断，进入中断处理程序，唤醒等待这个 I/O 的阻塞进程
    \##### 磁盘（机械硬盘）
- 尽量顺序或大块方式使用磁盘，磁盘随机写入效率很低
- 缓冲区：块设备读写，内存中缓冲磁盘块，减少磁盘 IO 次数
- 物理结构：
  - **盘片 (platter)**：圆形磁性圆盘，多张盘片堆叠；两面都可以存数据。
  - **磁道 (track)**：盘片上一圈一圈同心圆，同一半径的圆环。
  - **柱面 (cylinder)**：**所有盘片上半径相同的全部磁道组成一个柱面**。
  - **磁头 (head)**：每个盘面一个磁头；读写盘片数据。
  - **扇区 (sector)**：磁道被切分成一段段圆弧；**扇区是磁盘硬件最小读写单元，一般 512 字节**。
- 磁盘容量：磁盘能记录的最大位数
  ![计算机组成原理_CSAPP](../computer-organization/d1ac970a8e8fb0ce254236449e75df65c2dfb798.png)
- 磁盘读取时间 = **寻道时间 + 旋转延迟 + 数据传输时间**
  - 寻道时间：磁头移动到目标柱面（磁道）所花时间
  - 旋转延迟：盘片旋转，让目标扇区转到磁头下方的时间
  - 传输时间：扇区经过磁头，真正读写数据的时间
- 磁盘控制器：==逻辑块==一般 4KB，是文件系统操作磁盘的单位，由连续多个扇区组成，磁盘控制器维护逻辑块号和实际扇区之间的映射关系，对应磁盘地址（柱面号、磁头号、扇区号）
- 磁盘调度算法：多个请求排序，选择哪个请求先执行
  - SSTF 最短寻道优先：每次选择离当前磁头位置最近的请求
  - FIFO 先来先服务：按请求到达顺序处理
  - SCAN 电梯算法：磁头向一个方向移动，沿途处理所有请求；到达边界后调转方向折返
  - C‑SCAN 循环扫描：磁头单向移动，处理沿途请求；到达最末端，**直接跳转到磁盘起始端**
  - SPTF 最短定位时间优先：结合寻道时间和旋转时间确定最短时间，磁盘内部计算执行
    \###### RAID容灾备份
- 多个磁盘构建更快、更大、更可靠的磁盘系统，可以安装基于SCSI的RAID存储阵列
- RAID0（条带化）：至少2块盘，数据分割分散
  ![](../computer-organization/a6797bf874a3afa2539a6903fb330d6da1476666.png)
- RAID1（镜像）：至少2块盘，数据分割后备份分散
  ![](../computer-organization/ae8483b8edc123c28c86dfcbef7274af1addc8c5.png)
- RAID4（奇偶校验）：至少3块盘，RAID0基础上设置1块奇偶校验盘
  ![](../computer-organization/8a1e6299b7b21604090b8ee4dc1e7846d803aba9.png)
- RAID5（旋转奇偶校验）：至少3块盘，最常用，RAID4基础上校验循环分散到所有盘
  ![](../computer-organization/80b851d3b754ba7c5179f5675c9ede640e96467b.png)
  \### 文件系统（ext4为例）
- 文件系统是操作系统管理IO设备的中间设施，向上为用户层提供open/read/write等接口，向下操作设备驱动程序管理块设备
- Linux将所有I/O设备都抽象为Linux文件，输入和输出操作都可以看成是对文件的读写
- 文件类型：文件、目录项（文件名 + inode 号）、套接字
- 文件描述符fd：用于标识文件的唯一符号。在创建进程时，内核会默认打开三个文件，标准输入、标准输出和标准错误
- 文件描述符表：进程独有，每个文件描述符fd指向一个file结构，见[索引三层结构](5.1%20计算机组成原理.md#索引三层结构)
- 功能：
  - 抽象IO设备：设备即文件，目录代替块号
  - 元数据管理：文件大小、权限、时间、位置信息
  - 缓存加速：减少磁盘访问
  - 保护：权限控制、访问检查
  - 空间回收
- 主流文件系统：ext4（linux，inode）、xfs（linux）、fast32（windows）、ntfs（windows，inode）
- 自己实现的文件系统：不太重要，[快速文件系统](操作系统导论笔记.md#快速文件系统：)、[日志结构文件系统LFS](操作系统导论笔记.md#日志结构文件系统LFS：)
- 文件系统调用：
  - open：打开或者创建文件，返回文件描述符 fd
  - close：关闭文件描述符，释放内核打开文件资源
  - read：从打开的文件中读取数据到用户缓冲区
  - write：将用户缓冲区的数据写入文件（写入 Page Cache，不保证直接落盘）
  - lseek：修改文件读写偏移量，实现随机访问
  - fsync：将文件的数据与元数据强制刷写到磁盘持久化
  - fdatasync：仅将文件数据强制刷写到磁盘，不强制更新元数据
  - creat：创建普通文件，等价于带创建标志的 open
  - mkdir：创建目录
  - rmdir：删除空目录
  - link：创建硬链接，增加目录项，inode 硬链接计数 nlink 加 1
  - symlink：创建符号链接（软链接），新建独立 inode 存储目标路径
  - unlink：删除文件目录项，inode 硬链接计数 nlink 减 1
  - getdents /readdir：读取目录，获取目录项（文件名、inode 编号）
  - stat：通过文件路径获取文件 inode 元数据信息
  - fstat：通过文件描述符 fd 获取文件 inode 元数据信息
  - chmod：通过路径修改文件访问权限 rwx
  - fchmod：通过文件描述符 fd 修改文件访问权限 rwx
  - chown：通过路径修改文件所有者 uid、所属组 gid
  - fchown：通过文件描述符 fd 修改文件所有者 uid、所属组 gid
  - utime：修改文件的访问时间与修改时间
- 文件结构：
  ![计算机组成原理_CSAPP](../computer-organization/eeedf22a47d7242e60fffb52aecdf2e25fec5701.png)
  \#### 索引三层结构
- 进程：文件描述符fd表，进程私有，表项：`fd：file结构体指针`
- file 结构体（内核）：当前读写偏移offset、文件模式、引用计数、指向inode的指针
- inode（磁盘）/v-inode（内存）：文件元数据（大小、权限、uid/gid、硬链接计数、磁盘块指针）
- 真实磁盘块：保存真实文件

```
graph TD
    subgraph 用户进程A
        FD1[fd = 3]
        FD2[fd = 4]
    end
    subgraph 用户进程B
        FD3[fd = 5]
    end

    %% 第一层：进程文件描述符表
    FDT_A["进程A文件描述符表<br/>fd → file*"]
    FDT_B["进程B文件描述符表<br/>fd → file*"]

    %% 第二层：file结构体(打开实例)
    FILE1["file结构体1<br/>• 文件偏移offset<br/>• 打开标志flags<br/>• 引用计数f_count=2"]
    FILE2["file结构体2<br/>• 文件偏移offset<br/>• 打开标志flags<br/>• 引用计数f_count=1"]

    %% 第三层：内存v‑inode
    VINODE["内存v‑inode<br/>对应磁盘inode<br/>元数据、块指针、nlink"]

    %% 连线
    FD1 --> FDT_A --> FILE1
    FD2 --> FDT_A --> FILE1
    FD3 --> FDT_B --> FILE2

    FILE1 --> VINODE
    FILE2 --> VINODE
```

- 同一个进程只有一个对应file 结构体（子进程会复制父进程的fd表，共享同一个文件），可以有多个fd同时指向它
- 同一个inode也可以被多个file结构体指向操作
- 不一致性问题：内核对inode做互斥保护，但是不提供并发的同步顺序，同时写内容会乱
  \##### ext4 磁盘布局
- ext4 把**整个分区切分成若干个块组 Block Group**

``` txt
ext4分区
├─块组0（主超级块在这里）
├─块组1（超级块备份）
├─块组2
└─……块组N
```

- 单个**块组内部顺序**
- Superblock 超级块（备份，部分块组才有）：保存整个文件系统**全局信息**
  - 块大小（常见 4KB）
  - 总块数、总 inode 数量
  - 每个块组有多少块、多少 inode
  - 文件系统状态（正常 / 出错）、UUID、特性标记（日志、extent 等）
- GDT 块组描述符表 Group Descriptor Table：
  - 本块组的块位图磁盘位置
  - 本块组 inode 位图磁盘位置
  - 本块组 inode 表起始位置
  - 本块组空闲块、空闲 inode 统计计数
- Block Bitmap 块位图：1bit 代表本块组一个数据块，bit=1：已分配；bit=0：空闲
- Inode Bitmap inode 位图：1bit 代表本块组的一个 inode，bit=1：inode 已占用；bit=0 空闲
- Inode Table inode 表：inode 数组，每个 inode 固定 256 字节
- Data Blocks 数据块区：
  - 普通文件内容
  - **目录文件**：目录项（文件名 + inode 号）
  - 间接索引块
  - 符号链接内容（长软链接；短 fast‑symlink 直接放 inode 内部）
    \##### inode
- inode编号：`ls -i`
- 目录项（特殊文件）：保存目录内的文件名+inode 编号
- 硬链接：`link("a.txt","a_hard")`，多个不同文件名指向同一个inode，inode的nlink计数加1，不能跨文件系统
- 软链接：`symlink("a.txt","a_soft")`，创建全新的独立inode，指向原文件，可以跨文件系统
- 软链接指向的原文件如果被删除了，这个链接就会变成悬空链接，访问报错
- unlink：inode的nlink--，直到为0才删除，而且还要等所有使用inode的进程close才删除对应数据块和inode
- 结构：
  - 文件类型：普通文件、目录、符号链接、设备文件
  - 文件大小（字节）
  - 访问权限 rwx（ugo 用户、组、其他）
  - uid 文件所有者 ID，gid 所属组 ID
  - 时间
    - atime：访问时间（read）
    - mtime：内容修改时间（write）
    - ctime：元数据修改时间（权限、硬链接变化）
  - **nlink：硬链接计数**，多少个文件名指向该 inode
  - 磁盘块指针数组：直接指针、一级间接、二级间接、三级间接，找到文件的数据块
- 寻址（ext4）：
  - **12 个直接指针**：直接指向数据块；小文件直接访问对应数据块
  - **一级间接指针**：指向一个索引块，索引块存放一批数据块编号
  - **二级间接指针**：指向索引块，索引块再指向一级索引块
  - **三级间接指针**：三级搜索，范围巨大
- v-node：把磁盘上的inode读取到内核内存中，内核会在合适时机写回更新
  \##### 崩溃一致性
- 系统任意时刻断电 / 崩溃，重启之后，磁盘上文件系统元数据结构不能损坏，保证处于合法状态；要么全部修改生效，要么完全不生效
- 应用数据一致性：不保证，需要应用调用fsync
- 文件系统一致性：
  - 需要保证的内容：inode、位图、目录、超级块
  - 保护手段-日志：日志区是磁盘上一块独立区域
    - 写前日志，真正修改磁盘元数据之前，先把本次事务要做的变更，写入日志磁盘，关键任务标记commit
    - 把事务应用到真正磁盘位置
    - 事务全部完成，清除日志
    - 崩溃发生，检测日志，重放commit标记日志，丢弃未commit的事务
- 日志模式：
  - data=ordered：数据已经落盘，再将元数据写入日志
  - data=journal：数据和元数据全部写入日志
  - data=writeback：元数据写入日志，不保证数据块落盘
- 数据完整性和保护：
  - 日志
  - fsync
  - 并发保护
  - 元数据校验和（ext4支持）
  - 备份副本（ext4超级块）
  - RAID
    \#### 逻辑块真正落盘方式
- 连续分配：连续磁盘块；随机访问好；外部碎片，扩容困难
- 隐式链接：块内存下一块号；无碎片；随机访问极差
- 显式链接 FAT：FAT 表保存块链接；**没有 inode**，FAT32
- 索引分配 inode：ext4 采用；直接 + 间接指针；支持随机访问
  \#### 空闲磁盘管理
- 位图：bit 标记块 /inode 空闲占用，ext4 使用
- 空闲链表：空闲块用指针链接
- 成组链接：Unix，空闲块分组管理
- 空闲表：记录连续空闲块起始 + 长度
  \#### 页缓存
- read 先查缓存；未命中才读磁盘
- write 写入 page cache，标记脏页，内核后台回写磁盘，**write 不直接落盘**
- fsync：强制数据 + 元数据落盘；fdatasync：仅强制数据落盘
  \# 异常控制流ECF
- 异常控制流：改变cpu正常指令流的事件，不按程序正常流程进行
- 横跨硬件（CPU异常）、操作系统（中断、进程切换）、用户态（信号、longjmp）

| 类型 | 触发来源 | 保存的 rip | 返回行为 | 典型例子 | 是否内核态处理 |
|----|----|----|----|----|----|
| **陷阱 Trap** | 程序主动执行指令 | 下一条指令 | 处理完执行**下一条指令Inext​** | `syscall`系统调用 | ✅内核异常处理程序 |
| **故障 Fault** | 指令执行出错 | 原指令 | 修复成功：**重新执行出错指令​；失败发信号终止** | 缺页异常 page‑fault | ✅内核异常处理程序 |
| **终止 Abort** | 硬件致命错误 | 无 | **不返回，直接终止进程，生成 core dump** | 硬件内存损坏 | ✅内核异常处理程序 |
| **外部中断 Interrupt** | 外设硬件异步事件 | 原指令 | 处理完继续执行**被打断的指令** | 定时器、网卡中断 | ✅内核中断服务程序 |
| **信号 Signal** | 内核软件事件，非硬件异常 | 下一条指令 | handler 执行完，回到**被打断指令的下一条​** | SIGINT、SIGSEGV | 内核标记信号；**handler 运行在用户态** |

## 异常

- 硬件检测到的事件，触发后跳转到内核异常处理程序
- 类型：
  - 陷阱 Trap：有意的、人为触发的异常，比如系统调用![计算机组成原理_CSAPP](../computer-organization/c9bb5dea40fbf0cb27567354c99d306b1c670c8a.png)

  - 故障 Fault：指令执行出错，**可以修复**

    - 修复成功：回到发生故障的指令重新执行，比如缺页异常
    - 修复失败：进程收到信号，终止进程![计算机组成原理_CSAPP](../computer-organization/992d53e9883cb937880c1ec1c6c1f9f81c13a5b8.png)

  - 终止 Abort：严重硬件错误，**不可恢复**，直接终止进程，比如硬件故障![计算机组成原理_CSAPP](../computer-organization/c56d28e591fb90103141e3b2c67dd1e121468871.png)

  - 中断 Interrupt：外设通过 APIC 发送中断信号给 CPU，中断处理程序，回到被打断指令，比如定时器、网卡中断![计算机组成原理_CSAPP](../computer-organization/4dbfb18824cdd361df4d7ac002aef173a9f9737f.png)
- 系统调用：
  ![计算机组成原理_CSAPP](../computer-organization/4b226212c2f04130fc5b46aec544aca8ce3db834.png)
  \### 异常处理程序（内核）：
- 异常号：有些异常号由处理器设计者分配，有些异常号由操作系统内核设计者分配，分别用来表示不同层次的异常
- 异常向量表（IDT），表里面每一项存对应异常类型的**异常处理程序入口地址**
  - 陷阱 Trap：系统调用入口函数
  - 故障 Fault：故障处理函数
  - 终止 Abort：异常处理函数（记录硬件错误，直接终止进程）
  - 中断 Interrupt：中断服务程序 ISR
  - 其中除了中断是异步的（CPU以外），其他都是同步的（CPU以内）
- 中断向量表基本也是合并在异常向量表里的
- `IDTR`寄存器记录 IDT 在内存的**基地址与限长**
- IDT表项内容：
  - **段选择子**：指定内核代码段
  - **偏移地址**：异常处理程序入口地址（64 位拆高低两部分）
  - **DPL 特权级**：什么特权级可以触发该异常
  - **P 有效位**：1 = 该表项有效
  - 门类型：陷阱门 / 中断门
  - 下标 = **异常向量号**，其中0~31号异常是由Intel架构师定义的异常，32~255号异常是由操作系统定义的中断和陷阱。
    \## 上下文切换
- 内核态：内核不是独立进程，而是现有进程的超级用户模式
- 模式（特权级 Ring0/Ring3）放在CS 代码段寄存器的 CPL（Current Privilege Level，当前特权级）字段，内核模式 Ring0，用户模式 Ring3
- Linux提供了用户态访问内核的接口/proc
- 模式切换场景：
  - 用户切内核：异常（陷阱、故障、中断）
  - 内核切用户：异常返回（陷阱、故障、中断）
- 模式切换流程：见[受限直接执行（LDE）](5.1%20计算机组成原理.md#受限直接执行（LDE）)，中断可以打断内核代码，此时**不发生**模式切换
- 三种异常的上下文切换区别：见[异常](5.1%20计算机组成原理.md#异常)的异常类型
- 进程切换流程（全程内核态）：见[受限直接执行（LDE）](5.1%20计算机组成原理.md#受限直接执行（LDE）)
  \## 信号（用户态）
- 进程间通信的手段，接受进程用信号处理程序处理
- 本质：一个整数编号（SIGINT=2、SIGSEGV=11）
- 发送方：内核，也可由其他进程通过 kill 发送
- 处理时机：返回用户态时检查是否有待处理信号
- kill：
  - pid 参数四种情况
    1.  pid \> 0：信号发送给 pid 指定进程
    2.  pid = 0：发送给**调用进程所在进程组内所有进程**
    3.  pid = -1：发送给除 init (1) 和自身以外系统所有进程
    4.  pid \< -1：取绝对值`-pid`，发送给该进程组全部进程
  - sig：信号编号；传 0，不发送实际信号，仅做进程存在性检测
  - errno：ESRCH 进程不存在；EPERM 权限不足

  ``` c
  int kill(pid_t pid, int sig);
  ```
- 信号状态：
  - **产生 (generate)**：事件发生，内核生成信号
  - **未决 (pending)**：信号已经产生，还没有递送给进程执行
  - 阻塞 (block)：信号进入未决，但暂时不递送
  - **递送 (deliver)**：进程响应这个信号，执行动作
- 信号处理方式：
  - **默认动作**SIG_DEF：系统预设
  - 忽略SIG_IGN：收到信号直接丢弃
  - 捕获：跳转到用户态注册的处理函数执行
- 信号数据结构：
  - signal_struct：**线程组共享信号信息**，见[进程控制块PCB](5.1%20计算机组成原理.md#进程控制块PCB)
    - `shared_pending`：发给整个进程的共享未决信号
    - `action[]`：信号处理函数表，线程组共享
  - task_struct-\>blocked：线程私有信号阻塞掩码，标记哪些信号被阻塞
  - task_struct-pending：线程私有未决信号
- 全流程：
  - 发送信号：内核或用户调用kill把信号发送给线程或进程的未决信号组中，同一信号多次发送，pending 里只保留 1 份，重复信号丢失
  - 阻塞信号：
    - 隐式阻塞：内核默认阻塞当前正在处理信号的同类型信号
    - 显式阻塞：用户调用sigprocmask修改本线程task_struct-\>blocked，对应bit置1，成功阻塞
  - 接收信号：
    - 扫描私有和公有pending
    - 检查本线程blocked，如果没有阻塞，那就接收，清除pending标记
    - 根据signal_struct-action决定行为进行处理
      \### 信号处理程序：
- 用户态下执行，在被打断的线程上运行，信号是进程级资源，但可以按线程设置信号掩码
- 因为处于用户态，所以接收到新信号时是可以被别的信号处理程序、异常处理程序中断的
- signal：
  - signum：要处理的信号编号
  - handler 三种取值：
    1.  自定义函数地址：收到信号调用该回调函数
    2.  `SIG_IGN`：忽略该信号
    3.  `SIG_DFL`：恢复信号系统默认处理动作

    ``` c
    typedef void (*sighandler_t)(int);
    sighandler_t signal(int signum, sighandler_t handler);
    ```
- signation：
  - signum：信号编号（SIGKILL、SIGSTOP 不能设置）
  - act：入参，新的信号处理配置；传 NULL 表示不修改
  - oldact：出参，保存原来的信号处理配置；传 NULL 不需要获取旧配置

  ``` c
  int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
  ```
- struct sigaction 结构体：
  - sa_mask：额外指定其它要阻塞的信号集sigset_t
  - sa_flags：
    - ==SA_RESTART==：被信号打断的**慢系统调用 (read/accept/select/epoll_wait)**，信号处理完内核自动重新调用，不会返回 EINTR
    - SA_SIGINFO：使用`sa_sigaction`回调，拿到 siginfo_t 附加信息（发送信号的 pid、uid 等）
    - SA_NODEFER：信号处理函数执行期间，**不自动屏蔽本信号**，允许信号嵌套重入（危险）
    - SA_RESETHAND：信号处理调用完成后重置为 SIG_DFL，模拟旧 signal 不可靠行为

    ``` c
    struct sigaction {
      void (*sa_handler)(int);          // 传统信号处理函数，同signal的handler
      void (*sa_sigaction)(int, siginfo_t *, void *); // 扩展信号处理，带详细信息
      sigset_t sa_mask;                 // 信号处理函数执行期间，需要阻塞的信号集
      int sa_flags;                     // 标志位
      void (*sa_restorer)(void);        // 保留，用户不要使用
    };
    ```
- 阻塞信号集sigset_t设置：
  - how 参数三种选项
    1.  `SIG_BLOCK`：新掩码 = 旧掩码 \| set，**追加阻塞**
    2.  `SIG_UNBLOCK`：新掩码 = 旧掩码 & \~set，**解除部分信号阻塞**
    3.  `SIG_SETMASK`：新掩码直接等于 set，**完全替换掩码**
        \`\`\` c
        int sigemptyset(sigset_t *set); // 清空信号集，全部信号置0
        int sigfillset(sigset_t *set); // 将所有信号加入信号集
        int sigaddset(sigset_t *set, int signum); // 添加单个信号到集合
        int sigdelset(sigset_t *set, int signum); // 从集合删除某个信号
        int sigismember(const sigset_t \*set, int signum); // 判断信号是否在集合中

int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

    - 未决信号集查看：
        - 信号如果被阻塞就进入未决状态进入未决队列（未决一个信号只有一次）
        - SIGKILL、SIGSTOP 不能被阻塞，不会进入未决
    ``` c
    int sigpending(sigset_t *set);

- 信号处理程序并非线程安全，需要线程安全保护
  - 只使用异步信号安全函数
  - 使用`volatile sig_atomic_t`类型全局变量
  - 访问共享全局变量的临界区，用`sigprocmask`把对应信号 block，临时屏蔽信号递送
  - 逻辑尽量极简
    \### 多线程下的信号处理
- 进程收到信号，如果信号被当前线程掩码阻塞，内核会把信号递交给**本进程内某个没有阻塞该信号的线程**
- 所有线程共享同一个信号处理函数
- 应该定义一个专门的线程处理信号，在创建子线程之前调用pthread_sigmask屏蔽信号，然后信号处理线程调用sigwait处理信号（不需要信号处理函数了），也可以通过pthread_kill将信号发给指定线程处理
- pthread_sigmask：
  - `how`
    - `SIG_BLOCK`：把 set 中的信号**添加**到当前掩码（阻塞这些信号）
    - `SIG_UNBLOCK`：把 set 中的信号**从掩码移除**（解除阻塞）
    - `SIG_SETMASK`：直接把线程掩码设置为 set
  - `set`：要操作的信号集合；传`NULL`代表不修改。
  - `oldset`：输出，保存旧的掩码；传`NULL`不保存。

  ``` c
  int pthread_sigmask(int how, const sigset_t *set, sigset_t *oldset);
  ```
- sigwait：
  - 同步等待一组信号，阻塞调用线程，直到 set 中任意一个信号到来；拿到信号编号存入 sig

  ``` c
  int sigwait(const sigset_t *restrict set, int *restrict sig);
  ```
- pthread_kill：

``` c
int pthread_kill(pthread_t thread, int sig);
```

### 可重入函数：

- 函数可以被中断，中断回来再次调用该函数，运行结果依然正确，基本都是异步安全
- 不使用全局、静态变量
- 不持有全局锁
- 只用传入参数、栈上局部变量
  \### 具体信号
- 网络编程相关信号
  - SIGHUP：挂断信号发送，比如终端会话断开、重载配置文件
  - ==SIGPIPE==：管道断裂，比如管道对端关闭、socket对端关闭，这里有大坑默认是终止程序，必须处理
  - SIGURG：带外数据，除了[IO多路复用异常事件](../6.%20网络编程/6.2%20网络编程.md#fd事件集合)以外，内核也会发送这个信号给socket所属进程
- 具体信号表（序号根据优先级排列）

| 信号宏 | 编号 | 说明 | 默认动作 | 能否捕获 / 忽略 |
|----|---:|----|----|----|
| SIGHUP | 1 | 终端挂断；守护进程常用做 reload 配置 | 终止 | ✅ |
| SIGINT | 2 | Ctrl+C，键盘中断 | 终止 | ✅ |
| SIGQUIT | 3 | Ctrl+，退出，生成 core dump | 终止 + core | ✅ |
| SIGILL | 4 | 非法指令 | 终止 + core | ✅ |
| SIGTRAP | 5 | 调试断点 trap | 终止 + core | ✅ |
| SIGABRT | 6 | abort () 触发异常终止 | 终止 + core | ✅ |
| SIGBUS | 7 | 总线错误，非法内存对齐访问 | 终止 + core | ✅ |
| SIGFPE | 8 | 浮点运算异常（除 0） | 终止 + core | ✅ |
| SIGKILL | 9 | 强制杀死进程 | 终止 | ❌不能捕获、不能忽略、不能阻塞 |
| SIGUSR1 | 10 | 用户自定义信号 1 | 终止 | ✅ |
| SIGSEGV | 11 | 段错误，访问非法内存地址 | 终止 + core | ✅ |
| SIGUSR2 | 12 | 用户自定义信号 2 | 终止 | ✅ |
| SIGPIPE | 13 | 向已经关闭的管道 /socket 写数据 | 终止 | ✅（服务器必须忽略） |
| SIGALRM | 14 | alarm () 定时器超时 | 终止 | ✅ |
| SIGTERM | 15 | kill 默认信号，优雅终止 | 终止 | ✅（可做资源清理） |
| SIGSTKFLT | 16 | 栈故障 | 终止 | ✅ |
| SIGCHLD | 17 | 子进程状态改变 (子退出 / 停止) | 忽略 | ✅ |
| SIGCONT | 18 | 继续运行被暂停进程 | 继续运行 | ❌不能忽略，可以捕获 |
| SIGSTOP | 19 | 暂停进程，挂起 | 暂停 | ❌不能捕获、不能忽略 |
| SIGTSTP | 20 | Ctrl+Z，终端挂起 | 暂停 | ✅ |
| SIGTTIN | 21 | 后台进程读终端 | 暂停 | ✅ |
| SIGTTOU | 22 | 后台进程写终端 | 暂停 | ✅ |
| SIGURG | 23 | socket 紧急 OOB 带外数据 | 忽略 | ✅ |
| SIGXCPU | 24 | CPU 时间超出限制 | 终止 | ✅ |
| SIGXFSZ | 25 | 文件大小超出限制 | 终止 | ✅ |
| SIGVTALRM | 26 | 虚拟定时器到期 | 终止 | ✅ |
| SIGPROF | 27 | 剖析定时器到期 | 终止 | ✅ |
| SIGWINCH | 28 | 终端窗口大小变化 | 忽略 | ✅ |
| SIGIO | 29 | 异步 IO 事件通知 | 终止 | ✅ |
| SIGPWR | 30 | 电源故障 | 终止 | ✅ |
| SIGSYS | 31 | 非法系统调用 | 终止 + core | ✅ |

### 统一事件源：

- 把异步信号转化成 IO 事件，全部事件（socket 网络、信号、定时器）全部交给 epoll_wait 主循环同步处理
- pipe 管道实现统一事件源：管道读端 fd 注册进 epoll，信号处理函数中**只做一件事：把信号编号 write/send 写到管道写端**
- 现代替代方案 signalfd：Linux 特有，直接把信号变成 fd，不需要手动 pipe，`int signalfd(int fd, const sigset_t *mask, int flags);`
  \## 非本地跳转
- 由c运行时库实现，应用层形式，不按正常流程运行，可以一口气跳出多层函数调用
- setjmp(jmp_buf env)：保存上下文，设置跳转点
- longjmp(env, val)：直接跳转到跳转点
- 信号安全版本：sigsetjmp / siglongjmp
  \# Linux的实现：
  \#todo
  \## 进程
  \## 虚拟内存
- 虚拟地址空间：VMA+4级页表
- 物理页分配：slub分配器+伙伴系统
- 内核内存分配：vmalloc
- Swap+Dual‑Clock 双时钟 LRU
- 写时复制
- PageCache页面缓存
  \## 并发
- 线程TCB：不管进程还是线程，内核统一使用`task_struct`
  - 创建普通进程：clone，不设置共享标志
  - 创建线程：clone (CLONE_VM\|CLONE_FILES\|CLONE_SIGHAND)
- 锁：futex
  \## 附录
- 程序地址空间（进程+线程+内核）
