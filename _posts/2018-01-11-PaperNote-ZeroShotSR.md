---
title: PaperNote - “Zero-Shot” Super-Resolution using Deep Internal Learning
date: 2018-01-11 17:11:00
categories:
- PaperNote
tags:
- DeepLearning
- PaperNote
mathjax: true
---

<center>“Zero-Shot” Super-Resolution using Deep Internal Learning</center>



<!-- more -->



### 前言

目前大多数SR方法都需要大量对应的HR图和LR图作为训练集，其中LR图是使用特定的方式从HR图中获取的。但是真实的LR图千奇百怪，并没有特定的规律，这也就导致了这些方法在真实的LR图上效果欠佳。该文章就这个问题提出了一种非监督方法，无需是用大的LR-HR数据集进行预训练，而只在测试的时候训练网络。

### Internal Image Statistics

文章建立在这样一种思想之上：“自然图像有着非常强的内部数据重复性”。再同一张图像中，任何一个任意小的patch都可以在同一张图像中找到许多与之相似的大小不同的patches，而这些信息却几乎不存在于任何其他图像中。因此从图像本身提取出的信息，相比于从外部数据及提取信息，能更好的用于重建、恢复。

### 方法（Image-Specific CNN）

无需外部数据集作为预训练样本，给定测试图像$I$，下采样得到$I\downarrow{s}$（s是尺度系数）。$I$和$I\downarrow{s}$即构成了HR-LR训练样本，用此样本训练网络，即得到用于图像$I$的Image-Specific CNN。将$I$作为输入，则得到输出$I\uparrow{s}$，即SR图。其中CNN的全卷积网络，因此可以输入不同尺度的图像。具体过程如图所示：

<img src="https://github.com/mengyangniu/images/blob/master/zssr-Figure4.png?raw=true" style="zoom:60%"/>

但是这样的方法训练集中只有一张图像，因此做如下数据增广：

1. 对$I$下采样得到许多小图($I=I_0,I_1,…I_n$)，再对不同的小图分别做下采样，即得到更多的LR-HR样本组；
2. 对1中所获得的样本组，做随机旋转（ 0°、90°、180°、270°）和随机水平镜像、随机垂直镜像变换，这样就使样本数量增加到原来的8倍；
3. ​