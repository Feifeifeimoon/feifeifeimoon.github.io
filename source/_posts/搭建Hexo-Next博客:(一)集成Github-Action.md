---
title: '搭建Hexo+Next博客:(一)集成Github-Action'
tags: hexo
abbrlink: 51a7104d
date: 2022-01-12 00:11:16
update: 2022-01-15 20:54:12
---

这次重新折腾博客，主要有几个目标

- 添加多样性插件。比如支持评论
- 容易部署，如果切换电脑同步方便

最终选定了基于 `Hexo➕Next` 主题，托管在 `Github` 上，并且使用 `Github Action` 实现推送文章后自动部署。

<!-- more -->
## Overview

创建一个 `hexo` 分支，用来存放 `hexo` 相关代码。也有些文章推荐创建一个 `private` 的仓库来存放 `hexo` 相关代码，也能通过 `Action` 实现自动部署到 `username.github.io` 仓库里。这里为了不创建多个仓库，使用一个仓库的两个不同分支来管理。

基本流程如下，我们将文章或者博客的配置信息推送到 `Hexo` 分支然后触发 `Github Action` ，将 `Hexo` 分支生成的静态网页部署到 `Master` 分支，然后通过 `username.github.io ` 访问。

![](image.png)



## Environment

首先要进行环境配置，主要是 `node` 环境以及 `git` 配置。这里以 `mac` 环境为例其它环境类似只是安装工具不同。

### nodejs

`Hexo` 基于 [Node.js](https://nodejs.org/en/download/)，要先安装 `node.js` 和 `npm`。

```Bash
# 使用homebrew安装
$ brew update
$ brew install node
# 查看nodejs和npm版本
$ node -v
v17.3.0
$ npm -v
8.3.0
```

{% note info %}
注意 `node` 版本需要在 12 以上，否则使用 `hexo` 时会报错，可以用以下方法升级 `node` 版本。
{% endnote %}

```Bash
$ npm install -g n
$ n stable # latest #(升级node.js到最新版) stable #（升级node.js到最新稳定版）
  installing : node-v16.13.2
       mkdir : /usr/local/n/versions/node/16.13.2
       fetch : https://nodejs.org/dist/v16.13.2/node-v16.13.2-linux-x64.tar.xz
   installed : v16.13.2 (with npm 8.1.2)

Note: the node command changed location and the old location may be remembered in your current shell.
         old : /usr/bin/node
         new : /usr/local/bin/node
If "node --version" shows the old version then start a new shell, or reset the location hash with:
hash -r  (for bash, zsh, ash, dash, and ksh)
rehash   (for csh and tcsh)
$ node -v
v16.13.2
```


### git

因为使用 `Github` 进行博客的托管和部署。要先进行 `git` 相关配置，以便能够访问 `Github` 中的仓库

```Bash
# install git
$ apt-get install git
# config username and email
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com

```


然后配置 `ssh` 密钥来拉取 `Github` 仓库。按照[Github官方教程](https://docs.github.com/cn/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)即可.

## Init Project

环境配置好后就可以进行工程的建立，首先在 `GitHub` 上建立一个名为 `username.github.io` 格式的仓库。因为我们要用 `GitHub` 支持的静态网站部署博客。更多关于 [Github Pages](https://pages.github.com/)

![](image_1.png)

接着将这个仓库 `clone` 到本地，创建我们的 `hexo` 分支

```Bash
$ git clone https://github.com/username/username.github.io
$ cd username.github.io
$ git checkout -b hext
```


这时候就可以用 `hexo` 来初始化 `hexo` 工程，参照[hexo官方文档](https://hexo.io/zh-cn/docs/)

```Bash
# 首先安装hexo
$ npm install -g hexo-cli
$ hexo init
$ ls
_config.landscape.yml  _config.yml  node_modules  package.json  package-lock.json  scaffolds  source  themes
```


这时我们的 `hexo` 的项目就创立成功了，这时只要运行 `hexo g && hexo s` 就可以访问初始的 `hexo` 界面

```Bash
$ hexo g && hexo s

```


访问[http://localhost:4000](http://localhost:4000)

![](image_2.png)

### next theme

接着进行主题配置，`hexo` 有很多主题可以到[官网](https://hexo.io/themes/)查看。这里使用 `next` 主题。

关于 `next` 主题主要有两个仓库。

![](image_3.png)

第一个 `Star` 数量比较多，也是最开始 `next` 仓库，第二个仓库是国内大佬 `fork` 出来。主要原因在 [issues](https://github.com/next-theme/hexo-theme-next/issues/4)中提到

![](image_4.png)



这里选择支持国人的

```Bash
$ git submodule add https://github.com/theme-next/hexo-theme-next themes/next
# 如果之前使用的是git clone方法 直接添加submodule会保错
$ git rm -f --cached themes/next


```


拉取完子模块后，修改 `_config.yaml` 找到 `theme` 修改为 `next`。再执行 `hexo g && hexo s` 就可以看到主题已经切换为 `next`。

![](image_5.png)

这里还要说一点关于 `hexo` 的配置文件，首先在根目录下有一个 `_config.yaml` 文件，这个文件是整个网站的配置，比如网站标题、描述等等。除此之外，我们都会使用 `hexo` 的主题，主题也会有一个配置文件，比如`next`的配置文件在 `themes/next/_config.yml` 中。

但是如果我们直接修改 `themes/next/_config.yml`，由于是以子模块的方式组成的，就需要也提交这个文件显然是不允许的。其实 `hexo` 也想到这一点，可以看到在我们使用 `hexo` 创建出来的工程里会有一个文件叫 `_config.landscape.yml `。`landscape` 就是 `hexo` 的默认主题。那么同理我们只需要一个 `_config.next.yml` 就不需要修改 `next`工程中的配置文件。

```Bash
$ cp themes/next/_config.yml _config.next.yml
#  这个文件对我们已经没有用了
$ rm _config.landscape.yml
```


接着修改 `_config.next.yml` 里的主题使用 `Pisces` 

```Bash
# Schemes
#scheme: Muse
#scheme: Mist
scheme: Pisces
#scheme: Gemini
```


重新执行 `hexo g && hexo s` 就可以看到主题已经切换。

## Deploy

创建完博客后就可以部署到我们的 `Github` 上。首先我们先来了解一下 `hexo` 的部署方式。

首先 `Hexo` 提供了快速方便的[一键部署](https://hexo.io/zh-cn/docs/one-command-deployment.html)功能，只需一条命令就能将网站部署到服务器上。在执行前需要配置 `_config.yaml`。

```Bash
deploy:
  type: git
  # 注意这里一道要配置ssh的方式 如果使用https方式的话 后续无法通过Github Action部署
  repo: git@github.com:usernmame/username.github.io.git
  # 指定部署到的分支
  branch: [branch]
  message: [message]
```


接着只需要执行，就可以将我们的博客部署到 `GitHub` 上。

```Bash
$ npm install hexo-deployer-git --save
$ hexo deploy
```


但这种方式有一个问题，如果部署完后访问我们的仓库会发现在 `master` 分支上其实只是 `hexo` 生成的静态文件，而我们的主题配置等等还是在本地存放。这样我们就只能在当前这台电脑上进行写博客和发布，如果要同步又需要额外的同步手段。而我们使用 `hexo` 分支就是为了同步这些文件。

### Github Action

我们使用 `hexo` 分支来同步配置以及数据，但是每次写完博客或者修改了某些配置，又需要重新执行 `hexo deploy` 同步到 `master` 分支上。这又要求了我们每个可能写博客的机器又需要装一套 `node` 环境。

期望通过 `Github Action` 来完成 `depoly` 这件事情。实现当 `hexo` 分支有提交时，就自动 `deploy` 到 `master` 分支。这样写博客的机器就无须配置`node`之类的`hexo`环境。

配置 `Action` 十分简单，在项目的根目录中新建 `.github/workflow/` 目录，然后在目录下创建 `hexo.yml`(命名随意只要是 `yaml` 类型文件就可以)。目录结构如下

```Bash
username.github.io
└── .github
└──── workflows
└────── deploy.yml

```


然后思考我们需要 `action` 的流程应该是这样：

![](image_6.png)

- 拉取hexo分支的代码
- 安装node 环境
- 配置git（密钥、用户名..）
- 安装hexo依赖
- 运行hexo depoly

其中最复杂的就是 `git` 的配置，如果要想往仓库中提交代码就需要对应的权限，而 `Github` 的权限一般使用 `SSH` 密钥，这里我们生成一对密钥专门用来部署博客。

```Bash
$ ssh-keygen -f deploy-key
```


会生成 `deploy-key` 和 `deploy-key.pub` 两个文件，分别是私钥和公钥。接着需要将公钥 `deploy-key.pub` 作为 `Deploy keys` 添加到 `username.github.io` 项目中。

`Deploy keys` 就和我们往 `GitHub` 中添加 `ssh` 公钥一样，只不过这个 `Deploy keys` 是针对某个仓库的。


![](image_7.png)

有了公钥我们还需要私钥才能推送到仓库中。接着将私钥以 `Secrets` 的方式添加到项目中

{% note info %}
注意这里我们添加的名称为 `DEPOLY_KEY`
{% endnote %}

![](image_8.png)

接下来就是 `deploy.yml` 中的内容

```YAML
name: Hexo Deploy

on:
  # 指定只当hexo分支有push时才执行
  push:
    branches:
      - hexo

jobs:
  build:
    runs-on: ubuntu-latest
    name: A job to deploy blog.
    steps:
      #  拉取代码
      - name: Checkout
        uses: actions/checkout@v1
        with:
          submodules: true # Checkout private submodules(themes or something else).

      # 初始化node环境
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}

      # 配置环境 主要是git相关
      - name: Configuration environment
        env:
          # 这里就是在项目setting里配置的DEPLOY_KEY
          DEPLOY_KEY: ${{secrets.DEPLOY_KEY}}
        run: |
          mkdir -p ~/.ssh/
          echo "$DEPLOY_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name "feifeifeimoon"
          git config --global user.email "wangyufeimoon@gmail.com"

      # 安装hexo依赖
      - name: Install dependencies
        run: |
          npm i -g hexo-cli
          npm i
         
      # 部署hexo
      - name: Deploy hexo
        run: |
          hexo deploy

```


配置完成后，每次我们提交到 `hexo` 分支都会自动部署到 `master` 分支更新

![](image_9.png)

