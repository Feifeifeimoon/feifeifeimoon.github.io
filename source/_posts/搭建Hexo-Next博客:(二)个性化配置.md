---
title: '搭建Hexo+Next博客:(二)个性化配置'
tags: hexo
abbrlink: 6e3623f6
date: 2022-01-15 22:20:26
update: 2022-01-15 22:20:26
---

在之前的文章我们使用 `Hexo + Next` 创建了一个博客，并且使用 `Github Action` 实现了自动部署功能。但默认的 `Next` 主题还是有些简单，接着我们来给博客添加一些个性化配置。

<!-- more -->
## 语言和时区

默认 `next` 会使用英文，修改为中文并选择时区为 `Asia/Shanghai`。修改 `_config.yml`

```YAML
# Site
# 标题
title: Hexo
# 副标题
subtitle: ''
# 描述
description: ''
# 关键字
keywords: 
# 作者
author: XXXX
# 语言
language: zh-CN
# 时区
timezone: 'Asia/Shanghai'
```




## 启用搜索功能

我认为一个博客肯定需要有搜索功能，当文章多了之后能够快速的根据关键字搜索。效果如下：

![](image.png)

搜索功能需要先安装 `hexo-generator-searchdb` 插件

```YAML
$ npm install hexo-generator-searchdb --save
```


然后首先在 `_config.yml` 文件最后添加以下内容

```YAML
# search
search:
  path: search.xml
  field: post
  format: html
  limit: 10000

```


还要在 `_config.next.yml` 中找到 `local_search` 配置，将 `enable` 改为 `true`

```YAML
# Local Search
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
local_search:
  enable: true
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false
```




## 显示文章更新时间

效果如下：

![](image_1.png)

当文章有更新时间这个属性时，`next` 默认就会显示。所以主要是如何给文章添加更新时间。先来了解一下 `scaffolds/post.md` 文件。当我们使用 `hexo new` 创建一篇新文章时就会以这个文件作为模板。在这个模板中添加 `update` 字段。

```YAML
---
title: {{ title }}
date: {{ date }}
update: {{ date }}
tags:
---

```


然后在已有的文章中也添加这个字段，注意的就是在更新文章后需要手动修改这个字段的值。如果没有成功显示，在 `_config.next.yml` 中查看 `post_meta` 字段中的 `updated_at` 是否为 `true`

```YAML
# Post meta display settings
post_meta:
  item_text: true
  created_at: true
  updated_at:
    enable: true
    another_day: true
  categories: true
```




## 计算文章字数和阅读时间

显示每篇文章包含多少字以及预计阅读时间，效果如下：

![](image_2.png)

首先需要安装  `hexo-word-counter`  插件

```YAML
$ npm install hexo-word-counter
```


修改 `_config.yml`

```YAML
# 计算阅读时间
symbols_count_time:
  symbols: true    # 开启字符计数
  time: true    # 显示阅读时间
  total_symbols: true    # 博客总字符数目
  total_time: true    # 博客总阅读时间
  exclude_codeblock: false
  awl: 2
  wpm: 275
```


- `awl` – 平均字节长度（`Average Word Length`）。默认为`4`
    - `CN ≈ 2`
    - `EN ≈ 5`
    - `RU ≈ 6`
- `wpm` – 每分钟阅读字数（`Words Per Minute`）。默认为`275`
    - `Slow ≈ 200`
    - `Normal ≈ 275`
    - `Fast ≈ 350`

*如果文章中大多中文，那么设置*`*awl*`*为*`*2*`*，*`*wpm*`*为*`*300*`*比较合适*。

接着在 `_config.next.yml` 中找到 `symbols_count_time` 配置

```YAML
# Post wordcount display settings
# Dependencies: https://github.com/theme-next/hexo-symbols-count-time
symbols_count_time:
  separated_meta: false         # 是否另起一行显示（即不和发表时间等同一行显示）
  item_text_post: true          # 首页文章统计数量前是否显示文字描述（本文字数、阅读时长）
  item_text_total: false        # 页面底部统计数量前是否显示文字描述（站点总字数、站点阅读时长）
```




## 代码块

代码块的主题配置，效果如下：

![](image_3.png)

修改 `_config.next.yml`

```YAML
codeblock:
  # Code Highlight theme
  # Available values: normal | night | night eighties | night blue | night bright | solarized | solarized dark | galactic
  # See: https://github.com/chriskempson/tomorrow-theme
  highlight_theme: night
  # Add copy button on codeblock
  copy_button:
    enable: true
    # Show text copy result.
    show_result: true
    # Available values: default | flat | mac
    style: mac
```




## 支持切换主题颜色

安装 `hexo-next-darkmode` 插件

```Bash
$ npm install hexo-next-darkmode --save
```


在 `Next` 主题的 `_config.yml` 配置文件里添加以下内容

```YAML
# Darkmode JS
# For more information: https://github.com/rqh656418510/hexo-next-darkmode, https://github.com/sandoche/Darkmode.js
darkmode_js:
  enable: true
  bottom: '64px' # default: '32px'
  right: 'unset' # default: '32px'
  left: '32px' # default: 'unset'
  time: '0.5s' # default: '0.3s'
  mixColor: 'transparent' # default: '#fff'
  backgroundColor: 'transparent' # default: '#fff'
  buttonColorDark: '#100f2c' # default: '#100f2c'
  buttonColorLight: '#fff' # default: '#fff'
  isActivated: false # default false
  saveInCookies: true # default: true
  label: '🌓' # default: ''
  autoMatchOsTheme: true # default: true
  libUrl: # Set custom library cdn url for Darkmode.js

```


- `isActivated: true`：默认激活暗黑 / 夜间模式，请始终与 `saveInCookies: false`、`autoMatchOsTheme: false` 一起使用



## 右上角添加GitHub连接

效果如下：

![](image_4.png)

- 1.首先到  [GitHub Corners](https://tholman.com/github-corners/)  或者  [GitHub Ribbons](https://github.blog/2008-12-19-github-ribbons/)  选择自己喜欢的图标，然后`copy`相应的代码
- 2.然后将刚才复制的代码粘贴到 `themes/next/layout/_layout.swig` 文件中 `<div class="headband"></div>` 下面一行
- 3.把代码中的 `href` 后面的值替换成你要跳转的地址，比如你的 `GitHub` 主页





## 访问人数和阅读统计

![](image_5.png)

![](image_6.png)

`NexT`主题已集成了不蒜子的访客人数和文章阅读统计功能，修改 `_config.next.yml`。

```YAML
# Show Views/Visitors of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi
busuanzi_count:
    enable: true
    # 总访问人数
    total_visitors: true
    total_visitors_icon: user
    # 总访问次数
    total_views: true
    total_views_icon: eye
    # 文章访问统计
    post_views: true
    post_views_icon: eye
```


注意：在本地运行时不蒜子显示的数据会是假的，部署后就正常了

