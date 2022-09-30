---
title: 一行命令删除 VASP 输出文件
toc: true
date: 2022-09-30 08:17:47
tags: ["vasp"]
categories: ["software"]
description: vasp 的输出结果通常包含很多文件，这些文件占用了大量硬盘空间，做完分析后我们可以将这些文件删除来节省空间，本文记录了快速删除这些文件的命令。
---

# 命令内容

```bash
rm -f CHG* CONTCAR* DOSCAR* DYNMAT EIGENVAL IBZKPT OPTIC OSZICAR* OUTCAR* PROCAR* \
      PCDAT W* XDATCAR* PARCHG* vasprun.xml SUMMARY.* REPORT \
      wannier90.win wannier90_band.gnu wannier90_band.kpt wannier90.chk wannier90.wout \
      *.dat plotfile p4vasp.log \
      *.e[0-9]* *.o[0-9]* *.pe[0-9]* *.po[0-9]* *.err *.out
```

# 参考文献

https://www.vasp.at/
