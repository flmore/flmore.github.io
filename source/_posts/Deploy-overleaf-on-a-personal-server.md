---
title: 在个人服务器上部署Overleaf
toc: true
date: 2022-12-22 22:49:24
tags: ["OverLeaf", "LaTex"]
categories: ["实验室"]
description: 国内访问OverLeaf平台速度不佳，本文以腾讯云轻量服务器为例，搭建自己的OverLeaf平台，实现LaTex文档的在线编辑和协作。
---

# 安装docker和docker-ce

参考[这篇文章](https://www.runoob.com/docker/centos-docker-install.html)安装`docker` 和 `docker-ce`

安装完成后分别运行`docker version` 和 `docker-ce version` 即可验证是否安装成功。

# 安装Overleaf

首先将overleaf项目从github拉至本地

```bash
git clone https://github.com/overleaf/toolkit.git ./overleaf
```

然后进行初始化配置

```bash
cd ./overleaf
bin/init
```

这里我们进入config下的overleaf.rc文件进行配置：

```bash
vim ./config/overleaf.rc
```

更改如下两行：

```bash
SHARELATEX_LISTEN_IP= your ip
#如果本地使用按照原配置即可，腾讯云服务器需要改为**内网地址** （轻量服务器信息面板可找到对应内网地址）

SHARELATEX_PORT=8080 # 改为想映射的端口，默认的80端口常常被占用
```

> 注意，映射的端口需要在**腾讯云防火墙**和**centos内部的防火墙**中均被开放

其中还有许多个性化的配置：如网页抬头文字内容，网址标题，UI语言（中文）等，可以在[overleaf的Wiki页面](https://github.com/overleaf/overleaf/wiki) 中进行查看和配置。

执行容器，如果之前没有安装过docker-compose还需要预先安装

```bash
bin/up 
```

此时正在拉取镜像，可以等出现大量的log时使用 ctrl+c 停止，然后执行

```bash
bin/start
```

即可。

此时用浏览器打开

```bash
http://公网IP:8080/
```

会看到管理员注册界面，至此overleaf的安装结束。

# 安装完整texlive包

以上安装的overleaf配套的LaTeX不是完整版，所以需要继续下载。

首先进入容器的bash

```bash
docker exec -it sharelatex bash
cd /usr/local/texlive
```

然后执行以下命令：

```bash
# 下载并运行升级脚本
wget http://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh
sh update-tlmgr-latest.sh -- --upgrade

# 更换texlive的下载源
tlmgr option repository https://mirrors.sustech.edu.cn/CTAN/systems/texlive/tlnet/

# 升级tlmgr
tlmgr update --self --all

# 安装完整版texlive（时间比较长，不要让shell断开）
tlmgr install scheme-full

# 退出sharelatex的命令行界面，并重启sharelatex容器
exit
docker restart sharelatex
```

# 配置中文字体

由于LaTeX默认带的中文字库缺乏很多生僻字，因此可以添加Windows的字库做补充。

首先将Windows系统下找到：

`C:\Windows\Fonts`,将这个文件夹打包后上传至服务器，并置于容器数据的挂载点：

```bash
tar xzvf windwos/fonts/path.tar.gz
cp windows/fonts/path/ ./data/sharelatex
```

进入容器：

```bash
docker exec -it sharelatex bash
cd /usr/share/fonts/
ln -sv /var/lib/sharelatex/windows-fonts /usr/share/fonts/windows-fonts
```

然后更新字体，执行：

```bash
fc-cache
```

即可安装字体文件。

然后查看字体文件是否正确安装：

```bash
fc-list | grep windows
```

# 创建管理员账户

将下面的邮件换成自己的邮箱

```bash
docker exec sharelatex /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email=joe@example.com"
```

然后打开浏览器访问`http://localhost:8080/user/activate?token=<token>`，配置密码激活后即可使用。

# 参考文献

[1] https://github.com/overleaf/overleaf/wiki/Quick-Start-Guide

[2] [利用腾讯云服务器搭建自己的overleaf（写论文神器） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/481788258)

[3] [centos-7本地overleaf安装 | null (haidi-ustc.github.io)](https://haidi-ustc.github.io/2021/08/01/computer/latex/overleaf/)

[4] [win10系统搭建本地overleaf平台实现多人远程协作修改Latex文档 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/592295650)

[5] [在自己的服务器上配overleaf - zhanghb (zhang-hb.com)](https://www.zhang-hb.com/2022/01/01/在自己的服务器上配overleaf/)

[6] [CentOS Docker 安装 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/centos-docker-install.html)
