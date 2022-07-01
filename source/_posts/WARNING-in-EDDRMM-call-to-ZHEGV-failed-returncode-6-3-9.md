---
title: 'VASP 错误 WARNING in EDDRMM: call to ZHEGV failed, returncode = 6 3 9'
toc: true
date: 2022-07-01 14:06:43
tags: ["vasp"]
categories: ["ComputMat"]
description: "本文记录了vasp错误输出 EDDRMM: call to ZHEGV failed 的错误原因及解决方案"
---

# 前言

使用 `vasp` 对完全脱锂的镍锰酸锂做结构优化时报错，报错信息为 `WARNING in EDDRMM: call to ZHEGV failed, returncode = 6 3 9`

INCAR 文件如下

```text
Global Parameters
ISTART =  1            (Read existing wavefunction; if there)
ENCUT  =  520       (Cut-off energy for plane wave basis set, in eV)
PREC   =  Normal       (Precision level)
LWAVE  = .FALSE.        (Write WAVECAR or not)
LCHARG = .FALSE.        (Write CHGCAR or not)
ADDGRID= .TRUE.        (Increase grid; helps GGA convergence)
LASPH = .TRUE.
NPAR = 4
ALGO    = FAST
ISPIN    = 2

# d orbital
LMAXMIX  = 4

# mag system
AMIX = 0.2
BMIX = 0.0001
AMIX MAG = 0.8
BMIX MAG = 0.0001
LREAL= Auto

Electronic Relaxation
ISMEAR =  0            (Gaussian smearing; metals:1)
SIGMA  =  0.05         (Smearing value in eV; metals:0.2)
NELM   =  90           (Max electronic SCF steps)
NELMIN =  6            (Min electronic SCF steps)
EDIFF  =  1E-05        (SCF energy convergence; in eV)
# GGA  =  PS           (PBEsol exchange-correlation)

Ionic Relaxation
NSW    =  500          (Max ionic steps)
IBRION =  2            (Algorithm: 0-MD; 1-Quasi-New; 2-CG)
ISIF   =  3            (Stress/relaxation: 2-Ions, 3-Shape/Ions/V, 4-Shape/Ions)
EDIFFG = -1E-02        (Ionic convergence; eV/AA)
# ISYM =  2            (Symmetry: 0=none; 2=GGA; 3=hybrids)


# GGA+U
LDAU = .TRUE.
LDAUTYPE = 2
LDAUL = 2 2 -1
LDAUU = 4.84 6.00 0.00
LDAUJ = 0.00 0.00 0.00
LDAUPRINT = 2
```

# 错误原因

`vasp` 官方论坛给出了几种可能的原因

> the error is due to a LAPCK call (ZHEGV):
> ZHEGV computes all the eigenvalues, and optionally, the eigenvectors of a complex generalized Hermitian-definite eigenproblem .
> there may be several reasons for that error:
>
> 1. the RMM-DIIS diagonalisation algorithm is not stable for your specific setup of the calculation.
>      --> use ALGO = Normal (blocked Davidson) or ALGO = Fast (5 steps blocked Davidson, RMM-DIIS)
>
> 2. a) maybe your input geometry was not reasonable (error occurs at the very first ionic step, please have a look for the geometry data of your run in OUTCAR ) or 
>
>      b) the last ionic relaxation step lead to an unreasonable geometry (compare the input and output geometries of the last ionic relaxation steps in XDATCAR).
>      In that case (2b) it can be helpful to
>        --> switch to a different relaxation algorithm (IBRION-tag)
>        --> reduce the step size of the first step by setting POTIM smaller than the default value
>
> 3. The installation of the LAPACK on your machine was not done properly:
>     use the LAPACK which is delivered with the code
>     (vasp.4.lib/lapack_double.o)
>
> 4. If the error persist although you switched to the Davidson algorithm:
>     on some architectures (especially SGI) some LAPACK routines are not working properly. However, it is possible to avoid the usage of the ZHEGV subroutine by commenting the line
>     #define USE_ZHEEVX
>     in davidson.F, subrot.F, and wavpre_noio.F and recompiling VASP.
>
> [https://www.vasp.at/forum/viewtopic.php?t=214](https://www.vasp.at/forum/viewtopic.php?t=214)

大致意思为 `RMM-DIIS` 算法对于这个体系的计算仍不是很稳定（即 INCAR 中 ALGO 设置为 FAST 导致）；初始结构不太合理；`LAPACK` 安装有问题。

# 解决方案

主要参考了科音论坛的[这篇帖子](http://bbs.keinsci.com/thread-508-1-1.html) ，把 INCAR 中 `ALGO` 的值改为 `VeryFast` 即可。

修改 INCAR 后重新提交，就没有类似的报错了。
