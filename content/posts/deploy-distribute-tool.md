---
title: "C++工程常用工具链-部署发行篇"
slug: "deploy-distribut-tool"
date: 2026-08-24T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["工具", "c++", "部署", "发行"]
categories: ["技术笔记"]
summary: "一些C++工程中常用的部署发行工具链，包括docker、kubernetes、nginx、git、github-actions等。"
toc: true
comments: true
description: "C++工程常用工具链-部署发行篇"
---
# 版本管理：git

- 全流程是：新修改文件 → 工作区 →(add) 暂存区 →(commit) 本地仓库 →(push) 远程仓库
  \## 常用工作流

``` bash
git switch main
git pull origin main          # 更新主干
git switch -c feature/login   # 创建功能分支

# 编码修改
git add .
git commit -m "feat: 登录功能"
git push -u origin feature/login

# 去网页端提交PR/MR，合并到main（不是git原生命令，GitHub提供）：
# repo首页的Pull requests -> new pull request

# 合并完成后需要按需清理刚刚新增功能的分支
# 删除远程功能分支
git push origin --delete feature/login
# 删除本地分支
git branch -d feature/login
```

## 全局配置

- 配置用户名和邮箱

``` bash
# 全局配置（所有仓库生效）
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"

# 当前仓库单独配置（覆盖全局）
git config user.name "名字"
git config user.email "邮箱"

# 查看配置
git config --list
```

- 配置SSH配对
  - 使用以下代码生成密钥，进入`~/.ssh/`获取私钥`id_ed25519`和公钥`id_ed25519.pub`
  - 公钥内容复制到github，`ssh -T git@github.com`测试连通性
  - 连通后不需要密码就可以提交
  - 配置http/https代理对ssh协议完全无效

  ``` bash
  ssh-keygen -t ed25519（或者rsa） -C "你的github注册邮箱"
  ```
- 配置远程仓库：配置后远程仓库的分支也算你的本地分支

``` bash
# 查看远程仓库地址
git remote -v

# 添加远程仓库 origin（默认远程名称）
git remote add origin https://github.com/xxx/demo.git

# 修改远程地址
git remote set-url origin 新地址

# 删除远程关联
git remote remove origin
```

- 配置代理

``` bash
# http
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# socks5,推荐clash使用
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890

# 查看当前代理
git config --global --get http.proxy
git config --global --get https.proxy

# 查看全部global配置
git config --global --list

# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 仅当前仓库代理和取消：删除--global即可
```

## 初始化

- 初始化本地仓库

``` bash
# 在当前文件夹初始化git仓库
git init
```

- 拉取远程仓库

``` bash
# 克隆远程仓库到本地
git clone shturl.cc/NaD5kS3ptZvdtSSBvaCk
# 克隆并重命名本地文件夹
git clone https://github.com/xxx/demo.git my-project
```

## 文件提交

- 查看当前工作区状态

``` bash
# 查看文件状态（最常用）
git status

# 简洁版状态
git status -s
```

- 添加文件到暂存区

``` bash
# 添加单个文件到暂存区
git add test.js

# 添加多个文件
git add a.html b.css

# 添加当前目录所有新增/修改文件（不包含被删除文件）
git add .

# 添加所有变更（新增、修改、删除）
git add -A
```

- 提交到本地仓库

``` bash
# 提交暂存区内容，必须加注释
git commit -m "feat: 新增登录页面"

# 跳过add，直接提交已经追踪过的修改（新增文件无效）
git commit -am "fix: 修复按钮bug"
```

## .gitignore

- 告诉 Git：哪些文件 / 文件夹不需要追踪、不要提交到版本库
- 写一行就是一条忽略规则
- 文件已经被git追踪后，再写入gitignore无效，需要先移除追踪`git rm --cached 文件名`
- 常见模版：

``` txt
# 日志
*.log
# 文件夹
node_modules/
dist/
build/
# 系统文件
.DS_Store
# 环境配置
.env
```

## 查看提交日志（可以查提交id）

``` bash
# 查看提交日志
git log

# 简洁单行日志
git log --oneline

# 展示分支图形
git log --oneline --graph

# 查看最近n条记录
git log -3

# 查看某个文件的修改历史
git log test.js

# 查看某次提交详细改动
git show 提交id
```

## 分支操作

``` bash
# 查看本地分支
git branch

# 查看本地+远程分支
git branch -a

# 创建分支
git branch dev

# 创建并切换到新分支
git checkout dev
# 新版推荐
git switch dev
git switch -c dev  # 创建+切换

# 删除本地分支
git branch -d dev
# 强制删除（未合并分支）
git branch -D dev

# 重命名分支
git branch -m oldname newname
```

## 撤回

- 撤销工作区

``` bash
# git 2.23+ 推荐新命令
git restore test.js
```

- 撤销暂存区

``` bash
# 新版写法
git restore --staged test.js
```

- 撤销本地仓库

``` bash
# 软重置：退回上个提交，代码保留在工作区
git reset --soft HEAD~1

# 混合重置(默认)：退回上个提交，代码退回工作区
git reset HEAD~1

# 硬重置！谨慎！删除所有未提交改动，不可恢复
git reset --hard HEAD~1

# HEAD~1 = 上一个提交；HEAD~2 上两个
```

- 撤销远程仓库（推荐重新提交一个版本，因为可以保留历史）

``` bash
# 1. 查看提交记录，复制错误提交ID
git log --oneline

# 2. 反向撤销这条提交
git revert 提交哈希值

# 若出现冲突：手动解决冲突 → git add . → git revert --continue
# 想放弃本次撤销：git revert --abort

# 3. 正常推送到远程
git push origin main
```

- 撤销PR合并：GitHub/GitLab 网页上 PR 详情页一般直接有【Revert】按钮，点击自动生成撤销 PR，非常方便。
  \## 合并与冲突（强烈建议使用merge）
  提交顺序

``` bash
A---B---C (main)
     \
      D---E (feature)
```

- merge合并（所有改动一次性对比，分支分叉）

``` bash
A---B---C (main)
     \   \
      D---E---M (feature)

# 切换到feature分支
git switch feature

# 将main分支合并到feature
git merge main

# 出现冲突 → 解决冲突 → git add . → git commit
git push
```

- rebase合并（每个改动都重放一遍，单一分支）

``` bash
A---B---C (main)
          \
           D'---E' (feature)

# 切换到feature分支
git switch feature

git rebase main

# 某个提交冲突：解决 → git add . → git rebase --continue
# 不想继续：git rebase --abort
git push --force-with-lease origin feature
```

- 冲突处理：手动修改冲突代码，然后重新add
  \## 远程推拉
- ==注意拉取和推送的是远程仓库origin的main分支 -\> 本地当前分支哦==
- ==注意远程分支不能单独新建，用本地分支推送到自己命名的远程分支就自动新建好了==
- 拉取远程代码

``` bash
# 拉取远程代码（不合并）
git fetch origin

# 拉取+合并远程main分支（等价fetch + merge）
git pull origin main
```

- 推送本地分支

``` bash
# 推送本地分支到远程
git push origin main

# 首次推送建立追踪关系
git push -u origin dev

# 删除远程分支
git push origin --delete dev
```

## 对比差异

``` bash
# 对比工作区与暂存区
git diff

# 对比暂存区和本地仓库
git diff --cached

# 两个分支对比
git diff main dev

# 对比单个文件
git diff test.js
```

## 临时保存工作区

``` bash
# 储藏当前所有未提交改动
git stash

# 储藏并添加备注
git stash save "临时保存开发中代码"

# 查看储藏列表
git stash list

# 恢复最近一次储藏（储藏记录保留）
git stash apply

# 恢复并删除储藏记录
git stash pop

# 删除储藏
git stash drop stash@{0}
# 清空所有储藏
git stash clear
```

## 发布版本

``` bash
# 创建轻量标签
git tag v1.0.0

# 创建带注释标签（推荐发布版本使用）
git tag -a v1.0.0 -m "release v1.0.0"

# 查看标签
git tag

# 推送单个标签到远程
git push origin v1.0.0
# 推送所有标签
git push origin --tags

# 删除本地标签
git tag -d v1.0.0
# 删除远程标签
git push origin --delete v1.0.0
```

# 代码规范：clang-format

- 用于代码规范检查
- apt/brew等安装clang-format
- 项目根目录放 **`.clang-format`** 配置文件，内层覆盖外层
  \## 常用命令
- ==只格式化git已修改的源码==

``` bash
# 获取所有改动的cpp/h文件，批量格式化
git diff --name-only --cached | grep -E '\.(h|hpp|cpp|cc)$' | xargs clang-format -i
```

- 自动格式化代码：

``` bash
# 格式化单个文件
clang-format -i main.cpp

# 批量格式化所有 h/hpp/cpp
find . -type f \( -name "*.h" -o -name "*.hpp" -o -name "*.cpp" -o -name "*.cc" \) -exec clang-format -i {} +
```

- 检查哪个文件不规范：

``` bash
# 批量检查，如果有格式问题打印文件名
find . -type f \( -name "*.h" -o -name "*.hpp" -o -name "*.cpp" \) -exec clang-format --dry-run {} +
```

- 检查文件哪里不规范：

``` bash
clang-format --dry-run -Werror main.cpp
```

## 配置文件模板

``` bash
# 支持自动基于内置风格导出
# 可选风格：LLVM / Google / Chromium / Mozilla / WebKit
clang-format -style=Google -dump-config > .clang-format
```

# 单元测试：ctest

- cmake的单元测试模块
- 需要先在CMakeLists.txt中加入测试程序，可以结合[doctest](1.4%20开发与测试*.md#doctest)使用注册
  \## 注册测试

``` txt
cmake_minimum_required(VERSION 3.20)
project(demo)

enable_testing()  # ✅ 关键！开启测试支持，没有它 add_test 无效

# 编译测试可执行文件
add_executable(test_math tests/test_main.cpp)

# 注册测试
add_test(NAME math_test COMMAND test_math)
```

## 常用命令

``` bash
# 执行所有测试
ctest

# 列出所有测试，不运行
ctest -N

# 详细输出（看到程序stdout/stderr，最常用！不加这个失败只显示简短信息）
ctest -V
# 更详细
ctest -VV

# 只运行名字匹配的测试（正则）
ctest -R math

# 并行执行测试
ctest -j4

# 失败时停止
ctest --stop-on-failure
```

# 持续集成部署：GitHub Actions

- GitHub 内置免费的持续集成 / 持续部署工具（CI/CD）
- 用于代码规范检查、静态代码扫描、漏洞检测、单元测试、构建打包检测、生成打包结果、自动更新README、自动发布release、通知机器人、自动部署测试环境
- 注意：**GitHub 官方托管的运行器（GitHub-hosted）没有提供 CentOS 系统镜像**，在官方 Ubuntu 运行器中，通过 Docker 容器启动 CentOS 环境，所有编译、检查、测试步骤都在 CentOS 容器内执行
  \## 核心配置
- Workflow（工作流）：存放在仓库目录：`.github/workflows/xxx.yml`
- on：触发时机配置
- env：全局环境变量
- permissions：本工作流对整个仓库的临时访问权限
- Job（任务）：一个 workflow 里可以多个 job，可并行 / 串行执行。
- Step（步骤）：Job 里面一条条执行命令、脚本。
- Runner（运行器）：执行代码的虚拟机系统（GitHub 托管）
- action：实际的命令，分成uses和run，uses表示复用社区action，run表示直接执行的shell命令
  \## 内置和社区常用action
- GitHub 没有官方action支持配置gcc/cmake环境，自己用指令安装
- 拉取代码：

``` yaml
- uses: actions/checkout@v4
```

- 配置Node.js/npm/yarn/pnpm 环境：

``` yaml
- uses: actions/setup-node@v4
  with:
    node-version: 22
    cache: 'npm' # 自动缓存依赖
```

- 配置python环境：

``` yaml
- uses: actions/setup-python@v5
  with:
    python-version: "3.12"
```

- 配置java环境：

``` yaml
- uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: 'temurin'
```

- 配置golang环境：

``` yaml
- uses: actions/setup-go@v5
  with:
    go-version: '1.23'
```

- 上传build后文件：

``` yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
```

- 下载build后文件：

``` yaml
- uses: actions/download-artifact@v4
  with:
    name: build-output
    path: ./dist
```

- 缓存依赖：

``` yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

- 构建多架构 Docker 镜像：

``` yaml
- uses: docker/setup-buildx-action@v3
```

- 登录 Docker Hub / 容器镜像仓库：

``` yaml
- uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKER_USER }}
    password: ${{ secrets.DOCKER_PWD }}
```

- 构建并推送镜像：

``` yaml
- uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: username/demo:latest
```

- 自动创建 Release、上传二进制包：

``` yaml
- uses: softprops/action-gh-release@v2
  with:
    files: ./build/*.zip
```

- SSH远程服务器部署：

``` yaml
- uses: webfactory/ssh-agent@v0.9
  with:
    ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
```

## 触发事件（on配置）

- `push`：代码推送到分支
- `pull_request`：新建 / 更新 PR
- `schedule`：定时任务
- `workflow_dispatch`：手动点击按钮运行
- release、issue 创建
  \## 常用功能：自动发布
- 本地打标签推送

``` bash
# 创建版本标签 v1.0.0 严格以v开头！
git tag v1.0.0

# 查看本地所有标签
git tag

# 删除本地标签
git tag -d v1.0.0
# 删除远端错误标签
git push origin --delete v1.0.0

# 推送到github远程仓库
git push origin v1.0.0
```

- github actions配置好，会自动检查
- 自动在仓库 Releases 页面生成正式版本，并附带你的二进制压缩包
  \## 工程模板
- 配置secret变量：在仓库 `Settings → Secrets and variables → Actions` 中添加相应机密变量
- 配合ruleset门禁：在仓库`Settings → Rulesets → Require status checks to pass`中添加相应检查项
- 常用功能：

``` yaml
name: C++ Standard CI

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]       # v开头标签触发发布流程
  pull_request:
    branches: [ main ]

permissions:
  contents: write        # 发布 Release 需要写权限

env:
  BUILD_DIR: build
  BUILD_TYPE: Release
  CC: clang
  CXX: clang++

jobs:
  # 代码规范检查（同基础版）
  code-style-check:
    name: 代码规范校验
    runs-on: ubuntu-latest
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      - name: 安装 clang-format
        run: sudo apt update && sudo apt install -y clang-format
      - name: 格式检查
        run: |
          find . \
            -path ./${{ env.BUILD_DIR }} -prune -o \
            -path ./thirdparty -prune -o \
            -type f \( -name "*.h" -o -name "*.hpp" -o -name "*.cpp" -o -name "*.cc" \) \
            -exec clang-format --dry-run --Werror {} +

  # 新增：静态代码扫描
  static-analysis:
    name: 静态代码扫描
    runs-on: ubuntu-latest
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 安装 clang-tidy 与构建工具
        run: |
          sudo apt update
          sudo apt install -y clang clang-tidy cmake make

      - name: 生成编译命令数据库
        run: |
          cmake -B ${{ env.BUILD_DIR }} \
            -DCMAKE_BUILD_TYPE=${{ env.BUILD_TYPE }} \
            -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

      - name: 执行 clang-tidy 扫描
        run: |
          # 仅扫描 src 目录源码，排除第三方代码
          run-clang-tidy -p ${{ env.BUILD_DIR }} \
            -header-filter=^./src/.* \
            src/

  # 构建测试打包（同基础版）
  build-and-test:
    name: 构建与单元测试
    runs-on: ubuntu-latest
    needs: [ code-style-check, static-analysis ]
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      - name: 安装工具链
        run: sudo apt update && sudo apt install -y clang cmake make
      - name: CMake 配置
        run: cmake -B ${{ env.BUILD_DIR }} -DCMAKE_BUILD_TYPE=${{ env.BUILD_TYPE }}
      - name: 编译
        run: cmake --build ${{ env.BUILD_DIR }} -j$(nproc)
      - name: 单元测试
        working-directory: ${{ env.BUILD_DIR }}
        run: ctest -V --output-on-failure
      - name: 打包
        working-directory: ${{ env.BUILD_DIR }}
        run: cpack -G TGZ
      - name: 归档制品
        uses: actions/upload-artifact@v4
        with:
          name: cpp-release-package
          path: ${{ env.BUILD_DIR }}/*.tar.gz
          retention-days: 30

  # 新增：自动发布 Release（仅推送 v 开头标签时触发）
  publish-release:
    name: 自动发布版本
    runs-on: ubuntu-latest
    needs: build-and-test
    if: startsWith(github.ref, 'refs/tags/v')
    steps:
      - name: 下载构建制品
        uses: actions/download-artifact@v4
        with:
          name: cpp-release-package

      - name: 创建 GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true  # 自动根据提交生成更新日志
          files: ./*.tar.gz             # 上传二进制包
          body: |
            ## 版本说明
            CI 自动构建发布，基于 clang + Release 模式编译
            - 编译器：clang
            - 构建类型：Release
            - 全量单元测试校验通过
```

- 详细功能：

``` yaml
# C++ 项目全流程 CI/CD 工作流（CentOS 环境构建）
# 覆盖：代码规范检查、静态扫描、漏洞检测、单元测试、构建打包、制品归档、README更新、Release发布、机器人通知、测试环境部署
name: C++ Full CI/CD (CentOS)

# 触发条件
on:
  push:
    branches: [ main ]           # 主分支推送触发全流程
    tags: [ 'v*' ]               # 推送 v 开头标签（如 v1.0.0）触发发布流程
  pull_request:
    branches: [ main ]           # PR 提交触发质量校验流程

# 全局权限配置：遵循最小权限，发布、写代码时按需提升
permissions:
  contents: write                # 代码读写权限（更新README、发布Release需要）
  security-events: write         # 漏洞扫描结果写入权限（CodeQL需要）
  pull-requests: read            # PR信息读取权限

# 全局环境变量（统一配置，方便修改）
env:
  BUILD_DIR: build               # 构建目录
  CMAKE_BUILD_TYPE: Release      # 构建类型
  CC: clang                      # C编译器
  CXX: clang++                   # C++编译器

jobs:
  ###########################################################################
  # 阶段1：代码质量校验（CentOS 容器内执行，并行快速失败）
  ###########################################################################

  # 1.1 代码规范检查（clang-format）
  code-style-check:
    name: 代码规范检查
    runs-on: ubuntu-latest       # 宿主用官方Ubuntu，内部启动CentOS容器
    container: centos:stream9    # 所有 run 命令在 CentOS Stream 9 容器内执行
    steps:
      # 拉取仓库代码（自动挂载进容器）
      - name: 检出代码
        uses: actions/checkout@v4

      # 安装格式化工具（CentOS 使用 dnf 包管理器）
      # clang-format 包含在 clang-tools-extra 包中
      - name: 安装 clang-format
        run: |
          dnf install -y epel-release
          dnf install -y clang clang-tools-extra

      # 全量代码格式校验
      # 规则：读取根目录 .clang-format 配置，只检查不修改文件，格式不规范直接报错
      # 排除：构建目录、第三方依赖目录，不校验外部代码
      - name: 执行 clang-format 规范校验
        run: |
          find . \
            -path ./${{ env.BUILD_DIR }} -prune -o \
            -path ./thirdparty -prune -o \
            -type f \( -name "*.h" -o -name "*.hpp" -o -name "*.cpp" -o -name "*.cc" \) \
            -exec clang-format --dry-run --Werror {} +

  # 1.2 静态代码扫描（clang-tidy，检测内存泄漏、未初始化变量、逻辑bug等）
  static-code-scan:
    name: 静态代码扫描
    runs-on: ubuntu-latest
    container: centos:stream9
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      # 安装工具链：clang + clang-tidy + cmake + make
      - name: 安装静态扫描与构建工具
        run: |
          dnf install -y epel-release
          dnf install -y clang clang-tools-extra cmake make

      # CMake 配置：导出编译命令数据库（clang-tidy 必须依赖此文件解析代码）
      - name: 生成编译命令数据库
        run: |
          cmake -B ${{ env.BUILD_DIR }} \
            -DCMAKE_BUILD_TYPE=${{ env.CMAKE_BUILD_TYPE }} \
            -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

      # 执行静态扫描
      # 规则：读取根目录 .clang-tidy 配置，扫描 src 目录源码，发现问题直接报错阻断
      - name: 执行 clang-tidy 静态扫描
        run: |
          run-clang-tidy -p ${{ env.BUILD_DIR }} \
            -header-filter=^./src/.* \
            src/

  # 1.3 漏洞检测（GitHub CodeQL，官方原生代码安全扫描）
  # 说明：CodeQL 官方不推荐在自定义容器内运行，因此保留在原生 Ubuntu 执行，扫描结果与系统无关
  vulnerability-scan:
    name: 代码漏洞检测
    runs-on: ubuntu-latest
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      # 初始化 CodeQL 分析环境
      - name: 初始化 CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: cpp
          queries: security-and-quality  # 安全+质量规则集，覆盖高危漏洞

      # 自动构建项目（CodeQL 编译期插桩分析，必须执行真实构建）
      - name: 自动构建项目
        uses: github/codeql-action/autobuild@v3

      # 执行漏洞分析并上传结果到仓库 Security 页面
      - name: 执行漏洞分析
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:cpp"

  ###########################################################################
  # 阶段2：构建、单元测试、打包（CentOS 环境，依赖所有质量校验通过后执行）
  ###########################################################################
  build-test-package:
    name: 构建-测试-打包
    runs-on: ubuntu-latest
    container: centos:stream9
    needs: [ code-style-check, static-code-scan, vulnerability-scan ]
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      # 安装全套编译工具链
      - name: 安装构建工具链
        run: |
          dnf install -y epel-release
          dnf install -y clang clang-tools-extra cmake make tar gzip

      # CMake 工程配置
      - name: CMake 工程配置
        run: |
          cmake -B ${{ env.BUILD_DIR }} \
            -DCMAKE_BUILD_TYPE=${{ env.CMAKE_BUILD_TYPE }} \
            -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

      # 并行编译项目
      - name: 编译项目
        run: cmake --build ${{ env.BUILD_DIR }} -j$(nproc)

      # 运行单元测试（CTest）
      # -V 输出详细日志，失败时直接展示用例报错信息
      - name: 执行单元测试
        working-directory: ${{ env.BUILD_DIR }}
        run: ctest -V

      # 打包构建产物（使用 CPack 生成二进制压缩包）
      # 生成格式：项目名-版本-Linux.tar.gz，包含可执行文件、头文件、依赖库
      - name: 打包构建产物
        working-directory: ${{ env.BUILD_DIR }}
        run: cpack -G TGZ

      # 归档打包结果（保存为 CI 制品，可在工作流详情页下载，后续发布/部署复用）
      - name: 上传构建制品
        uses: actions/upload-artifact@v4
        with:
          name: cpp-centos-release-package
          path: ${{ env.BUILD_DIR }}/*.tar.gz
          retention-days: 30  # 制品保留30天

  ###########################################################################
  # 阶段3：测试环境自动部署（仅主分支推送时触发，PR不触发）
  ###########################################################################
  deploy-staging:
    name: 部署测试环境
    runs-on: ubuntu-latest
    needs: build-test-package
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      # 下载上一步打包好的制品
      - name: 下载构建制品
        uses: actions/download-artifact@v4
        with:
          name: cpp-centos-release-package

      # 通过 SSH 连接测试服务器，上传包并执行部署
      # 前置配置：仓库 Secrets 中添加 SSH_PRIVATE_KEY、DEPLOY_HOST、DEPLOY_USER
      - name: SSH 部署到测试服务器
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.DEPLOY_HOST }}        # 测试服务器IP/域名
          username: ${{ secrets.DEPLOY_USER }}    # 服务器用户名
          key: ${{ secrets.SSH_PRIVATE_KEY }}     # SSH 私钥
          port: 22
          script: |
            # 1. 创建部署目录
            mkdir -p /opt/app/staging
            cd /opt/app/staging
            
            # 2. 备份上一版本（回滚备用）
            if [ -f app-bin.tar.gz ]; then
              mv app-bin.tar.gz app-bin.backup.tar.gz
            fi
            
            # 3. 解压包、重启服务（根据你的实际服务调整命令）
            # tar -zxf app-bin.tar.gz
            # systemctl restart your-app.service
            echo "部署包更新完成，服务已重启"

  ###########################################################################
  # 阶段4：自动发布 Release（仅推送 v 开头标签时触发）
  ###########################################################################
  publish-release:
    name: 发布正式 Release
    runs-on: ubuntu-latest
    needs: build-test-package
    if: startsWith(github.ref, 'refs/tags/v')
    steps:
      # 下载构建制品
      - name: 下载构建制品
        uses: actions/download-artifact@v4
        with:
          name: cpp-centos-release-package

      # 创建 GitHub Release，自动生成更新日志，上传二进制包
      - name: 创建 Release 并上传制品
        uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true    # 自动根据提交记录生成更新日志
          files: ./*.tar.gz               # 上传二进制包作为 Release 附件
          body: |
            ## 版本说明
            本次版本通过 CI 自动构建发布，基于 CentOS Stream 9 环境编译。
            - 编译器：clang
            - 构建类型：Release
            - 完整单元测试校验通过

  ###########################################################################
  # 阶段5：自动更新 README（发布新版本后，更新README中的最新版本号）
  ###########################################################################
  update-readme:
    name: 自动更新 README 版本
    runs-on: ubuntu-latest
    needs: publish-release
    if: startsWith(github.ref, 'refs/tags/v')
    steps:
      # 检出主分支代码（更新要提交到主分支，而非tag）
      - name: 检出主分支代码
        uses: actions/checkout@v4
        with:
          ref: main
          token: ${{ secrets.GITHUB_TOKEN }}

      # 提取版本号（去掉 tag 前缀 v）
      - name: 提取版本号
        id: get_version
        run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

      # 替换 README 中的版本号占位符
      # 示例：将 README 中「最新版本：vX.X.X」替换为当前发布版本
      - name: 更新 README 版本信息
        run: |
          sed -i "s/最新版本：v[0-9.]*$/最新版本：v${{ steps.get_version.outputs.VERSION }}/g" README.md

      # 提交修改并推送到主分支
      - name: 提交并推送更新
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add README.md
          if ! git diff --cached --quiet; then
            git commit -m "docs: 自动更新README版本号为 v${{ steps.get_version.outputs.VERSION }}"
            git push origin main
          fi

  ###########################################################################
  # 阶段6：机器人通知（工作流完成后推送结果，支持飞书/钉钉/企业微信）
  ###########################################################################
  notify-bot:
    name: 机器人结果通知
    runs-on: ubuntu-latest
    needs: [ build-test-package, deploy-staging, publish-release, update-readme ]
    if: always()  # 无论成功失败都执行通知
    steps:
      # 计算整体工作流状态
      - name: 汇总执行状态
        id: status
        run: |
          if [[ "${{ contains(needs.*.result, 'failure') }}" == "true" ]]; then
            echo "STATUS=❌ 执行失败" >> $GITHUB_OUTPUT
          else
            echo "STATUS=✅ 执行成功" >> $GITHUB_OUTPUT
          fi

      # 发送 Webhook 通知（以飞书/钉钉通用 webhook 为例）
      # 前置配置：仓库 Secrets 中添加 NOTIFY_WEBHOOK 为你的机器人 webhook 地址
      - name: 推送通知到机器人
        run: |
          curl -X POST ${{ secrets.NOTIFY_WEBHOOK }} \
            -H "Content-Type: application/json" \
            -d '{
              "msg_type": "text",
              "content": {
                "text": "C++ 项目 CI/CD 执行结果\n状态：${{ steps.status.outputs.STATUS }}\n分支：${{ github.ref_name }}\n提交人：${{ shturl.cc/V1 }}\n提交信息：${{ github.event.head_commit.message }}\n详情：${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
              }
            }'
```

# 容器化：Docker

# 容器管理：Kubernetes(K8s)

# License

**开源 ≠ 免费**；开源是「公开源代码」，你依然有遵守协议的义务；闭源 = 不公开源码。
\## 闭源
- 你不对外发布源代码，仅分发二进制程序；用户无权逆向、修改、二次分发源码。
- 绝大多数商业软件默认就是专有许可证。
- 开源协议传染性：链接了GPL协议开源库，整个程序必须开源
\## 开源许可证
1. MIT License
2. Apache License 2.0
3. BSD 2-clause / 3-clause
4. GPL v2 / GPL v3（会传染）
5. LGPL（动态库常用，动态链接主程序可以不公开）
6. AGPL v3（会传染）
\## license文件
- MIT License

    Copyright (c) [年份] [你的名字/公司名]

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
