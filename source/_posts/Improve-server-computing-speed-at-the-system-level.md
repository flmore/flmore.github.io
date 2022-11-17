---
title: 系统级别优化服务器计算速度
toc: true
date: 2022-11-17 22:12:57
tags: ['vasp', 'cluster']
categories: ['ComputMat']
description: 计算集群搭建完成后，最好做一些系统层面的优化，使得程序跑的更快，这里搜集了一些常用的优化方法以备查。
---

# 设置CPU Performance 模式

## 简介

CPU 动态节能技术用于降低服务器功耗，通过选择系统空闲状态不同的电源管理策略，可以实现不同程度降低服务器功耗，更低的功耗策略意味着 CPU 唤醒更慢对性能影响更大。但对于对时延和性能要求高的应用，可以关闭 CPU 的动态调节功能，禁止 CPU 休眠，并把 CPU 频率固定到最高，以达到最好的性能。通常建议在服务器 BIOS 中修改电源管理为 Performance，如果发现 CPU 模式为 conservative 或者 powersave，可以使用 cpupower 设置 CPU Performance 模式，效果也是相当显著的。

## cpufreq 可用的几种模式

```bash
# 查看CPU频率可选模式
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
conservative userspace powersave ondemand performance

# 查看当前的调节器
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
performance

# 查看频率信息
cpupower frequency-info
```

**conservative** ：该策略平滑地调整 CPU 频率，频率的升降是渐变式的, 会自动在频率上下限调整，和 ondemand 的区别在于它会按需分配频率，而不是一味追求最高频率。

**userspace**：系统将变频策略的决策权交给了用户态应用程序，并提供了相应的接口供用户态应用程序调节 CPU 运行频率使用。也就是长期以来都在用的那个模式。可以通过手动编辑配置文件进行配置。

**powersave**：将 CPU 频率设置为最低的所谓 “省电” 模式，CPU 会固定工作在其支持的最低运行频率上。

**ondemand**：按需快速动态调整 CPU 频率， 一有 cpu 计算量的任务，就会立即达到最大频率运行，等执行完毕就立即回到最低频率。

**performance**: 该策略只注重效率，将 CPU 频率固定工作在其支持的最高运行频率上，而不动态调节。

## 使用cpupower 设置CPU策略

**安装相关工具**

```bash
yum install kernel-tools
```

**命令行设置**

```bash
# 设置节点上所有的CPU为performance策略
cpupower -c all frequency-set -g performance

# 设置节点上所有的CPU位posersave策略
cpupower -c all frequency-set -g powersave
```

# 解除系统资源限制

## 简介

在linux系统使用过程中，默认的系统设置足够使用，但是对于一些高并发高性能的程序会有瓶颈存在，这些限制主要通过ulimit查看和修改。

## 查看当前账户限制

```bash
ulimit -a

core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 766271
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

其中，

“core file size”（coredump记录文件大小，默认为0不记录）。

“open files”（进程打开文件最大数量，默认1024，网络连接较多时会存在瓶颈）。

“max user processes”（用户最大进程数，多进程程序修改）。

## 设置

ulimit资源设定的修改分硬限制和软限制，软限制无法超过硬限制的上限，硬限制设定需要修改系统配置文件。

```bash
vi /etc/security/limits.conf
#Each line describes a limit for a user in the form:
#
#<domain> <type> <item> <value>
#
#Where:
#<domain> can be:
# - a user name
# - a group name, with @group syntax
# - the wildcard *, for default entry
# - the wildcard %, can be also used with %group syntax,
# for maxlogin limit
#
#<type> can have the two values:
# - "soft" for enforcing the soft limits
# - "hard" for enforcing hard limits
#
#<item> can be one of the following:
# - core - limits the core file size (KB)
# - data - max data size (KB)
# - fsize - maximum filesize (KB)
# - memlock - max locked-in-memory address space (KB)
# - nofile - max number of open file descriptors
# - rss - max resident set size (KB)
# - stack - max stack size (KB)
# - cpu - max CPU time (MIN)
# - nproc - max number of processes
# - as - address space limit (KB)
# - maxlogins - max number of logins for this user
# - maxsyslogins - max number of logins on the system
# - priority - the priority to run user process with
# - locks - max number of file locks the user can hold
# - sigpending - max number of pending signals
# - msgqueue - max memory used by POSIX message queues (bytes)
# - nice - max nice priority allowed to raise to values: [-20, 19]
# - rtprio - max realtime priority
#
#<domain> <type> <item> <value>
* soft core unlimited
* hard core unlimited
* soft data unlimited
* hard data unlimited
* soft fsize unlimited
* hard fsize unlimited
* soft sigpending unlimited
* hard sigpending unlimited
* soft nofile 65536
* hard nofile 65536
* soft msgqueue unlimited
* hard msgqueue unlimited
* soft nproc 65536
* hard nproc 65536
* soft locks unlimited
* hard locks unlimited
* soft memlock unlimited
* hard memlock unlimited
```

修改账户启动执行脚本

```bash
vim ./bash_profile
ulimit -c unlimited
ulimit -d unlimited
ulimit -f unlimited
ulimit -i unlimited
ulimit -n 65536
ulimit -q unlimited
ulimit -u 65536
ulimit -x unlimited
ulimit -l unlimited
```

重启节点使配置生效。

# 升级内核

```bash
uname -r    #查看内核版本，若内核版本低于4.0则需要进行内核升级

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org #导入ELRepo源公共密钥

rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm #安装ELRepo源

yum --enablerepo=elrepo-kernel install kernel-ml #安装最新linux内核

grub2-mkconfig -o /boot/grub2/grub.cfg #重建内核启动配置文件

grub2-set-default 0 #设置新安装的内核默认启动（新安装的内核在内核顺序中一般为0）

reboot #重启后用uname -r验证内核是否升级成功
```

> 该优化方法未在组内服务器上测试过

# 参考文献

[1] [CPU 优化建议使用 cpupower 设置 CPU Performance 模式 | HelloDog (wsgzao.github.io)](https://wsgzao.github.io/post/cpupower/)

[2] [centos7系统资源限制整理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/570778523?utm_id=0)

[3] [AMD EPYC-7742（ZEN2）计算性能调优](https://mp.weixin.qq.com/s/x69LutkS_JruRhRDk1wgGw)

