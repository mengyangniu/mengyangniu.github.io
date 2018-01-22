---
title: Linux死机问题解决方案 
date: 2018-01-22 09:41:00
categories:
- Linux
tags:
- Script
mathjax: false
---

<center>转载自<a href="https://izhaolei.github.io/bug%E9%9B%86%E5%90%88/2018/01/21/linux-%E9%97%AE%E9%A2%98%E9%9B%86%E5%90%88/#">iZhaolei的博客</a></center>

<!-- more -->

### 问题

使用Ubuntu跑深度学习时，经常遇到这样一个问题：跑完程序电脑闲置一段时间后，使用htop可以看到CPU的一个线程持续100%被占用，此时使用gpu的深度学习程序也无法再次运行，nvidia-smi命令同样无法启动。并且一定概率无法使用shutdown、reboot、init等命令关机或重启。
