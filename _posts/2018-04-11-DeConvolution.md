---
title: About DeConvolution(Transposed Convolution)
date: 2018-04-11 14:48:00
categories:
- DeepLearning
tags:
- DeepLearning
- SuperResolution
- CNN
mathjax: true
---

<center>关于卷积、反卷积、转置卷积的简单理解。</center>

<!-- more -->

### 卷积

考虑如图卷积：

![](https://raw.githubusercontent.com/vdumoulin/conv_arithmetic/master/gif/no_padding_no_strides.gif)

4\*4的输入，2\*2的输出，padding=0，no strides。

将4\*4的输入特征图展为16\*1的向量$X$，将2\*2的输入特征图展为4\*1的向量$Y$。

则卷积过程可以表示为$Y=CX$，C是一个4\*16的稀疏矩阵：

![](https://pic2.zhimg.com/80/fcf85c4e66326ad5279563b480a80ae1_hd.jpg)

而在反向传播中，
$$
\frac{\partial Y}{\partial X}=C^T
$$
得出结论：卷积的平移不变形结合pooling操作，对图片提取特征，或对特征图降维。卷积操作是正向传播时$Y=CX$，反向传播时左乘$\partial Y=C^T\partial X$。

### 反卷积

反卷积也可以称为转置卷积（Transposed Convolution）。举个栗子：

<img src="https://raw.githubusercontent.com/vdumoulin/conv_arithmetic/master/gif/no_padding_no_strides_transposed.gif" style="zoom:80%"/>

<img src="https://raw.githubusercontent.com/vdumoulin/conv_arithmetic/master/gif/padding_strides_transposed.gif" style="zoom:65%"/>

反卷积就是通过padding和stride操作再进行卷积，使输出尺寸大于输入尺寸。在很多paper里都有用到这个，尤其是image to image的应用，只是很多paper不这么叫它罢了。

反卷积相对于卷积在神经网络结构的正向和反向传播中做相反的运算。

换句话说，反卷积在正向传播中左乘$C^T$，在反向传播中左乘${C^T}^T$，恰好与卷积操作形成转置，因此又成为转置卷积（Transposed Convolution）。