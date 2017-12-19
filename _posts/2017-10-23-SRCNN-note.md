---
title:  PaperNote-SRCNN
subtitle: SRCNN - Image Super-Resolution Using Deep Convolutional Networks
date: 2017-11-23 19:49:00
categories:
- PaperNote
tags:
- PaperNote
---

PaperNote - SRCNN - Image Super-Resolution Using Deep Convolutional Networks



在课题的选择上，我的初步打算是图像超分辨率（Super Resolution, SR）或者图像分割。但是由于图像分割Mask-RCNN一直未开源，GitHub上只有tensorflow复现且问题颇多，大家的pytorch复现也都遇到了问题；而超分辨率NITRE2017冠军EDSR开源了他们的代码，于是打算先把图像超分辨率领域的一些重点论文拿来看一看。

在图像超分辨率领域，在深度学习之前就已经有大量优秀的研究工作，例如Sparse Coding，暂且不表。而使用深度学习解决超分辨率问题的开山之作——SRCNN[[1]](https://arxiv.org/abs/1501.00092)，第一次将CNN引入了SR领域。

SRCNN的思路非常清晰，算法也很简单，开山之作大抵如此。

### Abstract

文章认为LR之所以能通过一定的算法升级为HR，主要是他们两者之间存在着一些“共通的特征”。作者采用的方法是在度分辨率图（LR）和高分辨率图（HR）之间直接学习一个end2end的映射，这个映射的表现形式为CNN。文章还将展示传统的Sparse Coding方法在本质上也可以看作是一个深度神经网络，区别在于传统方法将各个模块分开处理，而SRCNN统一优化所有参数。

### Method

#### 构思

*虽然SRCNN最终表现为一个CNN，但是文章并不是简单粗暴的说我们用的是CNN，而是在构思上下了不少功夫。*

*文章是在从另一个角度阐述一个简单的CNN是如何应用于SR问题的，非常值得学习。*

文章首先将bicubic interpolation当成预处理，低分辨率图进行双三次插值resize成高分辨率图，文中仍然称这个插值后的图为low resolution，作为后续输入。随后分为3步走，分别是

1. LR 特征提取（Patch extraction and representation），这个阶段主要是对LR进行特征提取，并将其特征表征为一些feature maps；
2. 特征的非线性映射（Non-linear mapping），这个阶段主要是将第一阶段提取的特征映射至HR所需的feature maps；
3. HR重建（Reconstruction），这个阶段是将第二阶段映射后的特征恢复为HR图像。

##### Patch extraction and representation

图像修复的流行策略是密集地从图像中提取patch并用预训练好的PCA、DCT或者Haar基来表示。这就如同是用一系列滤波器对图像卷积操作，而每个滤波器就如同一个基。因此第一层表现出来就是一个conv层+relu层，输出的结果是filter个LR的feature map。

##### Non-linear mapping

将上一个阶段提取出的LR的n1维Feature map映射至HR的n2维Feature map。即将LR的Feature map中的n1维vector映射至HR的Feature map中的n2维vector，有点类似于fc。但出于“filter”的思想，文章还是用1\*1的卷积代替fc，即形成了一个核为1*1的卷积层。

也可以用更大的卷积核、更多的卷积层来加强模型的非线性性。

##### Reconstruction

传统方法将HR的patch取均值来得到最后的HR图片。均值可以表示为在一组特征图上用训练好的filter做卷积。因此第三步也可以视作是一个卷积层。

虽然这三步操作都有不同的目的，但是都指向了相同的操作——卷积。将其合在一起就形成了CNN。

![SRCNN](https://raw.githubusercontent.com/mengyangniu/Images_for_MarkDown/master/SRCNN-Figure2.png)



#### 与基于Sparse Coding的方法的联系

对稀疏不了解，文章大意是sparse-codingbased SR方法可以被视为一种采用了不同的非线性映射的卷积神经网络。

> 对于sparse coding问题，n2=n1，因为它希望用自身的高阶特征来重新组合实现自编码。
>
> 而此处，n2<n1，文章给出了一组经验参数，f1 = 9，f3 = 5, n1 = 64, and n2 = 32。
>
> (1) 证明性能与网络的大小正相关，即采用更大的n1，n2有正向的结果。复杂度与图像尺寸线性相关。
>
> (2) 3层的性能对于重建问题已经足够，虽然是与稀疏编码统一的框架，但是由于是同时优化low-resolution dictionary 产生，non-linear mapping，以及reconstruction，所以性能更加优异。

### 训练

SRCNN直接计算原图与SR大图之间所有像素点的MSE作为损失函数，具体不表。

三个卷积层使用的卷积核的大小分为为9x9, 1x1和5x5，前两个的输出特征个数分别为64和32. 该文章分别用Timofte数据集（包含91幅图像）和ImageNet大数据集进行训练。

#### 颜色通道上的实验

SRCNN在不同颜色通道上分别进行SR，实验显示可以得到更好的效果。

### SRCNN的遗留问题

> (1) loss函数采用重建图像与原始图像的最小均方误差MSE，这在后面基本被perceptual loss全面替代。
>
> (2) bicubic上采样方法，后面也被学习的upscale卷积替代。 
>
> (3) 可以引入更深的网络。
>
> (4) residential有待引入。
>
> (5) 构造一个适用于不同放大尺度的网络。
>
> (6) 感受野太小。

### 后续阅读

VDSR[[2]](https://arxiv.org/abs/1511.04587)/DRCN[[3]](https://www.baidu.com/link?url=xXHiX7pMXhMwmokIUkPzZp28PoyJ_KfjQuc3eW5s-y_RrhkiAvwYFe-v0oKthyu5&wd=&eqid=88c352e0000224d40000000359ed9831)/ESPCN/VESPCN/**SRGAN**/EDSR



### 参考文献

[1] Dong C, Chen C L, He K, et al. Image Super-Resolution Using Deep Convolutional Networks[J]. IEEE Transactions on Pattern Analysis & Machine Intelligence, 2016, 38(2):295.

[2] Kim J, Lee J K, Lee K M. Accurate Image Super-Resolution Using Very Deep Convolutional Networks[J]. 2015:1646-1654.

[3] Kim J, Lee J K, Lee K M. Deeply-Recursive Convolutional Network for Image Super-Resolution[C]// Computer Vision and Pattern Recognition. IEEE, 2016:1637-1645.

[4] https://zhuanlan.zhihu.com/p/25532538?utm_source=tuicool&utm_medium=referral

