---
title: 课题组 HPC 计算服务器 vaspkit 配置
toc: true
date: 2022-10-15 22:05:22
tags: ["vasp", "vaspkit"]
categories: ["实验室"]
description: vasp与vaspkit配合使用将极大的提高效率。但默认情况下 vaspkit 会安装在用户目录下。如果很多用户都会使用到 vaspkit，那么将其安装在公共目录下更加合适。本文记录了如何在自己的账号中配置 vaspkit。
---

1. 在家目录下新建`.vaspkit` 文件

```bash
vim ~/.vaspkit
```

2. 进入编辑模式，粘贴如下内容：

```bash
# cp how_to_set_environment_variable ~/.vaspkit and modify the ~/.vaspkit file based on your settings!
VASP5                         .TRUE.                         # .TRUE. or .FALSE.; Set .FALSE. if you are using vasp.4.x
LDA_PATH                      ~/POTCAR/LDA                   #  Path of LDA potential.
PBE_PATH                      /opt/vasp/pseudopotential/PAW.PBE.54/                   #  Path of PBE potential.
GGA_PATH                      ~/POTCAR/GGA                   #  Path of PW91 potential.
POTCAR_TYPE                    PBE                           #  PBE, PW91 or LDA; Set PBE if you want to make PBE-POTCAR file
GW_POTCAR                     .FALSE.                        # .TRUE. or .FALSE.; For example, H_GW, O_GW will be chose when POTCAR_GW = .TRUE.
RECOMMENDED_POTCAR            .TRUE.                         # .TRUE. or .FALSE.; The recommended PAW potential will be chose when RECOMMENDED_POTCAR = .TRUE.
SET_FERMI_ENERGY_ZERO         .TRUE.                         # .TRUE. or .FALSE.; The Fermi Energy will be set to zero eV when SET_FERMI_ENERGY_ZERO = .TRUE.
MINI_INCAR                    .FALSE.                        # .TRUE. or .FALSE.; A simplified INCAR will be written when MINI_INCAR = .TRUE.
USER_DEFINED_INCAR            .FALSE.                        # .TRUE. or .FALSE.; whether to use embedded INCAR templates or user defined INCAR templates
WRITE_SELECTIVE_DYNAMICS      .FALSE.                        # .TRUE. or .FALSE.; whether the selective dymanics set will be forced to write when SET_SELECTIVE_DYNAMICS_MODE = .FALSE.
PYTHON_BIN                     /opt/anaconda3/bin/python3       #  Python executable program with its installation path. I recommend you install anaconda package for Python data science
PLOT_MATPLOTLIB               .FALSE.                        # .TRUE. or .FALSE.; Set .TRUE. if you want to generate Graphs. (Matplotlib and Numpy packages MUST be embedded in Python)
VASPKIT_UTILITIES_PATH        /usr/local/vaspkit/utilities            #  IF ADVANCED_USER is .TRUE., set VASPKIT_UTILITIES_PATH like ~/vaspkit.0.72/utilities in order to use scripts in it.
ADVANCED_USER                 .TRUE.                         # .TRUE. or .FALSE.; Please fill in your settings in the block 'USER_DEFINED' if you want vaspkit to integrate your own scripts in the 'UTILITIES' file.
SET_INCAR_WRITE_MODE           OVERRIDE                      #  OVERRIDE, APPEND, BACK-UP-OLD,BACK-UP-NEW;  "Customize INCAR File"  whether to override existing INCAR/appending existing INCAR/backup existing INCAR to INCAR.old && write into INCAR/write into INCAR.new
PHS_CORRECTION                .FALSE.                        # .TRUE. or .FALSE.; whether to make PHS correction during linear optical calculations. More details on this correction are given in Comput. Mater. Sci. 172 (2020) 109315.

# Reset the default values of variables in here
SYMPREC                        1E-5                          # Distance tolerance in Cartesian coordinates to find crystal symmetry (default value: 1E-5)
EMIN                          -20.0                          # Minimum energy for evaluation of DOS (default value: -20.0 eV)
EMAX                           20.0                          # Maximum energy for evaluation of DOS (default value:  20.0 eV)
NEDOS                          2001                          # Number of grid points in DOS (default value: 2001)
GAMMA_CENTERED                .TRUE.                         # .TRUE. or .FALSE.; (default value: .TRUE.)
VACUUM_THICKNESS               15.0                          # The thickness of vacuum to build slab or 2D materials (default value: 10 angstrom)
CENTER_SLAB                   .TRUE.                         # Center the slab in the z direction; (default value: .TRUE.)
```

3. 保存并退出
4. 开始愉快地使用`vaspkit` 做前处理和后处理。
