---
title: PaperNote - Image Style Transfer
data: 2017-12-26 09:38:00
categories:
- PaperNote
tags:
- DeepLearning
- PaperNote
mathjax: true
---

<center><i>"Image Style Transfer Using Convolutional Neural Networks"</i></center>

<!-- more -->

文章使用目标检测预训练网络VGG19中的16个卷积层和5个pooling层作为特征空间，文章对每一张图片不同位置的卷积滤波器的平均激活值都做归一化，由于只包含线性操作，没有使用BN或者pooling，因此归一化并不会改变VGG的输出 。同时网络不使用全连接层，并用Average Pooling代替Max Pooling（只是实验中发现这样能够提供更好的视觉感受）。

### 内容表达

CNN的每一层都是一组非线性滤波器，复杂度随着深度的增加而增加。

$N_l$表示第$l$层的滤波器个数；$M_l$表示第$l$层的特征图尺寸（$H\times{W}$）；则第$l$层的输出可以表示为一个矩阵$F^l\in\mathcal{R}^{N_l\times{M_l}}$，其中$F^l_{ij}$可以表示为$l$层在位置$j$的第$i$个激活值。

有原始图片$\vec{p}$和随机噪声$\vec{x}$，其对应的第$l$层的输出分别是$F^l$和$P^l$。则内容损失可以表示为：

$$
\mathcal{L_{content}}(\vec{p},\vec{x},l)=\frac{1}{2}\sum_{i,j}(F^l_{ij}-P^l_{ij})^2
$$

$\mathcal{L}_{content}$可以对输入$\vec{x}$求导，以使$\vec{x}$在网络中产生的响应的逼近$\vec{p}$产生的。

**文章使用经过目标检测预训练的网络，这样的网络在不同层学到不同的信息。在浅层，网络可以捕捉到具体的像素值；在深层，网络能够捕捉到高级的内容信息，如目标及其分布；而且这两种信息间具有很小的相关性。涉及到不同的重建问题，需要的信息也各不相同。对于图像风格迁移，显然应该使用深层的、高级的内容信息。**

由下图可以看到，越是浅层的特征就可以越好地重建出图像本身的信息，越是深层的特征就越可以重建出图像风格。

![Figure 1](https://github.com/mengyangniu/images/blob/master/Imagestyletransfer_Figure1.PNG?raw=true)

### 风格表达

使用不同滤波器响应之间的相关性来表达图像风格，组成特征空间$G^l\in\mathcal{R}^{N_l\times{N_l}}$。其中$G^l_{ij}$是$l$层的向量化特征图$i$和向量化特征图$j$之间的内积，即第$l$层的图像风格表达：

$$
G^l_{ij}=\sum_k{F^l_{ik}F^l_{jk}}
$$

有原始图片$\vec{a}$和$\vec{x}$，对应的第$l$层图像风格表达为$A^l$和$G^l$。则$l$层所产生的损失为：

$$
E_l=\frac{1}{4N^2_lM^2_l}\sum_{i,j}(G^l_{ij}-A^l_{ij})^2
$$

总的风格损失为：

$$
\mathcal{L}_{style}(\vec{a},\vec{x})=\sum_{l=0}^L{\omega_lE_l}
$$

其中$\omega$为权重。

### 风格迁移

有风格图片$\vec{a}$和照片$\vec{p}$，以及随机噪声$\vec{x}$，内容权重$\alpha$和风格权重$\beta$，则有：

$$
\mathcal{L}_{total}(\vec{p},\vec{a},\vec{x})=\alpha\mathcal{L}_{content}(\vec{p},\vec{x})+\beta\mathcal{L}_{style}(\vec{a},\vec{x})
$$
文章使用L-BFGS对$\mathcal{L}_{total}$进行优化，对$\vec{x}$求导，记得到了具有$\alpha$的内容、具有$\beta$风格的图片$\vec{x}$。

最终得到的效果是这样的：

![Figure 3](https://github.com/mengyangniu/images/blob/master/ImageStyleTransfer-Figure3.PNG?raw=true)

### 层级选择

先前提到过，越是浅层的特征就可以越好地重建出图像本身的信息，越是深层的特征就越可以重建出图像风格。，选择不同的层计算损失$\mathcal{L}_{content}$和$\mathcal{L}_{style}$对生成图像有很大的影响。

#### 用于重建风格的特征选取

文章选取了层‘conv1\_1’, ‘conv2\_1’, ‘conv3\_1’, ‘conv4\_1’和 ‘conv5\_1’计算$\mathcal{L}_{style}$，能产生主观感受更好的图片。

#### 用于重建内容的特征选取

在保持$\alpha/\beta=0.001$的前提下，分别取‘conv2\_2’和‘conv4\_2’用于计算$\mathcal{L}_{content}$，结果如下：

![Figure 5](https://github.com/mengyangniu/images/blob/master/ImageStyleTransfer-Figure5.PNG?raw=true)

可以看到，选取低层特征能更好地保留图像细节，选取高层图像能获得更抽象的图像。

### 初始化

可以从白噪声初始化，也可以从风格图像或内容图片初始化，生成的图像并没有什么区别。

![Figure 6](https://github.com/mengyangniu/images/blob/master/ImageStyleTransfer-Figure6.PNG?raw=true)

### 照片风格转换

该方法也可以用于照片风格转换，$\alpha/\beta$设为了0.02，比较好理解，因为需要保留更多的照片细节。

![Figure 7](https://github.com/mengyangniu/images/blob/master/ImageStyleTransfer-Figure7.PNG?raw=true)

### 讨论

1. 高分辨率图像导致计算量巨大；
2. 合成图片受到噪声的影响（艺术风格迁移的时候还好，但是照片风格迁移的时候较严重），但是噪声比较典型，应该可以通过图像处理的方式去除；
3. 图像风格和内容的分界线不清晰；
4. 风格转换的成功为生物学中人类视觉原理的研究提供了一条可以切入的点。

### 我的疑问

#### 关于$\mathcal{L_{content}}$

内容损失表示为：

$$
\mathcal{L_{content}}(\vec{p},\vec{x},l)=\frac{1}{2}\sum_{i,j}(F^l_{ij}-P^l_{ij})^2
$$

为什么前面的系数是$\frac{1}{2}$？如若改为：

$$
\mathcal{L_{content}}(\vec{p},\vec{x},l)=\frac{1}{H\times{W}}\sum_{i,j}(F^l_{ij}-P^l_{ij})^2
$$

其中$H$、$W$分别为对应特征图的尺寸，则$\mathcal{L_{content}}$就可以很好地表示为特征图之间的MSE，分明更具实际意义。

猜想：是否因为各层特征图尺寸不一，统一改为$\frac{1}{2}$能比较方便地处理权重$\omega$以及参数如$\alpha/\beta$？

#### 关于$\mathcal{L_{style}}$

图像风格表达为：$G^l_{ij}=\sum_k{F^l_{ik}F^l_{jk}}$，可以看出，其实就是第$i$个特征图和第$j$个特征图的内积，表达到向量上那就是向量的线性相关性，为什么特征图之间的线性相关性可以表示风格？这里面是什么原理？是否有其他办法可以有类似的效果？

希望看完Perceptual Loss之后可以有所解答。