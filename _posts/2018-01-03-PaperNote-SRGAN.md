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

1. 使用GAN来完成SR任务
2. 提出了由对抗损失（Adversarial Loss）和内容损失（Content Loss）组成的感知损失（Perceptual Loss）
3. 提出MOS代替PSNR

### 方法

$I^{HR}$表示原始的高分辨率图，$I^{LR}$表示对应的低分辨率图，$I^{SR}$表示由$I^{LR}$得到的超分辨率图，r为上采样率。

$G_{\theta_{G}}$为生成网络，是一个前馈CNN，参数为$\theta_{G}$，$\theta_G=\{W_{1:L};b_{1:L}\}$，表示第$L$层的权重和偏置项。对于图像$I_n^{HR}$和$I_n^{LR}$，训练过程可以表示为：

$$
\hat{\theta}_G=arg\min_{\theta_G}\frac{1}{N}\sum_{n=1}^Nl^{SR}(G_{\theta_G}(I_n^{LR}),I_n^{HR})
$$

其中$l^{SR}$为Perceptual Loss。

#### 对抗网络

有对抗网络$D_{\theta_D}$和生成网络$G_{\theta_{G}}$，则网络训练可以表示为解决对抗的min-max问题：
$$
\min_{\theta_G}\max_{\theta_D}&\mathbb{E}_{I^{HR}\sim{p_{train}(I^{HR})}}[logD_{\theta_D}(I_{HR})]\\&+\mathbb{E}_{I^{LR}\sim{p_G(I^{LR})}}[log(1-D_{\theta_D}(G_{\theta_G}(I^{LR})))]
$$
