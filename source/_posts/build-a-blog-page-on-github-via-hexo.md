---
title: 基于 Hexo 利用 github pages 搭建个人主页
toc: true
date: 2022-06-04 21:32:32
tags: [hexo, github, test]
categories: ["build blog pages"]
description: 本文记录了基于 hexo 利用 github pages 免费搭建个人主页的过程 
---

# 环境搭建

## 本机环境

> Windows edition:	Windows 11 Pro 
> Windows version: 10.0.22621.1 
> WSL version: 0.58.3.0
> Kernel version: 5.10.102.1
> WSLg version: 1.0.33
> MSRDC version: 1.2.2924
> Direct3D version: 1.601.0

所有环境都在 Windows 子系统（debian）中搭建。

## 安装 node.js

首先更新系统

```bash
sudo apt update && sudo apt upgrade
```

安装 nodejs 与 npm

```bash
sudo apt install nodejs
sudo apt install npm
```

但是这样安装后发现使用 hexo 时会出现错误

```txt
TypeError: Object.fromEntries is not a function
```

将 nodejs 和 npm 升级即可

### 升级node.js和npm

npm 中有一个模块叫做 “n” ，专门用来管理node.js版本

更新到最新的稳定版可在命令行中打下如下代码：

```bash
npm install -g n
n stable
```

npm升级只需要在终端中输入：

```bash
npm -g install npm@next
```

升级后检查其版本

```bash
node -v && npm -v
```

```bash
output
v16.15.1
8.12.1
```



## 安装 git

使用 debian 默认的包管理器安装 git

```bash
sudo apt install git
```

确认 git 是否被正确安装

```bash
git --version
```

```bash
Output
git version 2.20.1
```

## 安装 Hexo

在合适的地方新建目录

```bash
mkdir Blog
```

进入新建目录并安装 Hexo

```bash
cd Blog
sudo npm i hexo-cli -g #这里要加 sudo 否则会报错
```

验证 Hexo 是否安装成功

```bash
hexo -v
```

```bash
output
INFO  Validating config
hexo: 6.2.0
hexo-cli: 4.3.0
os: linux 5.10.102.1-microsoft-standard-WSL2 Debian GNU/Linux 10 (buster) 10 (buster)
node: 16.15.1
v8: 9.4.146.24-node.21
uv: 1.43.0
zlib: 1.2.11
brotli: 1.0.9
ares: 1.18.1
modules: 93
nghttp2: 1.47.0
napi: 8
llhttp: 6.0.4
openssl: 1.1.1o+quic
cldr: 40.0
icu: 70.1
tz: 2021a3
unicode: 14.0
ngtcp2: 0.1.0-DEV
nghttp3: 0.1.0-DEV
```

### Hexo  初始化

```bash
hexo init #初始化当前目录
sudo npm install #安装必备组件
```

### Hexo 常用命令

```bash
hexo g # 生成静态网页
hexo s # 打开本地服务器预览，http://localhost:4000/
hexo d # 部署网页到远端服务器
```

## 配置 github 与本地连接

在 Blog 目录下添加 github 信息

```bash
git config --global user.name "your name"
git config --global user.email "your email address"
```

生成 SSH 密钥

```bash
ssh-keygen -t rsa -C "youre email address" # 一路回车即可
```

打开 [github](http://github.com/) ，在头像下面点击 `settings`，再点击 `SSH and GPG keys `，新建一个`SSH`，名字随意，再填入刚刚生成的公钥即可。

使用如下命令查看公钥

```bash
cat ~/.ssh/id_rsa.pub
```

确认是否连接成功

```bash
ssh -T git@github.com
```

可以看到你的用户名即可

最后需要打开博客根目录下的 `_config.yml` 文件，配置部署信息，修改如下：

```bash
# vim _config.yml

deploy:
  type: git
  repo: https://github.com/yourname/yourname.github.io
  #repo: git@github.com:yourname/yourname.github.io.git #若部署时出现 Spawn failed 错误，可用这种方式解决
  branch: main
```

# 发布第一篇博客

在 `Blog` 根目录安装 `heox-deployer-git` 扩展用于将文章发布至 github

```bash
sudo npm i hexo-deployer-git
```

使用 `hexo new` 新建一篇文章

```bash
hexo new post "your article title"
```

命令执行后，`/your/blog/path/Blog/source/_posts` 目录下会生成这篇文章的 `markdown` 文件，可以使用 `markdown` 编辑器来编辑文章内容。

编写完成后，通过 `hexo g && hexo s` 生成静态网页文件并在本地预览，最后通过 `hexo d` 上传至 github，此时打开 `yourname.github.io` 就可以看到你发布的文章啦。

<img src="https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/20220605150017.png" alt="blog早期"  />

# 个性化配置

## 修改主题为 next 主题

`next` 主题是 `Hexo` 框架下最流行的主题，同时该主题也比较符合我的审美，因此将默认主题修改为 `next` 主题。

在 `Blog` 根目录下执行如下命令

```bash
git clone https://github.com/next-theme/hexo-theme-next themes/next
```

修改根目录下 `_config.yml` ，将默认主题改为 `next` 即可

```bash
## Themes: https://hexo.io/themes/
theme: next #landscape
```

**Note: 如果没有特殊说明， 以下个性化配置项目都是基于 `next` 主题的。**

## 设置主页显示文章摘要

### 通过设置 `description` 显示部分摘要

在文章的 `front-matter` 中添加 `description` ，其中 `description` 中的内容就会被显示在首页上，其余一律不显示。

```bash
---
title: 让首页显示部分内容
date: 2022-06-05 22:55:10
description: 这是显示在首页的概述，正文内容均会被隐藏。
---
```

### 文章截断

在需要截断的地方加入如下代码

```markdown
<!--more-->
```

首页就会显示这条以上的所有内容，隐藏接下来的所有内容。

### 自动生成摘要

需要在主题配置中添加如下代码

```bash
auto_excerpt:
  enable: true
  length: 150 # 摘要所截取的字符长度
```

## 代码高亮

修改站点配置文件 `_config.yml` 

```bash
highlight:
  enable: true
  line_number: true
  auto_detect: true #false
```

主题配置文件 `your/blog/path/themes/next/_config.yml` 中，修改 highliht_theme 后面的值。可选值有

```bash
normal
night
night eighties
night blue
night bright
```





## 添加 busuanzi  统计功能

![在网站底部显示访客数和浏览次数](https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/20220605175354.png)

修改主题配置文件

```bash
# Show Views / Visitors of the website / page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi
busuanzi_count:
  enable: true #false
  total_visitors: true
  total_visitors_icon: fa fa-user
  total_views: true
  total_views_icon: fa fa-eye
  post_views: true
  post_views_icon: fa fa-eye
```

# 参考文献

[1] [超详细Hexo+Github博客搭建小白教程](https://godweiyang.com/2018/04/13/hexo-blog/)

[2] [GitHub+Hexo 搭建个人网站详细教程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/26625249)

[3] [hexo博客front-matter格式 - luwanglin - 博客园 (cnblogs.com)](https://www.cnblogs.com/luckforefforts/p/13642702.html)

[4] [hexo 创建文章、标签、分类的Front-matter - 简书 (jianshu.com)](https://www.jianshu.com/p/6e4af897a3f0)

[5] [Hello Hexo！一款高效的个人博客框架 - 简书 (jianshu.com)](https://www.jianshu.com/p/265ff281e159)

[6] [hexo的next主题个性化教程:打造炫酷网站 - 简书 (jianshu.com)](https://www.jianshu.com/p/f054333ac9e6)

[7] [Hexo Next 博客自定义配置 | 斯是陋室，惟吾德馨 (yi-yun.github.io)](https://yi-yun.github.io/Hexo-Next-Custom/)

[8] [设置hexo首页只显示部分摘要（不显示全文） | Z Blog (yueyue200830.github.io)](https://yueyue200830.github.io/2020/02/23/设置hexo首页只显示部分摘要（不显示全文）/)

[9] [Hexo使用NexT主题设置主页显示文章摘要方法 | 春风十里 (jiangding1990.github.io)](https://jiangding1990.github.io/2017/04/25/Hexo使用NexT主题设置主页显示文章摘要方法/)

[10] [打造个性超赞博客 Hexo + NexT + GitHub Pages 的超深度优化 | reuixiy (io-oi.me)](https://io-oi.me/tech/hexo-next-optimization/#文章摘要图片)

[11] [How To Install Node.js on Debian 10 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-debian-10)

[12] [升级node.js和npm - SegmentFault 思否](https://segmentfault.com/a/1190000009025883)

[13] [Hexo生成博客遇到的坑_0xwang的博客-CSDN博客](https://blog.csdn.net/zhongyuemengxiang/article/details/122510536)

[14] [Hexo+github搭建个人博客 | 武师叔 (wushishu.xyz)](https://wushishu.xyz/post/be8880ea.html)

[15] [Hexo部署出现错误err: Error: Spawn failed解决方式 - 简书 (jianshu.com)](https://www.jianshu.com/p/c60ad2a33a1e)

[16] [Hexo使用说明 | JaydenZ's Blogs](https://jaydenz.github.io/2018/04/06/1.Hexo使用说明/#Hexo命令常见用法)

[17] [next-theme/hexo-theme-next: 🎉 Elegant and powerful theme for Hexo. (github.com)](https://github.com/next-theme/hexo-theme-next)

[18] [willin/hexo-wordcount: A Word Count Plugin for Hexo (github.com)](https://github.com/willin/hexo-wordcount)

















