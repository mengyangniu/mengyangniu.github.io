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

### 2018.01.24 更新

以下方法不能保证成功，强行重启还是会有重启失败的可能性。就我自己而言，截止昨日，已经有两次卡死后重启失败了。

猜测问题在于使用`watch nvidia-smi`命令导致cpu无法处理对显卡状态的频繁访问，导致内核进程卡住且无法释放。

目前尝试关闭`watch nvidia-smi`，若要查看显卡状态，手动执行`nvidia-smi`。不频繁访问`nvidia-smi`后，系统没有再卡死过，可行性有待进一步验证。

### 问题

使用 Ubuntu 跑深度学习时，经常遇到这样一个问题：跑完程序电脑闲置一段时间后，使用 htop 可以看到 CPU 的一个线程持续100%被占用，此时使用 GPU 的深度学习程序也无法再次运行，nvidia-smi 命令同样无法启动。并且一定概率无法使用 shutdown、reboot、init 等命令关机或重启。

使用 top 可以看到，一个名为 irq/${nnn}-nvidia 的进程占用 CPU 100%，且无法被 kill。主要是在跑调用 gpu 程序时，使用了 watch [-n 1] nvidia-smi 导致。

### 解决方案

sysrq组合键是内建于Linux内核的调试工具。只要内核没有完全锁住，不管内核在做什么事情，使用这些组合键都可以搜集包括系统内存使用、CPU任务处理、进程运行状态等系统运行信息。使用Alt + SysRq + [R-E-I-S-U-B] 就可以强行重启，但是sysrq组合键只能在连接着计算机的键盘上使用，无法远程控制重启。

而通过/proc/sysrq-trigger可以远程控制。

使用方式：

```bash
echo m > /proc/sysrq-trigger #导出内存分配信息

echo t > /proc/sysrq-trigger #导出当前任务状态信息

echo p > /proc/sysrq-trigger #导出当前CPU寄存器和标志位信息

echo c > /proc/sysrq-trigger #产生空指针panic事件，人为导致系统崩溃

echo s > /proc/sysrq-trigger #即时同步所有挂载的文件系统

echo u > /proc/sysrq-trigger #即时重新挂载所有的文件系统为只读

echo w > /proc/sysrq-trigger #转储处于uninterruptable阻塞状态的任务

echo b > /proc/sysrq-trigger #立即重启计算机

echo o > /proc/sysrq-trigger #立即关机计算机
```

在这个问题中，执行s、u、b即可。
