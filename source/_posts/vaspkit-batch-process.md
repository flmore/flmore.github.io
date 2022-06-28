---
title: vaspkit 批量提取态密度数据
toc: true
date: 2022-06-07 16:01:19
tags: [vasp, vaspkit, "batch process"]
categories: ["ComputMat"]
description: vaspkit 可以很方便地帮助我们处理 vasp 的计算结果，但如果同时计算了很多个结构，每个结构的后处理步骤相同或相似，就可以结合 shell 或其他编程语言批量处理，提高效率。这里记录一下我在提取态密度数据时所用到的批量处理命令。
---

# 提取总态密度（TDOS）和投影态密度（PDOS）

## vaspkit 提取态密度

vaspkit 中与态密度相关的功能代码为 `11*` ，具体如下（[官方文档](https://vaspkit.com/tutorials.html#extract-and-output-dos-and-pdos)）

| 代码 | 功能                                                         |
| ---- | ------------------------------------------------------------ |
| 111  | Get the total DOS.                                           |
| 112  | Output projected DOS for selected atoms to separate files.   |
| 113  | Output projected DOS for every elements to separate files.   |
| 114  | Output sum of projected DOS for selected atoms to one file.  |
| 115  | Output sum of projected DOS for selected atoms and orbitals to one file. |

输入不同代码使用不同功能，根据界面提示输入信息完成计算结果的处理。

## 单个计算快速提取态密度

以尖晶石型锰酸锂为例，DOS 计算结束后，可以通过如下命令提取总态密度信息，并将屏幕输出保存至包含当前日期的日志文件：

```bash
(echo '111') | vaspkit  > vaspkit-extract-dos-dat_`date +%Y%m%d`.log
```

命令执行后，当前目录下的 `TDOS.dat` 文件即可用于绘制 DOS 图。

提取指定原子和轨道的投影态密度要稍复杂些，如提取 `Mn` 的 `3d` 轨道和 `O` 的 `2p` 轨道，可以使用如下命令：

```bash
(echo '115'; echo 'Mn'; echo 'd'; echo 'O'; echo 'p'; echo -e '\n') | vaspkit > vaspkit-extract-dos-dat_`date +%Y%m%d`.log
```

vaspkit 最后一个输入参数 ` echo -e '\n'` 模拟回车键，退出 vaspkit 选择界面。

如果要同时提取 TDOS 和 PDOS，需要把第二个屏幕输出重定向改为追加模式，否则前一个的屏幕输出会被覆盖。

```bash
(echo '111')| vaspkit  > vaspkit-extract-dos-dat_`date +%Y%m%d`.log
(echo '115'; echo 'Mn'; echo 'd'; echo 'O'; echo 'p'; echo -e '\n') | vaspkit >> vaspkit-extract-dos-dat_`date +%Y%m%d`.log
```

> `date +%Y%m%d` 可以生成当前年月日 `20220607` ，`date +%T` 可以生成当前时间 (如 `17:23:07` ），`date +%H%M%S` 可以生成当前时间对应的时分秒 （如 `172307`）

## 多个相似计算提取态密度

以上介绍了对于一个计算结果，如何快速提取态密度数据，如果我们有多个计算结果，可以搭配循环语句快速提取每一个计算结果的态密度数据。

仍然以尖晶石型锰酸锂为例，当前目录下有 9 个子目录，每个子目录下都是一个态密度计算的结果（如下）：

```txt
LixMn2O4_111-Vac0_i0_w1    LixMn2O4_111-Vac3_i00_w48  LixMn2O4_111-Vac6_i00_w16
LixMn2O4_111-Vac1_i0_w8    LixMn2O4_111-Vac4_i00_w24  LixMn2O4_111-Vac7_i0_w8
LixMn2O4_111-Vac2_i00_w16  LixMn2O4_111-Vac5_i00_w48  LixMn2O4_111-Vac8_i0_w1
```

可以通过下面的脚本提取每个计算结果中的态密度数据

```bash
#!/bin/bash

for dirname in *
do 
	cd $dirname
	(echo '111')| vaspkit  > vaspkit-extract-dos-dat_`date +%Y%m%d`.log
	(echo '115'; echo 'Mn'; echo 'd'; echo 'O'; echo 'p'; echo -e '\n') | vaspkit >> vaspkit-extract-dos-dat_`date +%Y%m%d`.log
	cd ../
done
```

通过这个脚本可以在每个子目录下生成 `TDOS.dat` 和 `PDOS_USER.dat` ，我们再将其复制到本地作图即可。

### 使用 `printf`  格式化输出将文件传输至本地

我习惯使用 `scp` 在服务器和本地传输文件，但每次都要 `进入传输的文件路径 > 复制文件地址 > 本地敲命令传输` 十分繁琐，因此我构建了一个自动在屏幕输出 `scp` 传输命令的句子，就可以很方便地实现文件传输。 

```bash
printf "%s %s%s%s %s%s%s%s%s\n" scp `hostname` : `readlink -f $dirname/PDOS_USER.dat` ./ `date +%Y%m%d` _ $dirname _PDOS_USER.dat
```

将这个句子放入上面的循环中，就可以实现在屏幕上输出传输文件要使用的 scp 命令，屏幕输出结果类似 `scp yourhostname:/your/file/path ./20220607_path_PDOS_USER.dat` 之后就可以将其复制到你本地的 SHELL 传输文件喽~

> 我这里通过 `hostname` 来指定服务器地址是提前在 `~/.ssh/config` 中配置过，具体可以参考[这篇文章](https://daemon369.github.io/ssh/2015/03/21/using-ssh-config-file)配置。

同理，我们也可以写出传输 `TDOS.dat` 文件的句子：

```bash
printf "%s %s%s%s %s%s%s%s%s\n" scp `hostname` : `readlink -f $dirname/TDOS.dat` ./ `date +%Y%m%d` _ $dirname _TDOS.dat
```

上面讲的是态密度数据的提取，对于很多任务都是通用的。要抓住不同任务的相同点，灵活运用循环语句来提高数据处理效率。

# 参考文献

[1] [Tutorials — VASPKIT 1.3 documentation](https://vaspkit.com/tutorials.html#run-vaspkit-in-batch-mode)

