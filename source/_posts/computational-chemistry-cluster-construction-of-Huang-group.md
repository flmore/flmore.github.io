---
title: 课题组计算化学集群搭建
toc: true
date: 2022-10-30 12:31:32
tags: ["pbs", "cluster"]
categories: ["实验室"]
description: 本文记录了课题组计算化学集群的配置过程
---

# 系统安装

## 系统信息

```bash
[root@mn1 ~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

[root@mn1 ~]#
```



## 设置国内镜像源

设置清华镜像源

```bash
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo
```

更新软件包缓存

```bash
sudo yum makecache
```



## 网络设置

**修改主机名**

```bash
# permanent set
hostnamectl --static set-hostname newname
```

**更新** `/etc/hosts`

```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.123.200 mn1
192.168.123.210 cn1
```

**关闭 Selinux 和防火墙**

管理节点和计算节点均需关闭！

```bash
sed -i s/SELINUX=enforcing/SELINUX=disabled/ /etc/selinux/config
```

关闭防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
reboot
```

# 主机间无密码登录--SSH

以普通用户身份，在管理节点执行一下命令

```bash
ssh-keygen
cd ~/.ssh
cat id_rsa.pub > authorized_keys
chmod 600 authorized_keys # 一定要记得修改权限
```

# 主机间文件共享--NFS

## 服务器端

安装必要的包

```bash
yum -y install nfs-utils
```

启动服务

```bash
for i in rpcbind nfs; do echo starting $i ...; systemctl start $i; done
for i in rpcbind nfs; do echo check $i ...; systemctl status $i; done
for i in rpcbind nfs; do systemctl enable $i; done # auto-start
```

设置共享目录 `vim /etc/exports`

```bash
# shareDir ip(rw,no_root_squash,no_all_squash,sync)
# ip  192.168.0.0/24: 客户端 IP 范围，* 代表所有，即没有限制。
# rw: 权限设置，可读可写。
# sync: 同步共享目录。
# no_root_squash: 可以使用 root 授权。
# no_all_squash: 可以使用普通用户授权。

/home 192.168.123.0/24(rw,no_root_squash,sync,subtree_check)
/opt  192.168.123.0/24(rw,no_root_squash,sync,subtree_check)
```

重启服务

```bash
systemctl restart nfs
```

使用 `showmount -e localhost` 查看共享情况

```bash
Export list for localhost:
/opt  192.168.123.0/24
/home 192.168.123.0/24
```

## 客户端

安装必要的包

```bash
yum -y install nfs-utils
systemctl start rpcbind
systemctl enable rpcbind
```

> **客户端也不需要开启 NFS 服务，因为不共享目录。**

使用 `showmount -e mn1` 查看服务端（管理节点）nfs 情况：

```bash
Export list for mn1:
/opt  192.168.123.0/24
/home 192.168.123.0/24
```

挂载

```bash
mount mn1:/opt /opt
mount mn1:/home /home
```

添加到`/etc/fstab` 实现开机自动挂载

```bash
mn1:/home        /home        nfs    rw,defaults    0 0
mn1:/opt         /opt         nfs    rw,defaults    0 0
```

# 主机间用户同步--NIS

## 服务器端

安装必要的包

```bash
yum -y install ypserv ypbind yp-tools
```

设置域

```bash
echo "NISDOMAIN=cluster" >> /etc/sysconfig/network
```

启动服务

```bash
for i in ypserv rpcbind yppasswdd; do echo starting $i ...; systemctl start $i; done
for i in ypserv rpcbind yppasswdd; do echo check $i ...; systemctl status $i; done
for i in ypserv rpcbind yppasswdd; do systemctl enable $i; done # auto-start
```

更新用户信息数据库

```bash
/usr/lib64/yp/ypinit -m
# 新增账户时，需要更新数据库  
make -C /var/yp
```

## 客户端

安装必要的包

```bash
yum -y install ypbind yp-tools
```

设置域

```bash
echo "NISDOMAIN=cluster" >> /etc/sysconfig/network
```

设置用户认证信息读取顺序 `vim /etc/nsswitch.conf`

```bash
passwd:     nis files sss
shadow:     nis files sss
group:      nis files sss 
#initgroups: files sss

hosts:      nis files dns myhostname
```

设置NIS客户端

```bash
echo "domain cluster server mn1" >> /etc/yp.conf
```

修改系统认证文件`/etc/sysconfig/authconfig`，设置账号登入认证机制

```bash
sed -i 's/USENIS=no/USENIS=yes/g' /etc/sysconfig/authconfig
```

设置PAM授权

```bash
echo "session     optional      pam_mkhomedir.so skel=/etc/skel umask=077" >> /etc/pam.d/system-auth
```

启动服务

```bash
for i in rpcbind ypbind; do echo starting $i ...; systemctl start $i; done
for i in rpcbind ypbind; do echo check $i ...; systemctl status $i; done
for i in rpcbind ypbind; do systemctl enable $i; done # auto-start
```

## 客户端测试

`yptest` 测试

```bash
[flmore@cn1 ~]$ yptest
Test 1: domainname
Configured domainname is "cluster"

Test 2: ypbind
Used NIS server: mn1

Test 3: yp_match
WARNING: No such key in map (Map passwd.byname, key nobody)

Test 4: yp_first
flmore flmore:$6$YhSZb/BgQ4chyadg$o5vVBq2A7lx1jn.r7OvgBbsGio2FwqUSYhPy8N3Co.ScZWxUpqp6QEBpccsRkxYQzvBUhJCVP53asJQ3Iep1R/:1000:1000:flmore:/home/flmore:/bin/bash

Test 5: yp_next

Test 6: yp_master
mn1

Test 7: yp_order
1666941585

Test 8: yp_maplist
netid.byname
mail.aliases
protocols.byname
protocols.bynumber
services.byservicename
services.byname
rpc.bynumber
rpc.byname
hosts.byaddr
hosts.byname
group.bygid
group.byname
passwd.byuid
passwd.byname
ypservers

Test 9: yp_all
flmore flmore:$6$YhSZb/BgQ4chyadg$o5vVBq2A7lx1jn.r7OvgBbsGio2FwqUSYhPy8N3Co.ScZWxUpqp6QEBpccsRkxYQzvBUhJCVP53asJQ3Iep1R/:1000:1000:flmore:/home/flmore:/bin/bash
1 tests failed
```

`ypwhich` 查看NIS服务器是否正确

```bash
[flmore@cn1 ~]$ ypwhich
mn1
[flmore@cn1 ~]$ ypwhich -x
Use "ethers"    for map "ethers.byname"
Use "aliases"   for map "mail.aliases"
Use "services"  for map "services.byname"
Use "protocols" for map "protocols.bynumber"
Use "hosts"     for map "hosts.byname"
Use "networks"  for map "networks.byaddr"
Use "group"     for map "group.byname"
Use "passwd"    for map "passwd.byname"
```

# 编译器安装

# 并行计算环境配置

# 作业系统（PBS+maui配置）

## 管理节点

下载安装依赖环境和相关库文件

```bash
yum install libxml2-devel openssl-devel gcc gcc-c++ boost-devel libtool
```

解压 torque 安装包

```bash
tar -zxvf torque-6.1.2.tar.gz && cd torque-6.1.2
```

设置安装配置信息

```bash
./configure --prefix=/opt/torque/torque-6.1.2-gcc-4.8.5 --with-scp --with-default-server=mn1
#配置文件目录默认TORQUE_HOME=/var/spool/torque,可以指定为--with-server-home=/opt/torque/torque-6.1.2-gcc-4.8.5/share 
```

编译安装

```bash
make
make install
make packages
```

复制文件，设为开机启动：

```bash
cp contrib/init.d/{pbs_{server,sched,mom},trqauthd} /etc/init.d/
systemctl daemon-reload
libtool --finish /opt/torque/torque-6.1.2-gcc-4.8.5/lib
# for i in pbs_server pbs_mom trqauthd; do chkconfig --add $i; sudo chkconfig $i on; done
for i in pbs_server pbs_mom trqauthd; do systemctl enable $i; systemctl start $i; done
```

设置环境变量

```bash
vim /etc/profile.d/torque.sh

TORQUE_HOME=/opt/torque/torque-6.1.2-gcc-4.8.5
PATH=$TORQUE_HOME/bin:$TORQUE_HOME/sbin:$PATH
LD_LIBRARY_PATH=$TORQUE_HOME/lib:$LD_LIBRARY_PATH
```

```bash
source /etc/profile
```

torque 初始化

```bash
./torque.setup root
```

修改计算节点配置文件

```bash
vim $TORQUE_HOME/server_priv/nodes
# TORQUE_HOME具体位置由前面指定
#填入计算节点 np=核数 队列名， 如

mn1 np=56 batch
cn1 np=96 batch
```

```bash
vim /var/spool/torque/mom_priv/config
# add following two lines
$pbsserver mn1                                                                  
$logevent 255 
```

## 计算节点

登录计算节点安装 torque （**需注意NIS、NFS已配置好**）

```bash
cd /opt/torque/torque-6.1.2
./torque-package-clients-linux-x86_64.sh --install
./torque-package-mom-linux-x86_64.sh --install

vim /var/spool/torque/mom_priv/config
# add following two lines
$pbsserver mn1                                                                  
$logevent 255 

# restart pbs_mom.service and trqauthd.service
for i in pbs_mom trqauthd; do systemctl start $i;done
for i in pbs_mom trqauthd; do systemctl enable $i;done
```

## 配置PBS

管理节点增加队列

```bash
qmgr -c 'p s' # 输出当前队列配置
qmgr -c 'create queue batch' # 新增队列batch
qmgr -c 'set queue middle queue_type = Execution'
qmgr -c 'set queue middle resources_default.neednodes = middle' # 给队列分配分组为 middle 的节点
qmgr -c 'set queue middle resources_default.walltime = 1200:00:00' # 默认 walltime
qmgr -c 'set queue middle enabled = True' # 启用队列
qmgr -c 'set queue middle started = True' # 队列可以排队
qmgr -c "set queue middle acl_groups=zhaolab" # 限制特定用户组可以使用队列
qmgr -c "set queue middle acl_group_sloppy=true" # false 只查询用户的默认用户组; true: 查询用户所属的所有用户组
qmgr -c "set queue middle acl_hosts=node71" # 限制可以投递任务的节点
```

管理节点与计算节点的大小设为无限制

```bash
sudo sed -i '/END INIT INFO/s//&\nulimit -s unlimited/' /etc/rc.d/init.d/pbs_mom
sudo sed -i '/LimitSTACK/s/=.*/=infinity/' /usr/lib/systemd/system/pbs_mom.service
```

## MAUI配置

在服务节点上安装maui

```bash
./configure --with-pbs=/opt/torque/torque-6.1.2-gcc-4.8.5 --prefix=/usr/local/maui
make -j2
make install
cp etc/maui.sh /etc/profile.d/
source /etc/profile
sed -i '/^MAUI_PREFIX/s/=.*/=\/usr\/local\/maui/' contrib/service-scripts/redhat.maui.d
sed -i '/daemon/s/--user maui/--user root/' contrib/service-scripts/redhat.maui.d
cp contrib/service-scripts/redhat.maui.d /etc/init.d/maui
chmod a+x /etc/init.d/maui
chkconfig --add maui
chkconfig maui on
service maui start
```

## MAUI 修改用户资源限额

配置文件地址 `/usr/local/maui/maui.cfg`

```bash
# 默认用户限制
USERCFG[DEFAULT]  MAXJOB=40  MAXPROC=80  MAXNODE=20
# 对 jipfnew 账户单独进行设置
USERCFG[jipfnew]  MAXJOB=100 MAXPROC=300 MAXNODE=20
```

修改完配置后，重启 MAUI 服务使配置生效

```bash
service maui restart
```

# 参考文献

[1] [centos | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/centos/)

[2] [从0搭建Centos7 计算集群 (cndaqiang.github.io)](https://cndaqiang.github.io/2019/09/19/Centos7-CC19/)

[3] [CentOS7 配置 Torque + MAUI 作业调度系统 | Kevinzjy's Blog](https://kevinzjy.github.io/2019/06/14/CentOS_Gridview/)

[4] [CentOS下安装PBS+maui教程 - 高性能计算、集群、并行技术 (HPC , Cluster , Parallel Computing) - 计算化学公社 (keinsci.com)](http://bbs.keinsci.com/thread-20943-1-1.html)

[5] 李会民. (2015, November 29). *Linux高性能计算集群配置*.

[6] 张鋆. (2016). *计算化学集群构建教程*. http://www.zhjun-sci.com/docs/others/computchemclusterbuilding.pdf



