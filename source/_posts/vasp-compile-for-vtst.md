---
title: VASP 5.4.4 + VTST 编译安装
toc: true
date: 2022-07-21 13:48:42
tags: ["vasp"]
categories: ["software"]
description: "VTST是VASP的过渡态工具，本文记录了为 VASP 添加 VTST 功能的编译安装过程。"
---

# VTST 介绍

VTST(Transition State Tools for VASP)是VASP的过渡态工具，官网地址 [http://theory.cm.utexas.edu/vtsttools/index.html](http://theory.cm.utexas.edu/vtsttools/index.html) 。

# 编译安装过程

## 1. 解压 vasp 源码

```bash
tar zxvf vasp.5.4.4.tgz
cd vasp.5.4.4
```

## 2. 获取 vtst 源码

vtst 源码下载地址 http://theory.cm.utexas.edu/code/vtstcode-184.tgz

```bash
wget http://theory.cm.utexas.edu/code/vtstcode-184.tgz # download vtst code
tar zxvf vtstcode-184.tgz
```

## 3. 修改 vasp 源码

```bash
cp src/chain.F scr/chain.F.bak # bakup chain.F file
cp vtstcode-184/vtstcode5/* src
```

src/main.F 中需要把

```text
CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
     LATT_CUR%A,LATT_CUR%B,IO%IU6)
```

替换为

```text
CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
     TSIF,LATT_CUR%A,LATT_CUR%B,IO%IU6)
```

修改编译配置 src/.objects ，在chain.o前（大概第67行）添加如下内容：

```
bfgs.o dynmat.o  instanton.o  lbfgs.o sd.o   cg.o dimer.o bbm.o \
fire.o lanczos.o neb.o  qm.o opt.o \
```

## 4. 加载编译器环境

```bash
source /opt/intel/oneapi/setvars.sh intel64
```

使用 `which ifort` 查看 `ifort` 路径，屏幕会输出类似内容：

```bash
/opt/intel/oneapi/compiler/2021.3.0/linux/bin/intel64/ifort
```

同理，使用 `echo $MKLROOT` 查看 Intel MKL 环境：

```bash
/opt/intel/oneapi/mkl/2021.3.0
```

## 5.执行编译

```bash
cp arch/makefile.include.linux_intel ./makefile.include
make all
```

# 参考文献

[1] [VASP 5.4.1+VTST编译安装 (ustc.edu.cn)http://theory.cm.utexas.edu/vtsttools/installation.html)

[2] [Transition State Tools for VASP — Transition State Tools for VASP (utexas.edu)](http://theory.cm.utexas.edu/vtsttools/index.html)

[3] [Installing VASP.5.X.X - Vaspwiki](https://www.vasp.at/wiki/index.php/Installing_VASP.5.X.X)





