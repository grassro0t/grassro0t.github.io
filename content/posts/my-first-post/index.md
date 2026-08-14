---
title: "第一篇文章"
date: '2026-08-14T16:32:09+08:00'
draft: false   # true=草稿，构建默认忽略
tags: ["测试标签"]
categories: ["测试"]
comments: true
cover:
    image: "/cover/post001.webp"
    alt: "扭曲的车流：城市夜晚的不安与躁动"
    hidden: false
---

# 本篇用于博客测试，日常工作流如下
```
# 新建文章
hugo new posts/xxx.md
# 本地热更新预览
hugo server -D
# 在文章中取消草稿模式
draft = false
# 写完提交推送，Actions自动构建发布
git add .
git commit -m "add article"
git push
```

# Hugo 常用命令
> 前提：进入项目根目录执行所有命令

## 一、基础运行
```bash
# 启动本地开发服务器（最常用）
hugo server

# 启动并自动打开浏览器；草稿内容也渲染
hugo server -D

# 指定端口（默认1313）
hugo server -p 1314

# 关闭实时刷新
hugo server --noLiveReload
```
访问地址：`http://localhost:1313`

## 二、创建内容
```bash
# 创建文章，自动根据archetype模板生成md文件
hugo new posts/my-first-post.md

# 创建草稿（默认草稿不会被正式构建，需 -D 预览）
hugo new posts/draft-article.md
```
文件路径：`content/posts/my-first-post.md`

Front Matter常用标识：
```toml
title = "标题"
date = 2026-08-14T12:00:00+08:00
draft = true   # true=草稿，构建默认忽略
tags = ["标签"]
categories = ["分类"]
```

## 三、构建静态站点（部署用）
```bash
# 生成静态文件到 public/ 文件夹
hugo

# 构建同时包含草稿
hugo -D

# 生产环境构建（压缩、优化资源）
hugo --minify
```
输出目录：`public/`，直接丢到 GitHub Pages / Vercel / Netlify

## 四、主题相关操作
### 1. 添加主题（Git Submodule 方式，推荐）
```bash
# 示例PaperMod主题，按需替换仓库地址
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# 后续拉取主题更新
git submodule update --remote themes/PaperMod
```
然后在 `hugo.toml` / `config.toml` 设置
```toml
theme = "PaperMod"
```

## 五、配置文件
hugo 支持三种格式，任选其一：
- `hugo.toml`（新版推荐）
- `config.toml`
- `config.yaml` / `config.json`

## 六、实用进阶命令
```bash
# 列出所有页面信息
hugo list all

# 列出草稿
hugo list drafts

# 列出未来定时文章（future）
hugo list future

# 清理 public 缓存
rm -rf public resources
# Windows
Remove-Item public, resources -Recurse -Force
```

## 七、部署简易流程（GitHub Pages 思路）
1. 本地写完文章，`hugo server -D` 预览
2. 修改 frontmatter `draft=false`
3. 执行 `hugo --minify`
4. 将 `public` 内静态文件推送到 pages 分支

> 自动化方案：GitHub Action 直接仓库源码构建，不用本地生成 public

## 八、高频踩坑
1. 修改配置文件后，重启 `hugo server` 才生效
2. `draft=true` 的文章，不加 `-D` 看不到
3. 图片路径：优先使用 `static/` 存放全局资源；页面内资源放页面同名文件夹（Page Bundle）
4. 主题更新不要直接改 themes 内源码，使用 `layouts/` 目录覆写模板

## 九、Page Bundle（常用组织方式）
```
content/posts/article/
├── index.md      # 文章正文
├── pic1.jpg      # 当前文章专用图片
```
在 md 中直接引用 `![](./pic1.jpg)`
![](/posts/test.png)