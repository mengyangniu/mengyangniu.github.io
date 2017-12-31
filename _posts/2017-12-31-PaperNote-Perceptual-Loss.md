---
title: PaperNote - Perceptual Loss
data: 2017-12-31 23:42:00
categories:
- PaperNote
tags:
- DeepLearning
- PaperNote
mathjax: true
---

<center>Perceptual Losses for Real-Time Style Transfer and Super-Resolution</center>

文章主要考虑的是图像转换问题，如超分辨率和图像风格转换。大部分现有的方法都是计算输入与输出之间的逐像素误差来作为损失函数训练前馈神经网络；还有一些文章表明，通过从预训练的网络中提取高级特征来定义和优化损失，可以生成高质量的图像（如[Image Style Transfer](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf)，固定预训练网络参数不变，通过优化随机输入，来得到一张具有图片A的内容和图片B的风格的结果图）。文章结合两者，提出利用感知损失功能来训练前馈网络进行图像转换。

文章以图像风格转换问题为例，相对于基于优化的方法（即Gatys的[Image Style Transfer](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf)），有相似的结果，却快了3个数量级；在单图像超分辨率问题上也能有很好的效果。

### 方法

![](https://raw.githubusercontent.com/mengyangniu/images/master/Perceptual-Loss-Figure2.png)

如图所示，该方法由两个部分组成：图像转换网络$f_W$和损失网络$\phi$，其中$\phi$定义一系列损失函数$\mathcal{l}_1,\mathcal{l}_2,…\mathcal{l}_k$。图像转换网络$f_W$是一个深度残差卷积神经网络，参数为$W$，通过映射$\hat{y}=f_W(x)$将输入$x$转化为输出$\hat{y}$；损失函数$\mathcal{l}(\hat{y},y_i)$计算输出图像$y$与目标图像$\hat{y}$之间的差别。使用SGD来最小化各损失函数加权后的总损失$W^*$来训练网络：

$$
W^*=arg\min_{W}E_{x,\{y_i\}}[\sum_{i=1}\lambda_i\mathcal{l}_i(f_W(x),y_i))]
$$

> <i>($\lambda_i$是什么？)</i>

作者认为基于优化的图像生成方法[[1]](https://arxiv.org/abs/1412.0035)[[2]](https://arxiv.org/abs/1312.6034)[[3]](https://arxiv.org/abs/1506.06579)[[4]](https://arxiv.org/abs/1505.07376)[[5]](https://arxiv.org/abs/1508.06576)的关键在于它们所在用的预训练模型已经学会了编码感知信息和语义信息的能力，而这正是文章所提出的Perceptual Loss所要度量的。因此用图像分类预训练网络$\phi$作为固定的损失网络，用于定义损失函数。*然后再用这个损失函数来训练图像转换网络，而这个损失函数本身也是一个深度卷积网络。*