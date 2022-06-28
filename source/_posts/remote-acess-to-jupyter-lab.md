---
title: 远程访问 Jupyter lab
toc: true
date: 2022-06-08 19:35:53
tags: [jupyter, "data process"]
categories: [software]
description: 使用 Jupyter lab 可以很方便的写 python 代码，直观的交互界面比较适合数据分析类任务。通过适当的配置，Jupyter lab 可以不止在本地使用，还可以远程访问，本文记录了实现 Jupyter lab 远程访问的配置过程。 
---

# 安装 Jupyter lab 

```bash
pip install jupyterlab
```

# 配置 Jupyter lab

## 生成 Jupyter 密码密文

### 自动生成

```bash
jupyter notebook password
```

```bash
output:
Enter password:  
Verify password: 
[NotebookPasswordApp] Wrote hashed password to $HOME/.jupyter/jupyter_notebook_config.json
```

### 手动生成

```python
# 在 ipython 中依次输入
from IPython.lib import passwd
passwd()
# 根据提示输入密码和确认输入，完成后得到密文
Enter password: 
Verify password: 
'sha1:********************************************'
```

也可以使用如下命令生成密文

```python
from notebook.auth import passwd; passwd()
```

## 修改 Jupyter 配置文件

1. 生成配置文件（配置文件路径默认为 `~/.jupyter/jupyter_notebook_config.py`）

```bash
jupyter-lab --generate-config
```

2. 修改配置文件

```bash
vim ~/.jupyter/jupyter_notebook_config.py # 打开配置文件

c.NotebookApp.ip='*' # 官方文档建议修改成‘*’，但可能还是无法访问，可以修改成‘0.0.0.1’或者服务器IP 
c.NotebookApp.password = u'sha1:........' # 修改成将之前生成的密文
c.NotebookApp.open_browser = False # 服务器端不自动打开浏览器
```

## 将 Jupyter lab 配置为系统服务

为了方便管理，可以将 jupyter 配置为系统服务，具体步骤如下：

1. 创建服务脚本 `vim /usr/lib/systemd/system/jupyter.service`

```bash
[Unit]
Description=Jupyter Management
After=network.target
 
[Service]
User=root
Group=root
ExecStart= /home/flmore/anaconda3/bin/jupyter-lab --allow-root # 可以通过 which jupyter-lab 确定 jupyter 路径
 
Restart=on-failure
RestartSec=10
 
[Install]
WantedBy=multi-user.target
```

2. 管理 `jupyter.service` 服务

```bash
systemctl daemon-reload		# 重载 systemctl
systemctl enable jupyter	# 配置 jupyter 开机自启	
systemctl start jupyter		# 启动 jupyter
systemctl status jupyter	# 查看 jupyter 运行情况
systemctl restar jupyter	# 重启 jupyter
```

3. 检测服务端口是否启动

```bash
netstat -na |grep 8888
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN
```

# 远程访问 Jupyter lab

首先启动服务端 `Jupyter lab` ，在服务器终端输入：

```bash
jupyter-lab --no-browser # no-browser 参数代表不需要打开浏览器
```

启动 `Jupyter` 后终端弹出的提示信息会包含在服务器访问的地址（`http://localhost:8888/lab?...`）

```bash
[C 2022-06-08 22:59:18.094 ServerApp]

    To access the server, open this file in a browser:
        file:///home/flmore/.local/share/jupyter/runtime/jpserver-3102-open.html
    Or copy and paste one of these URLs:
        http://localhost:8888/lab?token=d3f99bdebf9faab19d3cff4a809242da2cb61678c0daa275
     or http://127.0.0.1:8888/lab?token=d3f99bdebf9faab19d3cff4a809242da2cb61678c0daa275
```

这表明 Jupyter 启动成功，现在可以转到本地进行操作。

## 通过 IP 地址访问

将上面得到的链接中的 `localhost:8888` 替换为服务器的 IP 地址，然后把替换后的地址放到本地浏览器打开即可。

> 可以使用 `ip addr` 查看服务器 IP 地址

但如果服务器的 IP 地址非公网 IP ，这种方法就不能远程访问 Jupyter，这就需要使用下面介绍到的方法。

## 通过 SSH 端口转发

1. 在本地终端输入如下命令，通过建立 SSH 隧道实现把服务器的 `8888` 端口转发到本地 `8888` 端口上来。

```bash
ssh -N -f -L localhost:8888:localhost:8888 name@IP 
# 第一个 localhost 指本地，第二个指服务器，name 和 IP 指服务器用户名和地址
```

2. 在本地浏览器输入 `http://localhost:8888/lab` 即可。

## 路由端配置 ddnsto 访问

无意中发现 [DDNSTO](https://www.ddnsto.com/) 可以在路由端自动映射局域网内的 web 页面，而 `Jupyter` 从使用角度来讲就是一个 web 页面，因此就可以利用 `ddnsto` 轻松实现 `Jupyter` 的远程访问。

1. `ddnsto` 安装

我的路由器为 `RT-N56UB1-newifi3D2-512M` ，安装了[老毛子系统](https://t.me/s/pdcn1) ，系统自带 `ddnsto` ，可以直接进行配置，其他路由器的安装可参考[ddnsto 官方文档](https://doc.linkease.com/zh/guide/ddnsto/)。

2. 路由端配置 `ddnsto`

登录官网 [控制台 ](https://www.ddnsto.com/app/#/login)拿到“令牌”

![复制令牌](https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/20220625104008.png)

登录路由器后台做如下操作

![路由器端配置 ddnsto](https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/20220625105047.webp)

在官网控制台设置域名，将局域网内的 `Jupyter` 主机地址设为目标主机地址（我这里将目标地址设置为 `http://192.168.123.60:8888`），[详细设置参考](https://doc.linkease.com/zh/guide/ddnsto/#step3-在官网控制台设置域名)。

3. 远程登录 `Jupyter`

等待一分钟左右，用生成的 `kooldns.cn` 后缀的域名即可访问。出于安全考虑，首次登录需要微信认证，认证通过后就会出现 `Jupyter` 的登录界面。

![Jupyter login](https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/202206251122.png)

输入密码或 token 登录即可。

> token 获取：如果是将 `Jupyter` 配置为系统服务，可 ssh 登录服务主机后使用 `journalctl -u jupyter.service | grep token` 获取。

4. ddnsto 方法缺点

虽然通过这种方法配置十分简单，但 `ddnsto` 出于安全考虑只允许用户本人使用，也就是说，多人访问 `Jupyter` 的情景不适合这种访问方式。

# 参考文献

[1] [Project Jupyter | Installing Jupyter](https://jupyter.org/install)

[2] [Jupyter notebook/lab安装及远程访问 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/166425946)

[3] [使用systemctl注册jupyter为服务_FX-Felix的博客-CSDN博客](https://blog.csdn.net/qq_15983061/article/details/109912193)



