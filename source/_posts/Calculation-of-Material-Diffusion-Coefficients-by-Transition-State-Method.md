---
title: 过渡态方法计算材料扩散系数
toc: true
date: 2022-10-04 16:17:37
tags: ["vasp", "transition state", "diffusion coefficient"]
categories: ["ComputMat"]
mathjax: true
description: 本文记录了过渡态方法计算扩散系数的公式推导，并给出了对应的Python计算脚本。
---

# 计算原理

阿仑尼乌斯公式(Arrhenius equation)不但揭示了化学 反应的速率常数与温度之间的关系, 同时也适用于表示扩散系数与温度间的关系。扩散系数的阿仑尼乌斯表达形式为：
$$
D=D_0\exp(-E_a/k_BT)
$$
式中 $$ D_0 $$ 为扩散常数, $$ E_a $$ 为扩散势垒, $$ k_B $$ 和 $$ T $$分别为玻尔兹曼常数和绝对温度, $$ k_B = 1.38 × 10^{−23} J·K^{−1} $$ 。

根据过渡态理论, 原子在固体中的跃迁率表达式为
$$
v=\nu\exp(-E_a/k_BT)
$$
式中 $$ \nu $$ 为原子的振动频率。由原子跃迁率, 可以把扩散系数表示为
$$
D=l^2\nu\exp(-E_a/k_BT)
$$
式中 $$ l $$ 为原子每次跃迁的距离, 通过对原子在跃迁过程中始末位置进行测量, 即使用 `dist.pl IS/POSCAR FS/POSCAR`得出的距离。根据温特-齐纳理论，振动频率 $$\nu$$ 可以近似地表示为
$$
\nu=(2E_a/ml^2)^{1/2}
$$
式中 $$m$$ 为单一原子的质量。

将以上求得的 $$l, \nu, E_a$$ 值并代入扩散系数计算公式, 即可得到原子在晶格中的扩散系数。

# 计算脚本

设 $$C$$ 在 $$V$$ 中的扩散能垒为 0.89 $$eV$$，$$C$$ 原子扩散距离为  2.141 $$\overset{\circ}{A}$$ ，$$C$$ 原子质量为 $$1.993×10^{−26} kg$$，则 500 $$K$$ 下的扩散系数计算脚本如下：

```python
import math


def calcDiffusion(l, m, Ea, T):
    Ea_iso = Ea*1.60217663e-19 # Converting Energy to SI
    Kb = 1.380649e-23 # The value of Boltzmann constant, J/K.
    
    v = ((2*Ea_iso)/(m*l**2))**(1/2) # vibration frequency.
    exp = math.exp(-Ea_iso/(Kb*T))
    
    D0 = v*l**2 # diffusion constant. Unit: (m^2/s)
    D = D0*exp # diffusion coefficient. Unit: (m^2/s)
    
    print("D0: {}, D: {}".format(D0, D))
    return((D0, D))


l = 2.14e-10 # The length of the migration path in m.
m = 1.99e-26 # The mass of atom in kg.
Ea = 0.89 # The migration barrier in eV.
T = 500 # absolute temperature in K.

calcDiffusion(l, m, Ea, T)
```

输出结果为：

```text
D0: 8.101257427262206e-07, D: 8.66457539553957e-16
```



# 参考文献

[1] Yang Biao, Wang Li-Ge, Yi Yong, Wang En-Ze, Peng Li-Xia. First-principles calculations of the diffusion behaviors of C, N and O atoms in V metal. *Acta Phys. Sin*., **2015**, 64(2): 026602. doi: 10.7498/aps.64.026602

[2] Said Oukahou, Mohammad Maymoun, Abdelali Elomrani, Khalid Sbiaai, and Abdellatif Hasnaoui. Enhancing the Electrochemical Performance of Olivine LiMnPO4 as Cathode Materials for Li-Ion Batteries by Ni–Fe Codoping. *ACS Applied Energy Materials* **2022** *5* (9), 10591-10603. DOI: 10.1021/acsaem.2c01319
