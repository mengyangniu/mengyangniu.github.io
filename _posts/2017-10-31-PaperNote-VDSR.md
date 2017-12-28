---
title:  PaperNote - VDSR
date: 2017-10-31 09:18:00
categories:
- PaperNote
tags:
- PaperNote
- SuperResolution
- DeepLearning
---

<center>VDSR(Accurate Image Super-Resolution Using Very Deep Convolutional Networks), CVPR2016</center>

<!-- more --> 

### Abstract

作者发现增加网络深度能够显著提升SR性能，收到VGG的启发，最终使用了包含20层weight layers的模型。作者说“By cascading small ﬁlters many times in a deep network structure, contextual information over large image regions is exploited in an efﬁcient way.” 虽然如此，但是模型也就变得非常难拟合，所以作者又引入了残差，并用极大的学习率只训练残差。

### Proposed Method

![VDSR](https://github.com/mengyangniu/images/blob/master/VDSR-Figure2.PNG?raw=true)

首先将LR图经过差值处理成一张模糊的高分辨率图，将其送入20层的CNN，输出残差，再将其与原图相加输出最终的HR图。20层的CNN除去首层和尾层，全部进行相同的卷机操作，有着相同的3\*3\*64卷积核和stride。

卷积使得可以根据N\*N个像素去推断目标像素值，图像处理中大家普遍认为像素之间是有相关性的，所以 根据更多的像素数据去推断目标像素，也是认为是一个提高效果的操作。但是图片通过逐步卷积，图像大小也越来越小，一个很小的图，首先不适合一个很深的网络（这会要求输入图片的size很大），其次，如果采取传统的卷积操作，边界像素会因考虑的区域少，感受野小而导致结果较差，于是传统的做法是对卷积结果做一个裁剪，剪去边界区域，随之产生一个新的问题：裁剪后的图像变的更小。在图像分类中，这个问题并不算严重，但是在SR问题中，这会导致输入与输出的不一致。为避免这个问题，作者对每一个卷积层都做了zero-padding处理，保证了卷积前后的一致性。

此网络还有一个创新点：将residual的思想引入SR，减轻了网络的“负担”，又加速了学习速率。虽然是端到端的CNN，但是训练残差和直接训练完全不一样。引入残差思想后，大多特征经过ReLu都输出为0，即无需反向传播，大大简化了训练难度，如图所示。

![residual vs non-residual](https://github.com/mengyangniu/images/blob/master/VDSR-Figure4.PNG?raw=true)

在这篇文章中，作者提到训练一个16层的网络用于SR只需要4个小时，这无疑极大地提升了训练速度。

残差思想的引入，使得深度网络能够应用于SR领域，而随着深度的加深，网络的perceptive field变得越来越宽，使得网络不仅具备局部特征，也能捕捉到更加全局的特征，全局特征的加入更有利于SR纹理细节的恢复。

一般的网络结构只能做特定尺度的upscale，如一个网络结构确定为放大尺度为2的网络，那么该网络训练后也就只能用于尺度为2的upscale，而随着网络结构的加深，训练成本相应增加，若网络只能做单一尺度的放大，那么网络的可重用性比较差。在这篇文章中，作者还引入了另一种令人惊讶的网络结构，该网络结构对外暴露多一个尺度参数，该尺度参数可控制图像放大的尺度。这种设计方案极大地提升了网络的可重用性。作者还在这篇文章中对比了单一尺度网络的效果和多尺度网络的效果，如表(10)。可以发现，多尺度网络的效果不仅不降低，反而提升了PSNR值。

![Scale Factor Experiment](https://github.com/mengyangniu/images/blob/master/VDSR-Table2.PNG?raw=true)

![compare](https://github.com/mengyangniu/images/blob/master/VDSR-Figure5.PNG?raw=true)

### Summary

一个很深的网络对图像的超分辨率是很必要的，很深的网络使得能够根据更多的像素即更大的区域来预测目标像素信息，其次，对于网络层次加深带来的梯度消失问题，可以通过残差学习和提高学习率结果，此外，残差学习加快了收敛速度。多分辨率训练能够提升网络性能。

从结果上看，还是逃不出一句话：网络层次越深，结果越好。

但是更深的结构为收敛和梯度传输带来困难，如何较好的解决此问题才是关键。
