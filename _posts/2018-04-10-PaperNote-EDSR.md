---
title: PaperNote - EDSR
date: 2018-04-10 15:36:00
categories:
- PaperNote
tags:
- DeepLearning
- PaperNote
- SuperResolution
mathjax: true
---

<center>State of Art的SR算法</center>

<!-- more -->

这篇文章提出的模型主要是提高了图像超分辨的效果，并赢得了NTIRE2017 Super-Resolution Challenge。文章做出的修改主要是在残差网络上。残差结构的提出是为了解决high-level问题，而不能直接套用到超分辨这种low-level视觉问题上。因此作者移除了残差结构中一些不必要的模块，结果证明这样确实有效果。另外，作者还设置了一种多尺度模型，不同的尺度下有绝大部分参数都是共用的。这样的模型在处理每一个单尺度超分辨下都能有很好的效果，称为MDSR。

在知乎上看到，<a href="https://www.zhihu.com/question/62599196/answer/200311621">“BN对超分辨率CNN有Impact在《Single Image Super-resolution with a Parameter Economic Residual-like Convolutional Neural Network》的实验中同样有相同的结论”</a>。在综述中看到过这篇文章，但没怎么注意，有时间可以看一下。

### EDSR

下图是作者提出的Residual Blocks和SRResNet(即SRGAN)采用的Residual Blocks和原始的Residual Blocks之间的对比：

![EDSR-Figure2](https://raw.githubusercontent.com/mengyangniu/images/master/EDSR-Figure2.png)

可以看到文章所提出的Residual Blocks只是去除了Batch Normalization，并没有什么特别的改动。

下图是EDSR的网络结构：

![EDSR-Figure3](https://raw.githubusercontent.com/mengyangniu/images/master/EDSR-Figure3.png)

作者认为Batch Normalization Layers只是通过归一化特征输出来去除模型的灵活度，因此去除比较好，还能减少GPU显存占用约40\%。太多的残差块会导致训练不稳定，因此作者采取了residual scaling的方法，即残差块在相加前，经过卷积处理的一路乘以一个小数（比如作者用了0.1）。这样可以保证训练更加稳定。

但是我觉得作者去除BN的理由都没有什么说服力，上网查阅，看到贾扬清这样评价：

> BN在classification当中的效果较好，主要原因是因为classification对于scale不敏感。但是superresolution这样图片到图片的变换，per image的scale还是一个有效信息。
>
> \textbf{类似的一个情形是style transfer，同样也是图片到图片的变换，Ulyanov et al.提出用instance normalization的方法，IN可以理解成为每个图片自己做BN，这样比BN能保留更多scale信息。更新的研究表明如果训练收敛不是问题的话，进一步去掉IN的效果也会更好。
>
> 总的来说BN对于简化调参有好处（更容易收敛），但是并不一定真的适合所有应用。[LINK](https://www.zhihu.com/question/62599196)

还看到有人说：

> 超分辨率CNN输入和输出可以认为有着很相似的空间分布，而BN白化中间的特征的方式完全破坏了原始空间的表征，因此在重建需要模型中部分层或者参数来恢复这种表征，所以同样多的参数，有BN的还要拿出一部分参数做这部恢复，效果自然就稍差了点。

### MDSR

将EDSR拓展成多上升尺度方法，先略过。