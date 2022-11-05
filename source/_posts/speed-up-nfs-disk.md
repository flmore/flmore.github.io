---
title: 优化NFS传输速度
toc: true
date: 2022-11-05 20:59:38
tags: ["network issue", "cluster"]
categories: ["Other"]
description: "默认情况NFS挂载方式会极大降低数据传输速度，导致VASP等程序在计算节点和管理节点上的运行速度有很大差距，本文记录了如何更改NFS服务器写入规则，优化NFS数据传输速度。"
---

# 数据传输速度测试

**本地磁盘速度测试**

```bash
time dd if=/dev/zero of=./SPEED bs=8k count=102400 # 测试写速度

102400+0 records in
102400+0 records out
838860800 bytes (839 MB) copied, 0.585492 s, 1.4 GB/s

real    0m0.591s
user    0m0.068s
sys     0m0.521s
```

写入 1G 数据量，其速度为 1.4GB/s。

**NFS挂载磁盘速度测试**

```bash
time dd if=/dev/zero of=./SPEED bs=8k count=102400

102400+0 records in
102400+0 records out
838860800 bytes (839 MB) copied, 12.4746 s, 67.2 MB/s

real    0m12.480s
user    0m0.014s
sys     0m0.622s
```

NFS挂载磁盘的速度比本地磁盘慢了10倍有余。

# NFS数据传输优化

更改NFS服务器写入规则，将同步写入内存和磁盘(sync)，改为先写入内存，后写入磁盘(async)。

```bash
$TARGET_PATH   192.168.1.0/24(rw,async,no_root_squash)
```

重启NFS服务，客户端不需要做配置

```bash
exportfs -rv
# 先启动rpcbind,再启动nfs
systemctl restart rpcbind
systemctl restart nfs
```

**再次测试NFS挂载磁盘速度**

```bash
time dd if=/dev/zero of=./SPEED bs=8k count=102400

102400+0 records in
102400+0 records out
838860800 bytes (839 MB) copied, 7.58768 s, 111 MB/s

real    0m7.590s
user    0m0.011s
sys     0m0.602s
```

可以看到，更改数据传输方式后，网络磁盘的数据传输速度得到了提升，但这种方式对VASP运行速度的影响有待进一步测试。
