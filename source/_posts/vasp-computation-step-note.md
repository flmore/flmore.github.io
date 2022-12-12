---
title: 常用VASP计算步骤备忘
toc: true
date: 2022-12-10 22:23:55
tags: ["vasp", "vaspkit", "transition state"]
categories: ["ComputMat"]
description: 如题，本文记录了一些VASP常见计算的计算步骤。如过渡态、虚频、电荷差分、Bader电荷等的计算。
---

# 过渡态计算  CINEB方法

复制始末态的CONTCAR为POSCAR1和POSCAR2

```bash
cp KPOINTS POTCAR INCAR job.pbs
```

> 查看两个POSCAR中的元素是否为一一对应关系  且选择了相同的固定方法

```bash
dist.pl POSCAR1 POSCAR2   #查看距离
```

使用 `nebmake.pl` 插点

```bash
nebmake.pl POSCAR1 POSCAR2 X 
# X 为插点个数，一般取距离/0.8，且必须被核心数整除
```

检查新建的目录，查看POSCAR的固定情况

```bash
for x in ??; do (cd $x; ln -sv ../POTCAR); done # 将POTCAR复制进入各目录
for x in ??; do (cd $x; vasp-pos-to-cif5 POSCAR); done > t.cif
```

使用 Jmol 观察`t.cif` 检查插点是否合理
修改INCAR

```bash
IBRION = 1 		# 不可等于2
POTIM = 0.2		# =0.1
IMAGES = X 		# 插点数  
SPRING = -5   
LCLIMB = .T.	# 需要爬坡
```

修改 `job.pbs` 中的资源数

```bash
nodes = 2 or  ppn  = 48 
```

复制一个OUTCAR进入始末文件夹下

提交计算任务

```bash
nebef.pl # 查看能量
for x in ??; do (cd $x; vasp-pos-to-cif5 CONTCAR) ; done > t.cif
# 检查计算结果 输出t.cif文件
```

# 虚频计算

`mkdir freq`新建freq目录
将过渡态计算中能量最高点目录下的CONTCAR拷贝进新建的目录 
固定除了气体分子以外的其他原子 命名为POSCAR
复制KPOINTS   POTCAR  INCAR  job.pbs 
对INCAR进行设置

```bash
IBRION=5 
POTIM=0.015 
#NPAR=4 关掉过渡态计算
```

提交任务
	

```bash
grep cm OUTCAR # 查看虚频计算结果
vasp-freq-to-xyz5 OUTCAR > f.cif
```

Jmol打开    开启震动  向量

# ORR反应

H2优化​

```bash
ISPIN = 0
```

优化
​直接新建slab o2-ads ooh-ads o-ads oh-ads  
​1.23 eV 能量矫正(存在电子转移的步骤)
​自由能矫正

```bash
zpe  cp CONTCAR  ./zpe  # 弛豫气体分子固定表面
```

INCAR设置

```bash
IBRION  = 5
POTIM   = 0.015
NFREE = 2
#NPAR = 2
```

输出矫正值
	气体分子(O2分子需要外推)
		vaspkit   5   502   298.15  大气压  1
	h2o(l)  #液态
		vaspkit   5   502   298.15  0.035   1  #矫正到饱和蒸汽压下的气态水 即为液态时的自由能
	step
		vaspkit   5   501   298.15

# 电荷差分

total自洽		

```bash
EDIFF   = 1E-6
NSW     = 0
EDIFFG  = -0.05
IBRION  = -1
POTIM   = 0.1
LWAVE = .F.
LAECHG = .T.
LCHARG = .T.
```

CONTCAR拆分成slab gas分别自洽
Chgdiff Visualization
获得CHGCAR-all CHGCAR-slab.vasp CHGCAR-gas.vasp 置入同一chgdiff文件夹
打开VESTA, 拖入CHGCAR-all
Edit--->Edit Data--->Volumetric Data
Import两个.vasp文件--->Operation--->Subtract from current data--->OK
Style下的Properties 
调整晶胞边界线
调整电荷密度等高线
电荷密度色彩
File--->Export Raster Image--->Scale x  输入3--->勾选背景透明--->OK

# Bader电荷

​	INCAR参数设置		

```bash
EDIFF   = 1E-6
NSW     = 0
EDIFFG  = -0.05
IBRION  = -1
POTIM   = 0.1
LWAVE = .F.
LAECHG = .T.
LCHARG = .T.
```

​	

```bash
chgsum.pl AECCAR0 AECCAR2
```

```bash
bader CHGCAR -ref CHGCAR_sum
```

读取ACF.dat文件

```bash
grep VRHFIN OUTCAR      # 输出原子的价电子层
```

