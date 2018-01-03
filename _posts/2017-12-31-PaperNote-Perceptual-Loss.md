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

<!-- more -->

文章主要考虑的是图像转换问题，如超分辨率和图像风格转换。大部分现有的方法都是计算输入与输出之间的逐像素误差来作为损失函数训练前馈神经网络；还有一些文章表明，通过从预训练的网络中提取高级特征来定义和优化损失，可以生成高质量的图像（如[Image Style Transfer](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf)，固定预训练网络参数不变，通过优化随机输入，来得到一张具有图片A的内容和图片B的风格的结果图）。文章结合两者，提出利用感知损失功能来训练前馈网络进行图像转换。

文章以图像风格转换问题为例，相对于基于优化的方法（即Gatys的[Image Style Transfer](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf)），有相似的结果，却快了3个数量级；在单图像超分辨率问题上也能有很好的效果。

### 结构

![](https://raw.githubusercontent.com/mengyangniu/images/master/Perceptual-Loss-Figure2.png)

如图所示，该方法由两个部分组成：图像转换网络$f_W$和损失网络$\phi$，其中$\phi$定义一系列损失函数$\mathcal{l}_1,\mathcal{l}_2,…\mathcal{l}_k$。图像转换网络$f_W$是一个深度残差卷积神经网络，参数为$W$，通过映射$\hat{y}=f_W(x)$将输入$x$转化为输出$\hat{y}$；损失函数$\mathcal{l}(\hat{y},y_i)$计算输出图像$y$与目标图像$\hat{y}$之间的差别。使用SGD来最小化各损失函数加权后的总损失$W^*$来训练网络：

$$
W^*=arg\min_{W}E_{x,\{y_i\}}[\sum_{i=1}\lambda_i\mathcal{l}_i(f_W(x),y_i))]
$$

> <i>($\lambda_i$是什么？)</i>

作者认为基于优化的图像生成方法[[1]](https://arxiv.org/abs/1412.0035)[[2]](https://arxiv.org/abs/1312.6034)[[3]](https://arxiv.org/abs/1506.06579)[[4]](https://arxiv.org/abs/1505.07376)[[5]](https://arxiv.org/abs/1508.06576)的关键在于它们所在用的预训练模型已经学会了编码感知信息和语义信息的能力，而这正是文章所提出的Perceptual Loss所要度量的。因此用图像分类预训练网络$\phi$作为固定的损失网络，用于定义损失函数。*然后再用这个损失函数来训练图像转换网络，而这个损失函数本身也是一个深度卷积网络。*

损失网络$\phi$用来定义特征重建损失$\mathcal{l}_{feat}^{\phi}$和风格重建损失$\mathcal{l}_{style}^{\phi}$。

对于风格转换问题，有内容目标$y_c$和风格目标$y_s$，需要对每一种风格训练一个网络。对于但图像超分辨率问题，只有内容目标$y_c$，无风格重建损失，对于每一级超分辨率尺度训练一个网络。

### 图像转换网络

文章使用的图像转换网络框架参考[https://arxiv.org/abs/1511.06434](https://arxiv.org/abs/1511.06434)，不使用Pooling层，而通过stride来完成网络中的上采样和下采样。网络包含5个残差块，结构遵照[http://torch.ch/blog/2016/02/04/resnets.html](http://torch.ch/blog/2016/02/04/resnets.html)，所有的非残差卷积层后都有spatial Batch Normalization和ReLu，而输出层则使用tanh并将其输出规范化到[0,255]。出了第一层和最后一层使用9\*9的卷积核，其它层使用3\*3的卷积核。[[Supplementary Material](https://pdfs.semanticscholar.org/9fa3/720371e78d04973ce9752781bc337480b68f.pdf)]

![style transfer networks](https://github.com/mengyangniu/images/blob/master/Perceptual-Losses-Supplementary-Material-Table1.png?raw=true)

![Super Resolution networks](https://github.com/mengyangniu/images/blob/master/Perceptual-Losses-Supplementary-Material-Table2.png?raw=true)

![Residual block](https://github.com/mengyangniu/images/blob/master/Perceptual-Losses-Supplementary-Material-Figure1.png?raw=true)

### Perceptual Loss Function

文章使用ImageNet预训练的VGG16来计算Perceptual Loss Function，分为特征重建损失（Feature Reconstruction Loss）和风格重建损失（Style Reconstruction Loss）。

##### 特征重建损失

（我个人理解特征损失就是内容损失。）

$\phi_j(x)$表示损失网络$\phi$在输入为$x$时第$j$层的激活值，若第$j$层是一个卷积层，则$\phi_j(x)$是维度为${C_j}\times{H_j}\times{W_j}$的特征图。特征重建损失表示为特征图之间的欧氏距离：

选用不同卷积层的特征图计算特征损失，用以重建图像。可以由下图看到，选用的层越深，重建出来的图像就越是倾向于只保留内容、结构信息，丢失颜色、质感等细节信息：
$$
\mathcal{l}^{\phi,j}_{feat}(\hat{y},y)=\frac{1}{C_jH_jW_j}{||\phi_j(\hat{y})-\phi_j(y)||}^2_2
$$

![](https://github.com/mengyangniu/images/blob/master/Perceptual-Loss-Figure3.png?raw=true)

##### 风格重建损失

使用风格损失表征颜色、质地、样式等细节信息的偏差。和上述相同，$\phi_j(x)$表示损失网络$\phi$在输入为$x$时第$j$层的激活值，若第$j$层是一个卷积层，则$\phi_j(x)$是维度为${C_j}\times{H_j}\times{W_j}$的特征图。定义${C_j}\times{C_j}$的格拉姆矩阵（Gram Matrix）$G_j^{\phi}(x)$，其元素：

$$
G_j^{\phi}(x)_{c,c^{'}}=\frac{1}{C_jH_jW_j}\sum_{h=1}^{H_j}\sum_{w=1}^{W_j}\phi_j(x)_{h,w,c}\phi_j(x)_{h,w,c^{'}}
$$

如果认为$\phi_j(x)$表示在$H_j\times{W_j}$网格中的每一点都有一个$C_j$维特征，则$G_j^{\phi}(x)$可以认为是，在网格中点与点相互独立的前提下，这些特征之间的协方差矩阵。也有更加简便的计算方式：将${C_j}\times{H_j}\times{W_j}$的$\phi_j(x)$展成${C_j}\times{H_jW_j}$的$\psi$，则：

$$
G_j^{\phi}(x)=\frac{\psi\psi^T}{C_jH_jW_j}
$$

则风格损失可以表示为：

$$
\mathcal{l}_{style}^{\phi,j}(\hat{y},y)=||G_j^{\phi}(\hat{y}-G_j^{\phi}(y)||_F^2
$$

（即使$\hat{y}$与$y$尺寸不同，他们的Gram Matrix也有相同的尺寸，因此风格损失函数可以计算不同尺寸的图像之间的损失）

文章考虑到不同层所表征的结构特征，最终使用不同层的风格损失$l_{style}^{\phi,j}(\hat{y},y)$相加得到总的风格损失$l_{style}^{\phi,J}(\hat{y},y)$。

> 该论文的风格损失和[图像风格转换](https://pdfs.semanticscholar.org/7568/d13a82f7afa4be79f09c295940e48ec6db89.pdf)中的风格损失并没有什么不同，关于同一层不同通道特征图之间的内积组成的Gram矩阵为何能够表征风格，并没有给出回答。

#### 总结

Perceptual Loss主要就是加快了图像风格转换的速度，将基于优化的方法变成了训练前馈神经网络。

但是关于风格表征还只是沿用了之前的工作，并没有做出更多的解释。