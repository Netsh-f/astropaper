---
author: Zhang Wenjin
pubDatetime: 2025-12-02T06:00:00.000Z
title: 如何编写AstroPaper博客
tags:
  - docs
description: 如何添加文章信息、引用图片等
hideEditPost: true
timezone: Asia/Shanghai
---

## Table of contents

## 文件位置

在`src/data/blog/`目录下创建 Markdown 文件，一个 Markdown 文件就是一篇博客。

可以使用子目录对文章进行归类，如`src/data/bolg/2025/life.md`，这会影响到博客文章的 URL。

## 文章前言区（Frontmatter）

在 Markdown 文件顶部使用 YAML 格式填写本文的标题、日期等信息。

一个包含了所有字段的样例：

```
---
title: 文章的标题
description: 这是示例文章的示例描述。
pubDatetime: 2025-09-21T05:17:19.673Z
modDatetime: 2025-10-21T05:17:19.536Z
author: 作者名
slug: the-url-of-the-post
featured: true
draft: false
tags:
  - some
  - example
  - tags
ogImage: ../../assets/images/example.png # src/assets/images/example.png
canonicalURL: https://example.org/my-article-was-already-posted-here
hideEditPost: true
timezone: Asia/Shanghai
---
```

细节说明：

* `title`, `description`, `pubDatetime`: 三个字段是必须的
* `pubDatetime`, `modDatetime`: ISO 8601 格式
* `slug`: 文章的 URL 标识，会覆盖自动生成的
* `featured`: 设置为 true 时，会在首页的 Featured 区域显示
* `draft`: 设置为 true 时，文章不会被发布
* `tags`: 如不填，默认为`others`
* `ogImage`: 文章的 OG 社交分享图片，默认是`SITE.ogImage`，可以是远程 URL 或相对于当前文件夹的图片路径。推荐 OG 图片尺寸为 1200 * 640 像素，如果未指定，系统将自动生成。
* `canonicalURL`: 权威 URL，如当前文章已在其他地方发布过，则填写该 URL 指向原文章，目的是告诉搜索引擎原文章链接，避免重复收录。
* `hideEditPost`: 设置为 true 时，不显示编辑按钮
* `timezone`: 文章的时区，IANA 格式的时区

**特别说明**

* `pubDatetime`必须早于当前时间，否则文章不会被发布
* 不知道为什么，笔者时区设置是失效的（包括文章`timezone`设置和`config.ts`设置），不管是本地构建还是 Github Actions 构建，AstroPaper 参考的都是 UTC+0 时间。也就是说假如你在东八区 UTC+8 Asia/Shanghai，你要填写8小时前的时间，即当前 UTC+0 时间，否则文章不会被立刻发布
* 你可以通过 nodejs 中的`new Date().toISOString()`命令获取当前时间

## 添加目录

在你想加目录的地方，填写二级标题
```
## Table of contents
```

## 图片

### 方法1：放在`src/assets/`目录中（推荐）

将图片放在`src/assets/`中，它们会被自动优化

`/src/assets/images/example.jpg`两种引用方式：

```
![something](@/assets/images/example.jpg)
![something](../../assets/images/example.jpg)
```

### 方法2：放在`public/`目录中

放在这里的图片不会被优化，你应该自行使用 [TinyPng](https://tinypng.com/) 或 [TinyJpg](https://tinyjpg.com/) 等网站进行优化，否则会影响网站整体性能。

`/public/assets/images/example.jpg`两种引用方式：

```
![something](/assets/images/example.jpg)
<img src="/assets/images/example.jpg" alt="something">
```

## 参考

[Adding new posts in AstroPaper theme](https://astro-paper.pages.dev/posts/adding-new-posts-in-astropaper-theme/)