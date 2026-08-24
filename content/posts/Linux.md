---
title: "计算机基础系列-Linux基础"
slug: "linux"
date: 2026-08-21T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["项目开发", "Linux", "c++"]
categories: ["技术笔记"]
summary: "Linux基础内容，包括目录结构、命令、文件权限、网络等。"
toc: true
comments: true
description: "Linux基础"
---
# Linux与UNIX

- UNIX：贝尔实验室，闭源商业系统，**POSIX 标准**：UNIX 制定的接口规范，统一系统调用，AIX、HP‑UX、Solaris
- Linux：Linus Torvalds，内核 + GNU 工具库 = Linux 发行版，开源免费，遵循POSIX 标准，CentOS/RHEL、Debian、Ubuntu、Fedora
- Linux与UNIX都符合POSIX，命令、行为高度相似，Linux 是**类 UNIX 系统**
  \# 远程登录
- SSH：端口默认 22，加密传输，C/S 架构，`sshd`服务端，`ssh`客户端
- 三方工具：vscode、xshell
- 服务端（远程linux机器）：

``` bash
# 查看sshd状态
systemctl status sshd
# 启动/开机自启
systemctl start sshd
systemctl enable sshd
```

- 客户端（本地机器）：

``` bash
# 基础语法
ssh 用户名@IP
# 指定端口（非22端口）
ssh -p 2222 user@192.168.1.100
```

- 远程拷贝：

``` bash
# 本地→远程
scp local.txt user@ip:/home/user/
# 远程→本地
scp user@ip:/home/user/file ./
# 指定端口
scp -P 2222 file user@ip:/tmp
```

# 目录结构

- 核心理念：一切皆文件，单根树结构，遵循 FHS（文件系统层次标准）
  \## 目录结构：
  - `/bin`：普通用户基础可执行命令，ls、cp、mv，系统启动就需要的命令
  - `/sbin`：系统管理员命令，root 使用，ifconfig、fdisk
  - `/boot`：系统启动相关，内核、grub 引导程序，不要乱改
  - `/dev`：设备文件，硬件映射，`/dev/sda`硬盘，`/dev/null`黑洞设备
  - `/etc`：**系统配置文件目录**，几乎所有服务配置都在这里，最高频
  - `/home`：普通用户家目录，`/home/username`，用户数据存放
  - `/root`：root 超级用户家目录，不在 /home 下
  - `/lib /lib64`：系统库文件，程序依赖的动态库
  - `/mnt`：临时挂载点，手动挂载 U 盘、硬盘
  - `/media`：自动挂载点，光盘、U 盘自动挂载到此
  - `/opt`：第三方大型软件，安装商业 / 第三方程序
  - `/proc`：**虚拟文件系统**，内存映射，不占磁盘；查看内核、进程信息，`/proc/cpuinfo`
  - `/sys`：虚拟文件系统，硬件内核参数，sysfs
  - `/tmp`：临时文件目录，所有用户可读写，重启会清空
  - `/usr`：Unix 系统资源，庞大应用目录，安装大部分软件程序
    - `/usr/bin`：用户应用程序命令
    - `/usr/sbin`：管理员应用命令
  - `/var`：**可变数据**，日志`/var/log`、缓存、数据库、邮件，会持续变大
    \## 高频目录：
  - `/home`：普通用户根目录
  - `/bin`：普通用户基础可执行命令
  - `/usr/bin`：用户应用程序命令
  - `/opt`：第三方大型软件
  - `/etc`
    - `/etc/passwd` 用户信息
    - `/etc/group` 用户组
    - `/etc/shadow` 用户密码
    - `/etc/sysconfig` 系统服务配置
  - `/var/log` 系统日志，排查故障第一站
  - `/usr/local` 编译源码软件默认安装位置
    \# 用户管理
- 相关配置文件：`/etc/passwd`、`/etc/shadow`、`/etc/group`、`/etc/gshadow`
- 用户登录注销

``` bash
# 登录
login
# 注销当前shell会话
logout
# 查看当前登录系统所有用户
who
# 查看登录用户 + 正在执行操作
w
```

- 添加用户
  - shell 设为`/sbin/nologin`，将禁止该账号 ssh 登录

  ``` bash
  # 创建普通用户，自动创建家目录、同名组
  useradd username
  # 指定家目录、shell
  useradd -m -s /bin/bash username
  ```
- 删除用户

``` bash
# 删除用户，保留家目录
userdel username
# -r 删除用户+家目录，彻底清理
userdel -r username
```

- 修改密码

``` cpp
# root可以改任意用户密码
passwd username
# 普通用户只能改自己密码
passwd
```

- 查询用户信息
  - `/etc/passwd`字段：`用户名:密码占位符:UID:GID:注释:家目录:登录shell`
  - UID：root=0；普通用户一般≥1000

  ``` bash
  id username          # 用户uid、gid、所属组，高频
  whoami              # 当前我是谁
  cat /etc/passwd     # 全部用户列表
  ```
- 切换用户
  - `sudo`：以管理员身份执行单条命令，无需登录 root

  ``` bash
  # 切换用户，不加载目标用户完整环境变量
  su username
  # - 完全切换，加载家目录环境，**强烈推荐**
  su - username
  # root切普通用户无需密码；普通切root要root密码
  ```
- 用户组操作

``` bash
groupadd groupname        # 创建组
groupdel groupname        # 删除组
usermod -g 主组 username  # 修改用户主组
usermod -aG 附属组 username # 添加附属组，-a不能丢，否则覆盖原有附属组
groups username           # 查看用户所属组
cat /etc/group            # 查看全部组信息
```

# 实用指令

## 指定运行级别

    - 0：关机
    - 1：单用户（找回 root 密码用）
    - 2：多用户无网络
    - 3：**多用户命令行（生产常用）**
    - 4：保留
    - 5：图形界面
    - 6：重启

``` bash
# 查看当前target
systemctl get-default
# 设置开机命令行模式
systemctl set-default multi-user.target
# 设置开机图形界面
systemctl set-default graphical.target
# 临时切换（不修改开机）
init 0   #关机
init 3   #命令行
init 5   #图形
init 6   #重启
```

## 找回 root 密码

    1. 开机 GRUB 界面按`e`进入编辑
    2. 找到 linux 行，修改：把`ro`改为`rw`，末尾添加 `init=/bin/bash`
    3. `Ctrl+x`进入单用户 shell
    4. `passwd root`重置密码
    5. `exec /sbin/init`正常启动系统

## 帮助文档

``` bash
man 命令        #查看完整手册，q退出
help 内置命令    #shell内置命令帮助，如 cd help
命令 -h / --help #简洁帮助输出，高频
info 命令       #详细文档，少用
```

## 文件目录

``` bash
pwd                 #显示当前绝对路径
ls [-l -a -h]       #-l详细信息，-a显示隐藏，-h人性化大小
cd 路径             #cd ~回家目录；cd -回到上一次目录
mkdir [-p] dir      #-p递归创建多级目录
rmdir 空目录        #仅删空目录
rm [-r -f] 文件/目录 #-r递归，-f强制不提示，慎用 rm -rf
cp [-r] 源 目标     #-r复制目录
mv 源 目标          #移动/重命名
touch 文件          #创建空文件
cat [-n] 文件       #一次性读取小文件，-n行号
more 文件           #分页浏览，空格翻页 q退出
less 文件           #大文件推荐，可上下滚动，q退出
head [-n 5] 文件    #看前5行，默认10行
tail [-n 5] 文件    #看末尾5行；tail -f 实时追踪日志输出⭐高频
>                   #输出重定向覆盖；
>>                  #输出重定向追加;
```

## 时间日期

``` bash
date                    #显示当前时间
date +"%Y-%m-%d %H:%M:%S" #格式化输出
date -s "2026‑08‑21 12:00:00" #设置系统时间(root)
cal                     #输出当月日历
cal 2026                #输出全年日历
```

## 搜索查找

- 磁盘遍历

``` bash
find 搜索路径 [选项]
find /home -name "*.txt"    #按文件名
find / -size +100M          #大于100M文件；+大于 -小于 =等于
find /root -user root       #按所属用户
```

- 数据库索引

``` bash
locate xxx.txt
updatedb        #更新索引库，新建文件需要更新才能搜到
```

- 关键词过滤

``` bash
grep [-n -i] "关键词" 文件
#-n显示行号；-i忽略大小写
#管道组合使用
cat test.txt | grep error
```

## 压缩解压

- gzip

``` bash
gzip file.txt      #生成file.txt.gz，原文件消失
gunzip file.txt.gz #解压
```

- zip

``` bash
zip -r out.zip dir/     #‑r压缩目录
unzip out.zip -d /tmp   #解压到指定‑d路径
```

- tar

``` bash
#常用参数组合：‑zcvf压缩；‑zxvf解压
#-z gzip压缩；-c打包；‑v显示过程；‑f指定包名；‑x解压
tar -zcvf test.tar.gz dir1/ file1.txt  #打包压缩
tar -zxvf test.tar.gz                  #解压到当前
tar -zxvf test.tar.gz -C /tmp          #解压到指定目录‑C
```

# 组和权限

## 组管理

- 相关配置文件：`/etc/group`组信息，`/etc/passwd`用户，`/etc/shadow`密码
- 每个用户**必有一个主组（初始组）**，可拥有多个附属组
- 文件 / 目录拥有：**所有者 (用户)、所属组、其他用户**
- 组操作

``` bash
groupadd 组名                # 创建组
groupdel 组名                # 删除组（不能有用户把它当主组）
groupmod -n 新组名 旧组名    # 修改组名称
```

- 用户组操作
  参见[用户管理](#用户管理)
- 文件目录组查看

``` bash
ls -l 文件/目录
# 输出示例：-rwxr-xr--  1 root dev  123  Aug 21 test.txt
# 权限位｜硬链接数｜所有者｜所属组｜大小｜时间｜文件名
```

- 文件目录组修改

``` bash
# 修改所有者
chown 新所有者 文件/目录
# 同时修改所有者+所属组
chown 所有者:组名 文件/目录
# -R 递归，目录连同内部全部文件，高频！
chown -R user1:group1 ./mydir

# 只修改所属组
chgrp 新组名 文件/目录
chgrp -R group2 ./mydir
```

## 权限管理

- 权限位共 9 位，分为 3 组，每组 3 位：`所有者权限 | 所属组权限 | 其他用户权限`
- 权限位分别是：rwx，r读，w写，x执行，有权限就是对应位置为1，无权限为0，可以将3位二进制转十进制来表示对应权限
- 目录必须有x权限才能访问内部
- 数字模式

``` bash
chmod 754 test.txt
# -R递归修改目录下全部内容
chmod -R 700 ./mydir
```

- 符号模式（u = 所有者 g = 所属组 o = 其他 a = 全部）

``` bash
chmod u+x test.sh          # 给所有者增加执行权限
chmod g-w test.txt         # 移除组写权限
chmod a=rwx test.txt       # 全部设置rwx
chmod u=rwx,g=rx,o=r test.txt
```

# 定时任务

## crond周期任务

- 配置文件
  - 用户任务：`/var/spool/cron/用户名`
  - 系统全局定时：`/etc/crontab`
  - 系统周期目录：
    `/etc/cron.hourly`每小时
    `/etc/cron.daily`每日
    `/etc/cron.weekly`每周
    `/etc/cron.monthly`每月
- 打开服务

``` bash
systemctl status crond     #查看状态
systemctl start crond
systemctl enable crond     #开机自启
```

- crontab操作

``` bash
crontab -e        #编辑定时任务（编辑当前用户任务）
crontab -l        #查看当前用户定时任务
crontab -r        #删除当前用户全部定时任务
crontab -u root -l #查看指定用户任务，root可用
```

- 定时任务文件格式
  - `*`：每，全部时间
  - `,`：多个时间，逗号分隔
  - `-`：范围
  - `/n`：每隔 n 单位

  ``` bash
  分 时 日 月 周 要执行的命令
  # 每1分钟输出时间到文件
  */1 * * * * date >> /tmp/time.log
  # 每天凌晨2点执行脚本
  0 2 * * * /root/backup.sh
  # 每周六23点30分执行
  30 23 * * 6 /root/clean.sh
  # 每天2、4、6点执行
  0 2,4,6 * * * df -h >> /tmp/disk.log
  # 8‑12点，每2小时执行一次
  0 8‑12/2 * * * echo hello
  ```

  ## at单次任务
- 配置文件
  - `/etc/at.allow`：允许使用 at 的用户（优先）
  - `/etc/at.deny`：禁止使用 at 的用户
- 打开服务

``` bash
systemctl status atd
systemctl start atd
systemctl enable atd
```

- at操作

``` bash
at 时间       #进入at交互输入命令，Ctrl+D结束输入
atq           #查看待执行at任务列表
atrm 任务编号  #删除指定at任务
```

- 时间格式

``` bash
at 14:30               #今天14:30执行
at 14:30 tomorrow      #明天14:30
at now + 5 minutes     #5分钟之后
at now + 2 hours       #2小时后
at 23:00 2026‑08‑25    #指定具体日期时间
```

# 磁盘分区挂载

- 磁盘在`/dev`下，分区是磁盘划分，挂载是把分区关联到目录
  \## 磁盘命名规则
- MBR：最多 4 个主分区；可以 3 主 + 1 扩展，扩展下多个逻辑分区。
- GPT：支持更多分区、大磁盘，无主 / 逻辑区分。
- SATA/SCSI 硬盘：`/dev/sda`第一块盘，`/dev/sdb`第二块盘
  - `sda1`：第一块盘第 1 个主分区；`sda5`第一个逻辑分区
- NVME 固态：`/dev/nvme0n1`，分区：`nvme0n1p1`
- IDE 老硬盘：`/dev/hda`
  \## 分区
- fdisk（MBR ，小于 2T）

``` bash
fdisk /dev/sda   #对sda磁盘进行分区操作
#交互内常用指令
m    #帮助
p    #打印分区表
n    #新建分区
d    #删除分区
w    #保存退出，生效
q    #不保存退出
```

- parted（GPT，大于 2T）

``` bash
parted /dev/sdb
```

- 格式化（生成文件系统）

``` bash
mkfs -t xfs /dev/sdb1   #格式化为xfs(CentOS7默认)
mkfs -t ext4 /dev/sdb2  #ext4
```

## 挂载

- 一般挂载到`mnt`目录下
- 临时挂载

``` bash
#语法 mount 设备 挂载点目录
mount /dev/sdb1 /mnt/data
```

- 卸载

``` bash
umount /dev/sdb1
#或者 umount 挂载点
umount /mnt/data
```

- 永久挂载
- 修改`/etc/fstab`，字段格式如下

``` bash
设备 挂载点 文件系统类型 挂载选项 dump（0不备份） fsck（0不自检）
/dev/sdb1 /mnt/data xfs defaults 0 0
```

## 磁盘情况查询

``` bash
df -h               #查看已挂载分区使用情况，‑h人性化单位⭐高频
df -i               #查看inode使用量（inode满磁盘也写不进）

du [-sh] 目录
du -sh /*           #统计各个目录总大小，排查磁盘爆满⭐高频
#‑s汇总，‑h人性化，‑a显示所有文件

lsblk               #列出块设备，磁盘+分区树，清晰直观
blkid               #查看分区UUID、文件系统类型
fdisk -l            #查看磁盘整体分区信息
```

## swap交换分区设置

``` bash
#查看swap
free -h
#临时启用swap分区
swapon /dev/sdb3
#关闭swap
swapoff /dev/sdb3
#fstab添加永久swap：/dev/sdb3 swap swap defaults 0 0
```

# 网络配置

- CentOS7+ 使用 `NetworkManager` 服务；传统 `network` 逐步废弃。
- 网卡命名示例：`ens33`、`eth0`、`enp0s3`
  \## 网卡配置
- 配置文件：`/etc/sysconfig/network-scripts/ifcfg-ens33`

``` ini
BOOTPROTO=dhcp|static   # dhcp自动获取；static静态IP
ONBOOT=yes              #开机启用网卡⭐必改
IPADDR=192.168.1.100    #静态IP
NETMASK=255.255.255.0   #子网掩码
GATEWAY=192.168.1.1     #网关
DNS1=223.5.5.5          #DNS服务器
```

- 重启网卡命令

``` bash
# NetworkManager方式（推荐）
nmcli connection reload
nmcli connection up ens33

# 旧版命令
systemctl restart network
```

## 主机名配置

- 配置文件：`/etc/hostname`、`/etc/hosts`

``` bash
hostname #查看主机名
hostnamectl set-hostname server01 #永久修改主机名
```

- hosts主机名映射文件

``` txt
192.168.1.100 server01
```

## 网络测试命令

- ping 测试连通性

``` bash
ping 192.168.1.1
ping baidu.com
ping -c 4 baidu.com      #只发4个包后自动结束
```

- 查看网卡信息

``` bash
ip addr                  #替代旧命令ifconfig（需要net-tools包）
ifconfig                 #需要安装 yum install net‑tools
```

- 查看路由

``` bash
ip route
route -n                 #路由表，‑n不解析域名，更快
```

- 端口连通性

``` bash
telnet IP 端口           #缺点：只支持tcp
nc -zv IP 端口           #nc，推荐，tcp/udp均可测试
```

- 追踪数据包

``` bash
traceroute IP
tracepath IP
```

- 排查服务端口

``` bash
ss -tulnp                #替代 netstat；‑t tcp ‑u udp ‑l监听 ‑n数字 ‑p进程
netstat -tulnp           #需要安装net‑tools
```

- DNS解析

``` bash
nslookup baidu.com
dig baidu.com
```

- 网络统计

``` bash
sar -n DEV 1             #每秒输出网卡流量
```

## 网络管理工具nmcli

``` bash
nmcli device status                #查看网卡设备状态
nmcli connection show              #查看连接配置
```

## 防火墙

``` bash
# 查看状态
systemctl status firewalld
# 开启防火墙
systemctl start firewalld
# 开机自启
systemctl enable firewalld
# 关闭防火墙（生产不建议）
systemctl stop firewalld
# 禁止开机启动
systemctl disable firewalld

# 查看已放行规则
firewall-cmd --list-all
# 删除端口（关闭某端口）
firewall-cmd --permanent --remove-port=80/tcp
# 放行端口范围
firewall-cmd --permanent --add-port=10000‑10010/tcp

# 查看支持的服务列表
firewall-cmd --get-services
# 放行指定服务
firewall-cmd --permanent --add-service=ssh

# 重载配置，永久规则才会生效⭐必须执行
firewall-cmd --reload
```

## 动态网络监控

- TCP状态：
  - `LISTEN`：监听，等待外部连接
  - `ESTABLISHED`：**已建立连接，正在通信**
  - `TIME_WAIT`：连接主动关闭后残留状态，短暂等待
  - `CLOSE_WAIT`：被动关闭，程序没正确关闭 socket，大量出现代表程序 bug

  ``` bash
  # 查看所有TCP/UDP监听端口，显示进程，最常用
  netstat -tulnp
  # 持续查看网络连接（静态快照，不是动态刷新）
  ss -tulnp
  # 查看全部连接（已建立、TIME_WAIT等）
  netstat -anp
  # 只看tcp全部连接
  netstat -antp
  # 过滤某个端口，例如22
  netstat -tulnp | grep 22
  # 过滤进程名
  netstat -anp | grep nginx
  # 查看路由表
  netstat -rn
  ```

  # 进程管理

  ## 进程信息
- ps -aux 关键字段：USER 用户、PID、% CPU 占用、% MEM 内存、STAT 进程状态、COMMAND 命令
- STAT 常用标识：`R`运行；`S`休眠；`Z`僵尸进程；`T`停止。

``` bash
ps -aux          # 系统全部进程⭐高频
# -a所有用户进程，-u显示用户，-x无终端进程
ps -ef           # 另一种格式，显示PPID父进程号

# 过滤进程，管道+grep
ps -aux | grep nginx
```

## 终止进程

- 信号 15：正常结束；信号 9：强制杀死。尽量优先 kill，不要直接‑9，容易损坏数据。

``` bash
kill PID                # 默认信号15，优雅关闭，允许清理资源
kill -9 PID             # 强制杀死，粗暴，慎用！⭐
killall 进程名           # 根据进程名批量杀进程
pkill 进程名
```

## 进程树

``` bash
pstree            # 进程树，显示父子关系
pstree -p         # 同时显示PID号⭐常用
pstree -u         # 显示进程所属用户
```

## 服务管理

``` bash
systemctl list-unit-files --type=service   #列出所有服务

systemctl status sshd       #查看服务状态
systemctl start sshd        #启动
systemctl stop sshd         #停止
systemctl restart sshd      #重启
systemctl enable sshd       #开机自启
systemctl disable sshd      #关闭开机自启
systemctl is‑active sshd    #查看是否正在运行
systemctl is‑enabled sshd   #查看是否开机自启
```

## 动态进程监控

- load average（系统 1 分钟、5 分钟、15 分钟负载；越接近 CPU 核数代表压力高）、% Cpu (s) CPU 占用、Mem 内存、Swap 交换分区。

``` bash
top
top -d 2 #每2秒刷新一次，默认3秒
htop #增强版top，需安装，交互更友好
```

- `P`：按 CPU 排序
- `M`：按内存排序
- `q`：退出
- `k`：输入 PID 杀死进程
- `1`：展开多核 CPU
  \# rpm与yum
- rpm：底层包管理工具，处理`.rpm`软件包；**不自动解决依赖**
- yum（centos7）：rpm 前端工具，自动下载、解决依赖
  \## rpm
- rpm 卸载不会自动处理依赖，`--nodeps`尽量不要用

``` bash
# 安装本地rpm包
rpm -ivh xxx.rpm

# 升级
rpm -Uvh xxx.rpm

# 查询是否安装
rpm -q nginx

# 查询全部已安装包
rpm -qa

# 过滤查询
rpm -qa | grep nginx

# 查看软件信息
rpm -qi nginx

# 查看该包安装哪些文件
rpm -ql nginx

# 卸载软件
rpm -e nginx
# 卸载遇到依赖报错，强制卸载（不推荐，容易破坏系统）
rpm -e --nodeps nginx
```

## yum

- yum 源：软件仓库，配置文件目录`/etc/yum.repos.d/*.repo`
- yum 缓存：/var/cache/yum/

``` bash
# 查询可安装软件
yum list

# 查询已安装
yum list installed

# 搜索软件
yum search nginx

# 安装软件
yum install -y nginx

# 升级软件
yum update nginx
# 全部系统升级
yum update

# 卸载软件
yum remove -y nginx

# 查看软件信息
yum info nginx

# 清理缓存
yum clean all
# 生成新缓存
yum makecache

# 下载rpm包到本地，不安装
yum install -y --downloadonly --downloaddir=/tmp nginx
```

# shell编程

- shell 脚本：`.sh`后缀；第一行 `#!/bin/bash` 指定解释器
  \## 脚本执行方式

``` bash
#方式1：bash 脚本名，不需要执行权限，新开子shell执行
bash test.sh
sh test.sh

#方式2：给执行权限，./执行（必须加./，代表当前路径）
chmod +x test.sh
./test.sh

#方式3：source / . 脚本，**在当前shell执行，不创建子进程**，变量会留在当前终端
source test.sh
. test.sh
```

## 输入输出

``` bash
#输出
echo "hello"
echo -n "不换行输出"

#读取键盘输入，存入变量
read -p "请输入数字：" num
echo $num
```

## 环境变量

- 配置文件：`~/.bashrc`、`/etc/profile`

``` bash
#导出为环境变量
export MYENV="hello"

#查看全部环境变量
env
printenv

#系统自带环境变量
$HOME 家目录
$PATH 命令搜索路径
$USER 当前用户
$PWD 当前路径
```

## 变量

- 定义：`变量名=值`,等号两边不能空格
- 只读变量：`readonly num=10`
- 使用：`${变量名}`
- 删除：`unset name`
- 字符串：双引号`""`解析变量；单引号`''`原样输出，不解析
- 系统内置变量：

``` bash
$?    #上一条命令执行返回值，0成功，非0失败⭐高频
$$    #当前shell脚本PID
$!    #后台运行最后一个进程PID
```

- 位置参数变量：(函数或命令时传入的参数)

``` bash
$0   #脚本本身名字
$1   #第1个参数
$2   #第2个参数
${10} #第10个参数，大括号不可少
$*   #全部参数，当做一个整体
$@   #全部参数，分开每个参数（循环优先用$@）
$#   #参数总个数
```

## 运算符

- shell 本身不支持数学运算，用 `$(( ))` 整数运算
- 只支持**整数运算**，小数用`bc`工具

``` bash
a=2
b=3

c=$((a+b))
echo $c

#支持 + - * / %
echo $(( 10*2 ))

#比较不要用数学> <，用test / [ ]
```

## 流程控制

- if判断
  - `-eq`等于；`-ne`不等于；`-gt`大于；`-ge`大于等于；`-lt`小于；`-le`小于等于
  - `-f 文件` 是否普通文件；`-d 目录` 是否目录；`-e` 文件是否存在
  - 字符串：`[ "$str1" = "$str2" ]`

  ``` bash
  if [ $a -gt $b ]
  then
  echo "a大于b"
  elif [ $a -eq $b ]
  then
  echo "a等于b"
  else
  echo "a小于b"
  fi
  ```
- for循环

``` bash
#写法1
for i in $@
do
  echo $i
done

#写法2 C语言风格
for ((i=1;i<=5;i++))
do
  echo $i
done
```

- while循环

``` bash
i=1
while [ $i -le 3 ]
do
  echo $i
  i=$((i+1))
done
```

- case分支

``` bash
case $1 in
"start")
  echo "启动"
;;
"stop")
  echo "停止"
;;
*)
  echo "其他参数"
;;
esac
```

## 函数

- shell 函数 return 只返回退出状态；想要返回数据用 echo 输出，用变量捕获

``` bash
add(){
  echo $(( $1 + $2 ))
}
res=$(add 10 20)
echo $res
```

# 日志管理

## 基础

- 日志存放主目录：**`/var/log`**
- 日志分类：系统内核日志、服务日志、安全登录日志、应用日志
- 日志常见后缀：`.log`普通日志；`.gz`轮替压缩旧日志
- 查看日志工具：`cat、more、less、head、tail -f、grep`
  \## 系统日志
- 部分应用（nginx、mysql）会把日志输出到自身配置指定目录，不一定在`/var/log`
- `/var/log/messages`：**CentOS 通用系统总日志**，大部分服务普通信息记录（RHEL/CentOS）
- `/var/log/secure`：⭐安全日志，ssh 登录、账号认证、密码修改，排查爆破登录重点
- `/var/log/cron`：crond 定时任务执行日志
- `/var/log/dmesg`：系统开机内核启动信息；也可用`dmesg`命令直接输出
- `/var/log/maillog`：邮件服务日志
- `/var/log/boot.log`：系统开机启动过程日志
  \## 日志管理
- 服务名：`rsyslog`
- 配置文件：`/etc/rsyslog.conf`，单独服务`/etc/rsyslog.d/xxx.conf`
- 配置规则格式：`设施.级别 日志存放路径`，级别表示此级别以及以上的log被记录
  - 设施 (facility)：`kern` 内核、`auth` 认证、`cron` 定时、`mail` 邮件、`user` 用户程序、`*`全部
  - 级别 (severity)：`debug`调试、`info`普通信息、`notice`、`warn`警告、`err`错误、`crit`严重、`alert`、`emerg`紧急、`*`全部
- 重启生效修改：`systemctl restart rsyslog`
  \## 日志轮替
- 日志不断增长，防止磁盘占满；做切割、改名、压缩、删除旧日志
- crond定时调用logrotate，不是服务
- 配置文件：`/etc/logrotate.conf`，单独服务`/etc/logrotate.d/xxx.conf`
- logrotate配置文件
  - `daily`：按天轮替；`weekly`周；`monthly`月
  - `rotate N`：保留 N 份历史旧日志
  - `compress`：gzip 压缩旧日志
  - `create`：轮替之后新建空日志文件
  - `dateext`：旧日志附加日期后缀，如`secure‑20260821.gz`
  - `size 100M`：达到大小就切割，不等待时间周期
  - `postrotate/endscript`：轮替完成后执行脚本

  ``` txt
  /var/log/secure {
    monthly
    rotate 4
    compress
    dateext
    create 0600 root root
  }
  ```
- 测试轮替

``` bash
# 模拟，不实际执行
logrotate -d /etc/logrotate.conf
# 强制立即执行轮替
logrotate -f /etc/logrotate.conf
```

# 备份恢复

## 简单备份方法

- 压缩打包

``` bash
tar -zcvf backup.tar.gz /home
```

- rsync远程备份

``` bash
rsync -av /home root@192.168.1.100:/backup/
```

## dump备份

- 安装：`yum install dump`，仅支持ext2/ext3/ext4，xfs 文件系统不支持 dump
- 备份级别：
  - 0 级别：**完整备份（全量备份）⭐必记**
  - 1‑9 级别：增量备份，只备份比上一级更新的文件。1 备份 0 之后变更，2 备份 1 之后变更。
- 常用参数：
  - `-0` \~ `-9`：备份级别
  - `-f`：指定备份输出文件（必须）
  - `-u`：备份成功后更新`/etc/dumpdates`，记录备份时间与级别，用于增量备份
  - `-j`：备份同时 bzip2 压缩
  - `-W`：查看分区 dump 备份记录
    \`\`\` bash
    \# 0级全量备份 /boot分区，备份到/backup/boot0.dump，更新记录
    dump -0u -f /backup/boot0.dump /boot

# 1级增量备份，只备份0级之后新增修改的文件

dump -1u -f /backup/boot1.dump /boot

# 查看备份记录

dump -W

    ## restore恢复
    - 用来还原 dump 生成的备份文件
    - 常用参数：
    - `-C`：对比校验，备份与当前文件对比，检查是否损坏
    - `-i`：交互模式，手动选择要恢复哪些文件⭐高频
    - `-t`：查看备份包内文件列表，不解压
    - `-r`：完整恢复整个分区（还原整个备份）
    - `-f` 指定 dump 备份文件
    ``` bash
    # 查看备份包里面有哪些文件
    restore -t -f /backup/boot0.dump

    # 校验备份文件和磁盘上文件差异
    restore -C -f /backup/boot0.dump

    # 完整恢复，把整个备份恢复到当前目录（注意当前目录！）
    restore -r -f /backup/boot0.dump

    # 交互模式，选择性恢复部分文件
    restore -i -f /backup/boot0.dump
    #交互内命令
    # ls 查看包内目录；add 文件名加入恢复列表；extract执行恢复；quit退出

# 可视化运维（不建议）

- ⚠️注意以下两者都以 root 权限运行，公网暴露端口有安全风险！！！
- Webmin：**通用系统级 web 管理面板**，侧重操作系统底层管理

``` bash
# 安装依赖
yum install -y wget perl
# 添加webmin源，安装
wget http://download.webmin.com/download/yum/webmin-1.990-1.noarch.rpm
rpm -ivh webmin-1.990-1.noarch.rpm

# 启动服务
systemctl start webmin
systemctl enable webmin

#防火墙放行10000端口
firewall-cmd --permanent --add-port=10000/tcp
firewall-cmd --reload
```

- BT（宝塔面板）：**网站建站运维面板**，侧重 LNMP/LAMP、站点、数据库、web 服务管理

``` bash
yum install -y wget
wget -O install.sh https://download.bt.cn/install/install_6.0.sh
sh install.sh

bt              #进入宝塔工具箱交互菜单
bt 1            #停止面板服务
bt 2            #启动面板
bt 5            #修改面板密码
bt 6            #修改面板用户名
bt 7            #修改面板端口
bt 14           #查看默认登录账号密码
bt 16           #修复面板
```
