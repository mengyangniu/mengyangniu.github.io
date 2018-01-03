---
title: PaperNote - SRGAN
data: 2018-01-03 10:16:00
categories:
- PaperNote
tags:
- DeepLearning
- PaperNote
mathjax: true
---



<center>Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network</center>



<!-- more -->



### Abstract

文章的重点在于：

1. 设计了SRResNet网络，可以单独实现超分辨率，仅使用MSE训练即可以在PSNR或SSIM上做到state of the art；
2. 使用SRResNet作为GAN中的G，将GAN应用于SR问题；
3. 使用基于特征图计算的Perceptual Loss替代MSE；
4. 提出PSNR的不合理性，并提出了MOS作为额外的测试手段。

### 方法

$I^{HR}$表示原始的高分辨率图，$I^{LR}$表示对应的低分辨率图，$I^{SR}$表示由$I^{LR}$得到的超分辨率图，r为上采样率。

$G_{\theta_{G}}$为生成网络，是一个前馈CNN，参数为$\theta_{G}$，$\theta_G=\{W_{1:L};b_{1:L}\}$，表示第$L$层的权重和偏置项。对于图像$I_n^{HR}$和$I_n^{LR}$，训练过程可以表示为：

$$
\hat{\theta}_G=arg\min_{\theta_G}\frac{1}{N}\sum_{n=1}^Nl^{SR}(G_{\theta_G}(I_n^{LR}),I_n^{HR})
$$

其中$l^{SR}$为Perceptual Loss。

#### 网络结构

有对抗网络$D_{\theta_D}$和生成网络$G_{\theta_{G}}$，则网络训练可以表示为解决对抗的min-max问题：

$$
\min_{\theta_G}\max_{\theta_D}\mathbb{E}_{I^{HR}\sim{p_{train}(I^{HR})}}[logD_{\theta_D}(I_{HR})]+\mathbb{E}_{I^{LR}\sim{p_G(I^{LR})}}[log(1-D_{\theta_D}(G_{\theta_G}(I^{LR})))]
$$

对抗网络和生成网络分别为：

![NetWorks](https://github.com/mengyangniu/images/blob/master/SRGAN-Figure4.png?raw=true)

#### Perceptual Loss Function

总的损失函数如下，由content loss和adversarial loss组成：

$$
l^{SR}=\underbrace{\underbrace{l^{SR}_X}_{content\ loss}+\underbrace{10^{-3}l^{SR}_{Gen}}_{adversarial\ loss}}_{perceptual loss(for\ VGG\ based\ content\ loss)}
$$

##### Content Loss

最广泛使用Content Loss是像素级别的MSE loss function：

$$
l^{SR}_{MSE}=\frac{1}{r^2WH}\sum_{x=1}^{rW}{\sum_{y=1}^{rH}({I_{x,y}^{HR}-G_{\theta_G}(I^{LR})_{x,y})^2}}
$$

然而当PSNR较高时此损失函数无法引入更多的高频信息，这会导致SR图过于平滑，在感知上的质量不足。况且作者通过请26位评判者打分的方式，证明高PSNR并不能带来良好的感官效果。因此针对VGG19，作者提出了新的content loss，基于预训练的19层VGG网络的ReLU激活层来定义损失函数：

$$
l^{SR}_{VGG/i.j}=\frac{1}{W_{i,j}H_{i,j}}\sum_{x=1}^{W_{i,j}}{\sum_{y=1}^{H_{i,j}}(\phi_{i,j}(I^{HR})_{x,y}-\phi_{i,j}(G_{\theta_G}(I^{LR}))_{x,y})^2}
$$

##### Adversarial Loss

$$
l^{SR}_{Gen}=\sum_{n=1}^{N}{-logD_{\theta_D}(G_{\theta_G}(I^{LR}))}
$$

### 实验

文章的实验基于几个Bench-mark数据集，Set5，Set14，BSD100和BSD300的test集。实验中上采样比例为4x。计算PSNR、SSIM时，使用去除了图像边缘的4像素宽度进行center crop得到的图像计算。

#### 训练

使用ImageNet中350000张图像作为训练集，对每个mini batch，都对原始图像crop出16张不同的96\*96的HR图像。将LR图像归一化到[0,1]，将HR图像归一化到[-1,1]，则MSE的范围就被约束到[-1,1]。VGG的特征图同样乘以一个系数$\frac{1}{12.75}$来使VGG得到的Loss和MSE有相同的尺度，这相当于$l^{SR}_{VGG/i.j}$乘以系数0.006。优化方法使用$\beta_1=0.9$的Adam。使用MSE预训练的SRResNet作为SRGAN中G的初始网络。