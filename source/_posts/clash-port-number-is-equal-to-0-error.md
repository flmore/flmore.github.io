---
title: Clash for Windows 端口号显示为 0 错误
toc: true
date: 2022-06-14 09:55:28
tags: [Clash, "network issue"]
categories: [software]
description:  Windows 开机后发现无法上网，检查 clash for windows 发现监听端口为 0。 
---

![Clash for Windows 端口号为 0 错误](https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/20220614100032.png)

检查 Clash 运行日志： `C:\Users\<username>\.config\clash\logs`  ，发现这行报错：

```log
level=error msg="Start Mixed(http and socks) server error: listen tcp 127.0.0.1:7890: bind: An attempt was made to access a socket in a way forbidden by its access permissions."
```

# 解决方案

管理员身份运行 CMD 执行下面的命令

```cmd
netsh int ipv4 set dynamicport tcp start=49152 num=16383
netsh int ipv4 set dynamicport udp start=49152 num=16383
# https://blog.csdn.net/tian2342/article/details/108934646

netsh int ipv4 set dynamic tcp start=49152 num=16384
# https://github.com/Fndroid/clash_for_windows_pkg/issues/671
```

之后检查结果

```cmd
netsh int ipv4 show dynamicport tcp
```

端口恢复正常后重启计算机。

# 参考文献

[1] [解决 Clash for windows 端口为 0 导致无法使用 - Frytea's Blog](https://blog.frytea.com/archives/564/)

[2] [关于Windows端口没被占用提示An attempt was made to access a socket in a way forbidden by its access permissions_tian2342的博客-CSDN博客](https://blog.csdn.net/tian2342/article/details/108934646)

