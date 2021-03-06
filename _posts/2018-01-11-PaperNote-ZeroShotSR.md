---
title: PaperNote - “Zero-Shot” Super-Resolution using Deep Internal Learning
date: 2018-01-11 17:11:00
categories:
- PaperNote
tags:
- DeepLearning
- PaperNote
- SuperResolution
mathjax: true
---

<center>“Zero-Shot” Super-Resolution using Deep Internal Learning</center>

<br />

<center>这篇笔记非常水。</center>



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
3. 增加更多的尺度因子($s_1<s_2<…<s_m=s$)，对于$s_i$，增加生成的SR图$HR_i$以及其下采样、旋转之后的版本，对这些图进行尺度为$s_{i+1}$的下采样来得到新的LR-HR训练样本，重复直至$s_m$。（有点没看明白）

### 网络结构

有监督的CNN方法，有需要使用大数据集训练，需要学习LR图与HR图之间所有可能的联系，因此网络都非常深且复杂。相反的，单图像LR与HR之间的联系就简单得多，因此可以用轻量化的网络实现。

文章使用了8个隐藏层的全卷积网络，每一个卷积层均有64个channel，每一层后面都是用ReLU作为激活函数。为了减少计算量，只训练LR与HR之间的残差。

### 普适性

有监督的SR方法在下采样方式恒定的前提下能有非常好的效果，但是实际中这些都是变化的，不可知的。比如说，不同的下采样核、不同的噪声环境、不同的压缩方法等等。有监督学习的方法无法针对所有的情况进行训练。而Image-Specific CNN就能适应这样的问题，网络可以从用户接收参数：

1. 下采样核（默认是双三次差值）；
2. SR系数s；
3. 尺度因子的数目(m)，比较均衡的值是6；
4. 是否强制BP？
5. 是否增加噪声

### 总结

文章写得有点模糊，很多细节都模棱两可。但是文章的思想还是很值得借鉴的，使用Internal Image Statistics来重建图像，我持中立态度。文章所提出的一些监督学习存在的问题，在大多数刷PSNR的方法中也确实是存在的。文章所提出的方法对于一些损坏较严重的图像来说比较实用，这是很多TSoA方法所不具备的。
