---
title: 解决网页视频进度条禁止拖动
toc: true
date: 2022-06-17 18:09:49
tags: ["刷课"]
categories: ["Other"]
description: 刷课的时候很无聊，网页又限制视频不能拖动，下面是找到的一个解决方案。
---

> 本文转载自 [网页视频禁止拖动进度条](https://blog.csdn.net/weixin_46435234/article/details/114437203)

1. 按键盘`F12` 进入开发者模式

![进入开发者模式](https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/20220617182234.png)

2. 发现标志,html5播放器，属于原生支持最方便实现加速的

3. 在开发者模式中找到 `Console`  调式窗口，输入以下代码进行设置

![console 调试](https://flmore-github-io-1306099430.cos.ap-beijing.myqcloud.com/markdown-img/20220617182435.png)

**设置视频播放速度**

```javascript
/* play video twice as fast */
document.querySelector('video').defaultPlaybackRate = 1.0;//默认一倍速播放
document.querySelector('video').play();

/* now play three times as fast just for the heck of it */

document.querySelector('video').playbackRate = 10.0;  //修改此值设置当前的播放倍数

```

**直接跳过视频**

```javascript
function skip() {
    let video = document.getElementsByTagName('video')
    for (let i=0; i<video.length; i++) {
        video[i].currentTime = video[i].duration
    }
}
setInterval(skip,200)
```

