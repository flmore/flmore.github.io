---
title: 课题组计算服务器任务提交脚本
toc: true
date: 2022-10-14 18:51:49
tags: ["vasp", "pbs"]
categories: ["实验室"]
description: 本文记录了课题组服务器集群的任务提交脚本，方便大家取用。
---

# VASP提交脚本

在任务目录新建提交脚本

```bash
vim job.pbs
```

粘贴以下内容

```bash
#!/bin/bash
#========================================#
# Job submission script for VASP
# Created by flmore on October 14, 2022
#========================================#
# 1. PBS job control

#PBS -S /bin/bash
#PBS -N vasp
#PBS -l nodes=1:ppn=24
#PBS -q batch
#PBS -l walltime=30:00:00
#PBS -m e
#PBS -M your-email@xxx.com # A reminder message will be sent to this mailbox when the task is over.

# 2. load intel oneapi environment variable
source /opt/intel/oneapi/setvars.sh intel64 > /dev/null
export I_MPI_OFFLOAD_DEVICES=0
export I_MPI_DEBUG=5
export I_MPI_FABRICS=shm:ofi

# 3. run program
NP=`cat $PBS_NODEFILE | wc -l`
cd $PBS_O_WORKDIR

VASP=/opt/vasp/vasp.5.4.4/bin/vasp_std_intel-oneapi_gcc-11.2.0-avx2 # set vasp program path
#VASP=/opt/vasp/vasp.5.4.4/bin/vasp_std 
#VASP=/opt/vasp/vasp.5.4.4/bin/vasp_gam
#VASP=/opt/vasp/vasp.5.4.4/bin/vasp_ncl

mpirun -np $NP $VASP &> out.log

```

# GAUSSIAN提交脚本

在任务目录新建提交脚本

```bash
vim job.pbs
```

粘贴以下内容

```bash
#!/bin/bash
#PBS -N Gaussian
#PBS -o job.log
#PBS -e err.log
#PBS -l nodes=1:ppn=24
#PBS -l walltime=30:00:00
#PBS -l mem=96000mb
#PBS -q batch
#PBS -j oe

# this script is used to submit gaussian tasks
# created time: 2021.11.24
# edit by flmore
# ref url: http://bbs.keinsci.com/thread-22162-1-1.html

# run on AMD CPU
export PGI_FASTMATH_CPU=sandybridge

cd $PBS_O_WORKDIR

# run g16 program
for inf in *.gjf
do
    echo $inf
    g16 $inf
    wait
    formchk ${inf%.gjf}.chk
done &> out.log

# Delete Temporary Files to Reduce Hard Disk Usage
rm -rf $HOME/.g16/scratch/*
```

