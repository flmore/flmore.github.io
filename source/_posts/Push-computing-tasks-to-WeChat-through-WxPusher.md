---
title: 通过 WxPusher 推送计算任务
toc: true
date: 2022-06-27 09:42:10
tags: ["WxPusher", "python script"]
categories: ["ComputMat"]
description: 之前使用发邮件的方式来推送计算任务。但这种方式配置比较繁琐，接收端还需要打开邮件查看的时间成本。最近发现了 WxPusher 这个信息推送工具，配置简单，可以将信息直接发送到微信上，这篇文章记录了配置过程。
---

# 前言

当服务器计算任务完成之后，为了让我们能够及时收到任务信息，可以使用 [WxPusher](https://wxpusher.zjiecode.com/docs/#/) 等工具来推送消息。

# WxPusher 管理后台配置

我们首先需要在 `WxPusher` 的管理后台进行配置，主要步骤如下：

- 在[后台管理](https://wxpusher.zjiecode.com/admin/main)中注册应用，获得 `appToken`
- 新建主题，需要接收推送消息的人扫码订阅，获取接收者 `UID`
- 使用 api 接入，发送 `get` 或者 `post` 消息即可

详细过程见[官方文档](https://wxpusher.zjiecode.com/docs/#/?id=快速接入) ，在此不再赘述。

配置完成后，我们需要获取的必要信息为：**appToken**、**UID**。

# ~~Python 脚本调用 WxPusher~~

**实际使用过程中该脚本还存在问题，对于某些任务完成后并不会推送，有待进一步修正**

python 脚本如下，将以下内容写入 `wxpusher_notification.py` ，在对应位置替换你自己的 `appToken` 和 `UID`

```python
#! /usr/bin/env python
# coding utf-8

import requests
import json
import sys
import os
import re


def get_current_jobid():
    id_content = os.popen("qstat").read()
    pattern = re.compile(r'^\d{1,}.hpc', re.MULTILINE) # hpc is the server's hostname
    result = pattern.findall(id_content)
    pending_jobid = [ re.findall("^\d+",i)[0] for i in result ]
    for jobid in pending_jobid:
        current_dir = os.getcwd()
        jobdetail_info = os.popen("qstat -f " + str(jobid)).read()
        pattern = re.compile(current_dir)
        if re.search(pattern,str(jobdetail_info)):
            return jobid

def get_job_detail_info(jobid, jobinfodic):
    query_lst = ["Job_Name", "job_state", "total_runtime"]
    os.system("qstat -f " + str(jobid) + " > job_info.log")

    for line in open('job_info.log'):
        if query_lst[0] in line:
            jobname = line.split("=")[1].strip()
            jobinfodic["jobname"] = jobname
        elif query_lst[1] in line:
            jobstate = line.split("=")[1].strip()
            jobinfodic["jobstate"] = jobstate
#         elif query_lst[2] in line:
#             jobtime = line.split("=")[1].strip()
#             jobinfodic["jobtime"] = jobtime
    os.system("rm job_info.log")
    return jobinfodic


def send_mesg(jobinfo_dic, url, app_token, uids):

    jobinfo_tup = (jobinfo_dic["jobid"], jobinfo_dic["jobname"], jobinfo_dic["jobstate"], jobinfo_dic["jobdir"])
    main_body = "# job has finished on hpc \n **job id** \t %s \n **job name** \t %s \n **job states** \t %s \n **job workdir** \t %s" %jobinfo_tup

    content = {"appToken": app_token,
               "content": main_body,
               "summary":"job " + str(jobinfo_dic["jobid"]) + " has finished on hpc",
               "contentType":3,
               "uids": [uids],
               "url":"http://wxpusher.zjiecode.com"
              }

    content = json.dumps(content)
    r = requests.post(url, data=content, headers={'Content-Type': 'application/json'})
    return


def main():
    
    # input required parms
    url = "http://wxpusher.zjiecode.com/api/send/message"
    app_token = "your app_token" # your app_token
    uids = "UID_xxxxxxxxxxxxxxxxxxx" # UID of the sending target

    # get job detail info
    jobinfo_dic = {}
    jobinfo_dic["jobid"] = get_current_jobid()
    jobinfo_dic["jobdir"] = os.getcwd()
    get_job_detail_info(jobinfo_dic["jobid"], jobinfo_dic)
    
    # send message with wspusher
    send_mesg(jobinfo_dic, url, app_token, uids)

if __name__ == '__main__':
    main()
```

给 `wxpusher_notification.py` 添加写权限

```bash
chmod +x wxpusher_notification.py
```

将 `wxpusher_notification.py` 移动至环境目录下

```bash
mv wxpusher_notification.py ~/.loca/bin
```

# PBS 提交脚本配置

pbs 提交脚本的模板如下

```bash
#!/bin/bash
#PBS -N your-job-name
#PBS -l nodes=1:ppn=24
#PBS -q batch
#PBS -l walltime=100:00:00
#PBS -m e
#PBS -M email-address

MPIRUN_COMMAND  # your command to run software

wxpusher_notification.py
```

即在原有的 pbs 提交脚本最后添加 `wxpusher_notification.py` 

# 任务推送测试

通过上面的步骤配置好后，任务完成后，在微信端就可以接收到消息了。

![wxpusher 接收消息测试](https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/wxpusher-send-msg-sample.webp)

# 参考文献

[1] [计算任务的推送 (cheng-group.net)](https://wiki.cheng-group.net/wiki/集群使用/notification_for_hpc)



