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

文章建立在这样一种思想之上：“自然图像有着非常强的内部数据重复性”。即同一张自然图像中，有许多同尺度或者不同尺度的patch有着很高的相似度，这样的规律在几乎任何一张自然图像中，对于任意一个小patch都适用。而且文章认为，有些图像中的待恢复的细节，不存在于任何其他图像中；有研究证明，单一图像的patches的内部熵远小于图像集合的patches之间的外部熵；这些都证明图像内部统计信息能够提供比大规模的外部数据集更强大的恢复能力。

### 方法（Image-Specific CNN）

无需外部数据集作为预训练样本，给定测试图像$I$，构建为此图像量身定做的超分辨率网络Image-Specific CNN，对$I$下采样获得更低分辨率的$I\downarrow{s}$（s是SR系数）作为训练集。再将$I$输入训练好的CNN，即获得SR输出$I\uparrow{s}$。其中CNN的全卷积网络，因此可以输入不同尺度的图像。具体过程如图所示：

<img src="https://github.com/mengyangniu/images/blob/master/zssr-Figure4.png?raw=true" style="zoom:60%"/>

对$I$做图像增广以获得更多的LR-HR训练样本。办法就是将$I$以不同的下采样方式（文章没细说，我自己的理解），相同的下采样率$s$，得到不同的$I\downarrows$，也就得到了许多LR-HR样本组。进一步，将这些样本组进行0度、90度、180度、270度的随机旋转以及随机的水平和垂直镜像，可将训练集增大8倍。

为了获得更好的鲁棒性，SR是逐渐进行的。算法应用于一些中间的尺度因子$(s_1,s_2,…s_m=s)$，将其生成的SR图$HR_i$以及其对应经过下采样并旋转的图添加到训练集中