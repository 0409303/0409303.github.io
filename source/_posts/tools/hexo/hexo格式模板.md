---
    title: {{ title }}
    categories: tools
    tags: hexo
    creator: cjq
    create_time: 2021/02/20



---



[toc]



## 给我一个Front-matter例子

这里是一个包含了常用字段的 **Hexo Front-matter 示例**，可以直接复制使用：

yaml

复制下载

```
---
title: 使用Hexo搭建个人博客的完整指南
date: 2026-04-24 10:30:00
updated: 2026-04-25 15:20:00
tags:
  - Hexo
  - GitHub Pages
  - 教程
categories:
  - [技术, 前端, 静态博客]
  - [工具]
keywords: Hexo,博客搭建,教程
description: 这是一篇详细介绍如何使用Hexo和GitHub Pages搭建个人博客的文章，包含完整的配置步骤和常见问题解答。
author: 你的名字
layout: post
comments: true
toc: true
---
```



### 📌 各项字段说明

| 字段          | 是否必需 | 作用                                         |
| :------------ | :------- | :------------------------------------------- |
| `title`       | ✅ 必需   | 文章标题，会在页面上显示                     |
| `date`        | ✅ 必需   | 文章创建时间，影响排序和归档                 |
| `updated`     | ❌ 可选   | 文章最后修改时间，SEO友好                    |
| `tags`        | ❌ 可选   | 文章标签（可以是列表或单值）                 |
| `categories`  | ❌ 可选   | 文章分类，支持**多级层级**（写法如示例所示） |
| `keywords`    | ❌ 可选   | 用于SEO的页面关键词                          |
| `description` | ❌ 可选   | 页面描述，在搜索引擎结果中展示               |
| `author`      | ❌ 可选   | 作者名（若主题支持多作者时可使用）           |
| `layout`      | ❌ 可选   | 指定布局模板，默认`post`                     |
| `comments`    | ❌ 可选   | 是否开启评论（需主题支持）                   |
| `toc`         | ❌ 可选   | 是否显示目录（需主题支持）                   |

### 💡 关于 `categories` 多级写法的补充说明

如果你希望一篇文章同时属于 **两个不同的分类体系**（例如既属于“日常/游记”，又属于“创作/散文”），可以这样写：

yaml

复制下载

```
categories:
  - [日常, 游记]    # 父级“日常”，子级“游记”
  - [创作, 散文]    # 另一个独立的分类路径
```



Hexo 会自动为这两个分类路径分别生成页面，文章会出现在两个分类页面中。

如果你希望一个分类下有两级（只有一条路径），也可以简写为：

yaml

复制下载

```
categories:
  - 技术
  - 后端
```



但这会被解析为两个**同级**分类，而不是层级关系。要表示层级，必须用数组嵌套：`- [技术, 后端]`。

### ⚙️ 快速生成 Front-matter

可以使用 `hexo new "标题"` 命令自动生成带有 `title` 和 `date` 的基础 Front-matter；如需更多字段，建议使用 VS Code 的 **Hexo Front-matter** 等插件，或自己复制模板文件到 `scaffolds/post.md` 中。

例如，修改 `scaffolds/post.md` 为你想要的模板：

yaml

复制下载

```
---
title: {{ title }}
date: {{ date }}
updated: 
tags:
categories:
description: 
toc: true
---
```



这样之后每次 `hexo new "标题"` 都会生成带有这些字段的文件。