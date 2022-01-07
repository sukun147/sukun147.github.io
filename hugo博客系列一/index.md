# Hugo博客系列(一)


<!--more-->

# Hugo博客系列（一）

## 前言

### 为什么要有自己的博客？

写博文能很好地分享自己的想法，能记录生活，还能充当一个电子笔记的作用，有效防止今后某一天面对一个以前遇到过的问题但现在不会解决的情况出现。一篇好的博文能为你带来大量的流量，你可以在搜索引擎中搜索到自己，我相信这是一件足够令人雀跃的事情。并且你还能利用他做一篇网页简历，当你找工作时你可以有着更加花里胡哨的简历！除此以外，你还能通过交换友链建立一个优质的社交圈，因为大家都写了很多高质量的文章，你能与他们进行深入交流，这不像同学圈一样脆弱，它能稳定存在很久！

在其他博客平台写作你讲或多或少地受到限制，想自己 DIY 页面还得向官方申请，甚至不会审批通过。而且你无法使用多种多样的第三方插件，还得面对审查，写的文章有可能被删除、撤回。有广告干扰，任谁也不喜欢看着一篇文章然后突然蹦出来一个广告吧？当然，你也可以自己接点广告在网站上。

### 常见博客框架的选择

#### hexo

hexo 以前影响力还不错，但这几年已经不如从前人们预判的那么发展的好了，GitHub 上 hexo 项目有着 34k 个 star 看出，其中最为出名的主题便是 Next，有着 15.7k 个 star，这也是因为 Next 目前是 hexo 主题中功能最齐全最好用的一个。而如此之多的人使用也就意味着 hexo 这个框架的作者用收到非常多的反馈，因此 hexo 更新优化做的很好，作者也很有动力继续做下去。

但是这样的一个框架缺点也是有的：

1. 环境配置麻烦

   因为要使用 hexo 你得在本地安装 Node.js、Git，会熟练使用 GitHub，而且由于 GitHub 的特殊性，你还得学会翻墙，不然还用不了，这就对一点基础都没有的小白不是很友好了。

2. 无后端

   这意味着你没有一个后台还方便地对网站进行操作，只能通过先写好 Markdown 文件然后 Git 推送到云端。并且原生的 hexo 是没有评论系统的，想加评论还得找第三方评论系统。除此以外，一旦本地文件被你不小心删除掉了，那当你下次 push 的时候你之前的博文也就跟着丢失了。

3. 渲染时间久

   200 篇左右的博文用 Hexo 需要 10 分钟去生成静态网页，当你写博客写的时间久了之后文章多了起来，相信我，你会无法忍受这种折磨。

总结：hexo 适合有一定基础的人，然后写博客时间不长或者只是随便写来玩的人。当然尤其其广泛的传播性，当你遇到问题的时候拿到网上去搜一般都是有解决方案的，这也可以为你省下一点力气。

最后，如果你想用 hexo，我建议主题用 Next。

#### Hugo

Hugo 几年前的影响力是不如 hexo 的，但现在越来越多的人从 hexo 迁移到了 Hugo，Hugo使用人数也多了起来，GitHub 上 Hugo 项目有 56.2k 个 star，已远远超过了 hexo，因此你也不用太担心 Hugo 会不会太小众化的问题，但是 Hugo 上的主题选择会更少一些，其中最受欢迎的是 wowchemy，但也仅有 6.1k 个star，而本站采用的是 LoveIt 主题，它的 star 就更少了，才 1.6k 个。当然，如果你是搞前端开发的，或者乐意自己写主题，那这些就不重要了。

Hugo优点：

1. 速度快

   Hugo 采用 Go 语言编写，它的速度用作者的话来形容就是世界上最快的构建网站工具。并且 Hugo 是即时渲染的，这意味着你可以边写边改样式，直到你满意为止。即使是你写了几百篇文章，它也能在几秒之内全部渲染完成。

   > The world’s fastest framework for building websites

2. 配置更为简单

   你需要安装只是 Hugo，不像 hexo 还得安装 Node.js。并且Hugo 中是不区分站点和主题的配置文件的，Hugo 中只有一个位于站点根目录下的 config.toml 配置文件，你只用在这里面进行修改就可以了。

3. 方便自定义

   你可以在不修改主题文件的前提下方便地定制主题。在 Hugo 中，如果你想要定制主题，你只需在站点目录下新建相应的文件即可。这是非常利于主题的维护的，你只需使用 Git 的 submodule 的方式安装 Hugo 的主题，然后更新时只需直接在站点根目录下敲一条命令回车即可，非常方便！

缺点：

主题比较少，很可能大家都是用的同一个主题，并且主题作者更新会更少一点。

总结：如果你喜欢 DIY，我建议使用 Hugo。如果你是个专业博主，写了很多文章需要渲染，我建议使用 Hugo！

#### Typecho

这是一个非常轻量级的博客框架，但是需要你拥有一个服务器。并且它对服务器要求极低，即使只有 512M 内存或是更低，它也能跑起来。它可以满足你对博客的基本需求，而且 Typecho 是带后端的，意味着只要你能上网，你就可以自由地写你的文章，不会被设备所拘束。当然，你将免除配置环境的苦恼。

缺点：

1. 更新慢

   奇慢无比，作者已经 9 年没有进行更新了，一些插件也已经不能用了。

2. 自由度低

   你不能随心所欲地进行 DIY，当然，如果你只是用来写博客的话问题不是很大。

#### WordPress

世界上最受欢迎的建站工具！具体有多受欢迎？每三个网站就有一个是 WordPress 搭建，并且美国白宫自2017年起，其官网 Whitehouse.gov 网站的內容管理系統（Content management system，CMS）从 Drupal 换成 WordPress！

> WordPress 是一个以 PHP 和 MySQL 为平台的自由开源的博客软件和内容管理系统。WordPress 具有插件架构和模板系统。截至2018年4月，排名前1000万的网站中超过30.6%使用WordPress 。WordPress是最受欢迎的网站内容管理系统。全球有大约30%的网站(7亿5000个)都是使用WordPress架设网站的。WordPress 是目前因特网上最流行的博客系统。

并且 WordPress 并不只是可以用来写博客，它能用来打造一切你想要的网站，哪怕是用来建个电商网站也没有问题！

优点：

1. 超广泛传播性

   你在使用 WordPress 遇到的任何问题，你都可以在网上找到对应的解决方案，它的使用人数之多以至于每一个坑都有人替你趟过了！

2. DIY 自由度高，难度低

   你可以随心所欲地添加插件，WordPress 提供了大量的优质插件，甚至有大量的人就以制作 WordPress 上的插件谋生！

3. 安装简单

   网上有非常多的 WordPress 一键安装脚本，你可以根据自己的需求进行选择，无需面对安装过程中的问题！并且其安装时间非常短，只需要5分钟就能搞定！

缺点：

1. 需要有一定性能的服务器

   PHP，MySQL这些对服务器有一定的要求，会占用比较多的内存，它不像 Typecho 一样轻便。

2. 太过臃肿，不简洁

   太多的功能与选择造成了页面的繁琐，并且你会对着页面一直修改，这不利于你专心地撰写博文。

总结：适合有服务器，懒得折腾环境配置，喜欢开箱即用的人。

## Hugo快速上手教程

### Hugo安装

首先，请前往 GitHub 上下载最新版的 Hugo 压缩包，[Releases · gohugoio/hugo (github.com)](https://github.com/gohugoio/hugo/releases)，建议选择 extended 版本，这将更有利于后续的 DIY 操作！

![image-20220107180905839](https://s2.loli.net/2022/01/07/zHNxB3PCmiMhdgR.png)

下载完成后解压到一个你认为合适的位置，然后把 `hugo.exe` 所在的文件夹添加至环境变量中的 `Path` 中即可。

当然，你也可以采用源码编译的方式进行安装，这里就采用最简单的方法了。

{{< admonition >}}

如果你的 `Path` 变量字符数达到上限了，你可以新建一个变量，然后在这个变量下添加你要添加的 `Path`，然后再在 `Path` 中添加一个变量值为 `%name%` 即可。

![image-20220107181734504](https://s2.loli.net/2022/01/07/BjuasOCpk2HUDzr.png)

![image-20220107181857756](https://s2.loli.net/2022/01/07/2EJYS7FLlqy8j3a.png)

{{< /admonition >}}

检查一下上一步操作是否正确

```bash
hugo version
```

![image-20220107181019244](https://s2.loli.net/2022/01/07/FmoDJsKtEC2H5yM.png)

然后找一个合适的文件夹，在该目录下输入以下指令新建一个 Hugo 项目

```bash
hugo new site my_website
cd my_website
```

### Hugo主题选择、安装与快速上手

我这里采用 LoveIt 主题进行演示，事实上还有很多主题也很棒，比如 even、meme、wowchemy。

**LoveIt** 主题的仓库是: https://github.com/dillonzq/LoveIt.

你可以下载主题的 [最新版本 .zip 文件](https://github.com/dillonzq/LoveIt/releases) 并且解压放到 `themes`目录.

另外, 也可以直接把这个主题克隆到 `themes` 目录:

```bash
git clone https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

或者, 初始化你的项目目录为 git 仓库, 并且把主题仓库作为你的网站目录的子模块:

```bash
git init
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

那么如何更新主题呢？

```bash
git submodule update --rebase --remote
```

把 `\themes\LoveIt\exampleSite`目录下的`config.toml`复制下来，替换掉站点根目录下的同名文件。

然后对这个文件进行一些自定义修改。

然后进入根目录下的`archetypes`文件夹中，修改`default.md`文件为下面的内容（这个文件是模板文件，通过指令创建的文章将以模板为基础内容）

```markdown
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
tags: [""]
categories: [""]
toc:
  enable: true
description: 
draft: true

---

<!--more-->
```

现在开始撰写文章

```bash
hugo new posts/first_post.md
```

注意，后缀为md，建议使用 Typora 进行编辑。

首先修改 frontmatter，其中`title`表示文章标题，`date`为生成文章当时的时间，`tags`为标签，`categories`为目录，`toc enable`为启用文章目录（需要自己在文章中生成），`description`为文章摘要，`draft`表示是否为草稿（写完了文章把这里改为 false 即可），`<!--more-->`为 LoveIt 主题的摘要标识符，该标识符上方的内容为文章摘要，如果上方为空，则采用 frontmatter 中设置的`descriptions`为文章摘要。

例如本文的 frontmatter 为

```yaml
title: "Hugo博客系列(一)"
date: 2022-01-04T18:40:38+08:00
tags: ["Hugo"]
categories: ["web"]
toc:
  enable: true
description: 本系列教程第一章讲解了几种常见的博客框架选择，最终以 Hugo 框架为基础，教授了如何在 GitHub pages 上部署个人博客，还使用 GitHub actions 以及一个简单的 bat 脚本实现自动化发布。
draft: true
```

写完了文章进行网页的构建

```bash
hugo serve -D -e production
```

`-D`表示草稿也要渲染，`-serve`表示启动一个本地服务器，即时渲染，方便修改。

{{< admonition >}}

`hugo serve` 的默认运行环境是 `development`, 而 `hugo` 的默认运行环境是 `production`。

由于本地 `development` 环境的限制, 评论系统**, **CDN 和 fingerprint 不会在 `development` 环境下启用。

你可以使用 `hugo serve -e production` 命令来开启这些特性。

{{< /admonition >}}

值得一提的是不论输入的是`server`还是`serve`都是一样的。

![image-20220107191749776](https://s2.loli.net/2022/01/07/wPcCxbrhV5TKtWB.png)

![image-20220107191842429](https://s2.loli.net/2022/01/07/sGRqPztyM3L1OjQ.png)

在浏览器中前往它给出的 [http://localhost:1313](http://localhost:1313) 就能看到你刚生成的博客了。

{{< admonition  tip >}}

当你运行 `hugo serve` 时, 当文件内容更改时, 页面会随着更改自动刷新.

{{< /admonition >}}

现在再输入指令

```bash
hugo -D
```

这会生成一个 `public` 目录, 其中包含你网站的所有静态内容和资源. 现在可以将其部署在任何 Web 服务器上。

确认无误后就要把它发到公网上了，这里采用 GitHub pages 进行部署（当然，也有很多种方法也能达成这一目的）

### GitHub pages部署

如果你是第一次使用 GitHub，请自行搜索如何配置，这里不做讲解！

首先确保你有一个 GitHub 账号，然后新建一个仓库，名为`yourname.github.io`，注意，你应该保证这里的 your name 为你的 GitHub 账号名称！然后再进行以下步骤：

```bash
cd public
git init
git remote add origin https://github.com/yourname/yourname.github.io.git
#此URL可在你的repo中找到
git add .
git commit -m "update %date%,%time%"
git push origin master
```

![image-20220107193538729](https://s2.loli.net/2022/01/07/XCuPaW2mTYlnDNv.png)

如果一切顺利的话打开你的 GitHub repo，你就能看到相应的文件了，接着在 settings 页面中下滑，找到 GitHub pages，选择分支`master`，`root`路径，然后保存即可。如果你有自己的域名，还可以在下方的`custom domain`中输入你的域名，等待一段时间就可以用这个域名访问了。

![image-20220107193043431](https://s2.loli.net/2022/01/07/nDyJkuH16poXW54.png)

当然，在此之前你还需要再次修改`config.toml`文件中的`baseURL`为`https://yourname.github.io`，否则发布到网上也无法访问！

### GitHub actions实现自动部署(CI/CD)

是否觉得这个发布太过于繁琐了？别担心，这里提供两种解决方案！分别是本地 bat 脚本和GitHub actions。

首先是本地 bat 脚本，这将免除每次发布都要敲至少 5 行指令的痛苦。只需要每次要发布的时候双击运行一下程序即可。

```bash
hugo -D
cd public
git add .
git commit -m "update %date%,%time%"
git push origin master
pause
```

另一种方法是前往 GitHub，新建一个仓库。

点击`Actions`选择`simple workflow`，内容如下

```yaml
name: CI #自动化的名称
on:
  push: # push的时候触发
    branches: # 那些分支需要触发
      - master
jobs:
  build:
    runs-on: ubuntu-latest # 镜像市场
    steps:
      - name: checkout # 步骤的名称
        uses: actions/checkout@v2.3.4 #软件市场的名称
        with: # 参数
          submodules: true
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2.4.13
        with:
          hugo-version: latest
          extended: true
      - name: Build
        run: hugo -D
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
        deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        env:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          EXTERNAL_REPOSITORY: 14772/14772.github.io # 注意要修改本处地址
          PUBLISH_BRANCH: master
          PUBLISH_DIR: ./public
```

然后在本地输入以下命令在当前目录下生成密钥对

```
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
# You will get 2 files:
#   gh-pages.pub (public key)
#   gh-pages     (private key)
```

`-t rsa`表示 rsa 加密，`-b 4096`则表示长度为 4096bit，`-C`后面的是备注，`-f`后面的是文件名，`-N`是新密语

现在前往`yourname.github.io`仓库，选择Settings > Deploy keys > Add deploy key，勾选 Allow write access，内容为公钥(有`pub`字样的文件)

再前往之前存放了`master.yml`文件的仓库，选择Settings > Secrets > New secret，名称填`ACTIONS_DEPLOY_KEY`，内容为私钥

然后在站点根目录下执行以下命令

```bash
git remote add origin https://github.com/yourname/yourrepo.
#此repo为你放了master.yml文件的仓库
```

现在再来写一个 bat 脚本

```bash
git add .
git commit -m "update %date%,%time%"
git push origin master
```

之后直接双击运行它就行了

