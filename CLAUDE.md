# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个基于 Jekyll 的静态博客项目（Hux Blog），fork 自 [huxpro/huxpro.github.io](https://github.com/Huxpro/huxpro.github.io)。

## 常用命令

```bash
# 本地预览博客
jekyll serve

# 安装依赖
npm install
```

## 目录结构

```
huxblog-boilerplate/
├── _posts/                    # 博客文章目录
│   └── YYYY-MM-DD-title.markdown    # 文章文件，命名格式为日期-标题
│
├── _layouts/                  # 页面布局模板
│   ├── default.html           # 基础布局（HTML 骨架）
│   ├── post.html              # 文章详情页布局
│   ├── page.html              # 普通页面布局（如 about 页面）
│   └── keynote.html           # 演讲/幻灯片布局
│
├── _includes/                 # 可复用的 HTML 组件
│   ├── head.html              # <head> 标签内容（Meta、CSS 链接等）
│   ├── nav.html               # 顶部导航栏
│   └── footer.html            # 页脚
│
├── css/                       # 编译后的 CSS 文件
│   ├── hux-blog.css           # 博客主样式
│   ├── bootstrap.css          # Bootstrap 框架样式
│   └── syntax.css             # 代码高亮样式
│
├── less/                      # Less 源文件（开发用）
│   ├── hux-blog.less          # 博客样式
│   ├── variables.less        # 变量定义
│   ├── mixins.less            # Mixin 集合
│   └── sidebar.less           # 侧边栏样式
│
├── js/                        # JavaScript 文件
│   ├── hux-blog.js            # 博客交互逻辑
│   ├── jquery.js              # jQuery 库
│   ├── bootstrap.js           # Bootstrap JS
│   └── jquery.tagcloud.js     # 标签云插件
│
├── img/                       # 图片资源
│   ├── avatar-hux.jpg         # 博主头像
│   ├── home-bg.jpg            # 首页背景图
│   ├── post-bg-*.jpg          # 文章页背景图
│   └── in-post/               # 文章内使用的图片
│
├── fonts/                     # 字体文件（Bootstrap glyphicons）
│
├── _config.yml                # 博客配置文件（标题、URL、社交账号等）
│
├── index.html                 # 首页（博客列表）
│
├── about.html                 # 关于页面
│
├── tags.html                  # 标签聚合页面
│
├── 404.html                   # 404 错误页面
│
├── feed.xml                   # RSS 订阅文件
│
├── CNAME                      # 自定义域名配置
│
├── Gruntfile.js               # Grunt 构建配置（Less 编译、压缩等）
│
└── package.json               # NPM 依赖配置
```

## 核心文件说明

| 文件/目录 | 作用 |
|-----------|------|
| `_posts/*.markdown` | 博客文章内容，使用 Front Matter 定义元数据 |
| `_layouts/*.html` | 页面布局模板，文章通过 layout 属性指定使用哪种布局 |
| `_includes/*.html` | 可被布局和页面 include 的组件片段 |
| `_config.yml` | 站点配置，包含博客标题、URL、Disqus、GA、百度统计等 |
| `css/hux-blog.css` | 博客主样式，由 `less/hux-blog.less` 编译生成 |
| `Gruntfile.js` | 定义了 less 编译、css/js 压缩、监听文件变化等任务 |

## Front Matter 格式

文章文件头部需要包含如下 YAML 块：

```yaml
---
layout: post
title: "文章标题"
date: YYYY-MM-DD HH:MM:SS
categories: [分类1, 分类2]
tags: [标签1, 标签2]
---
```

## 添加新文章

在 `_posts/` 目录下创建 `YYYY-MM-DD-title.markdown` 文件，添加 Front Matter 后编写 Markdown 内容即可。
