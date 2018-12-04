---
title: MSE, MLE and KLD
date: 2018-12-04 11:00:00
categories:
- MachineLearning
tags:
- MLE
mathjax: true
---

<center>"A Note for MSE, MLE and KLD"</center>

<!-- more -->

### MLE和KLD

假设随机变量$Y$有条件概率密度函数$f(y|\theta)$。

若有$Y$的N个独立随机观测值$y_1,y_2,y_3,...y_N$，则有似然函数
$$
l(\theta)=\prod_{i=1}^{N}f(y_i|\theta)
$$

则MLE(maximum likelihiid estimation)即为

$$
\theta=\max_\theta l(\theta)=\max_\theta \prod_{i=1}^Nf(y_i|\theta)=\max_\theta\sum_{i=1}^Nln(f(y_i|\theta))
$$

若假设$N\rightarrow+\infty$，且Y有真实分布的概率密度函数$g(y)$，则

$$
\theta=\max_\theta\lim_{N\rightarrow+\infty}\frac{1}{N}\sum_{i=1}^Nln(f(y_i|\theta))=\max_\theta E(ln(f(y|\theta)))
$$

其中

$$
E(ln(f(y|\theta)))=\int g(y)ln(f(y|\theta))dy
$$

则衡量随机变量Y的真实概率分布$g(y)$与假设的基于$\theta$的条件概率密度函数之间$f(y|\theta)$的相对熵(relative entropy)或KL散度(Kullback-Leibler Divergence)为：

$$
\begin{split}
KL(g||f)&=E(ln(g(y)))-E(ln(f(y|\theta)))\\&=\int{g(y)ln(g(y))}dy-\int{g(y)ln(f(y|\theta))}dy\\&=\int{g(y)ln\frac{g(y)}{f(y|\theta)}}dy
\end{split}
$$

当$g(y)$等于$f(y|\theta)$时，KL散度为0。若随机变量给定，则$E(ln(g(y)))$是常量，因此极大似然估计(MLE)是当$N\rightarrow+\infty$特殊情况下的KL散度的最小化求解。

### MSE和MLE

设有长度为N的标注序列(可以认为是ground truth) $Y=[y1,y2,...y_N]$，和长度为N的随机变量序列(通常是prediction) $X=[x_1,x_2,...x_N]$，则$X$和$Y$之间的MSE(mean square error)或$l_2 loss$为：

$$
MSE(X,Y)=\frac{1}{N}\sum_{i=1}^N(x_i-y_i)^2=||X-Y||_2^2
$$

可以认为是有可加的独立同分布高斯白噪声$N(\epsilon|0,\sigma^2)$存在于序列Y中，



设模型的长度为N的输入序列为$X=[x_1,x_2,...x_N]$，长度为N的标注序列(可以认为是ground truth) $Y=[y1,y2,...y_N]$，模型为$F$，模型参数$\theta$。则最小化MSE(mean square error)或$l_2 loss$可以表示为：

$$
\min_\theta(MSE(F(X;\theta),Y))=\min_\theta(\frac{1}{N}\sum_i(F(x_i;\theta)-y_i))=\min_\theta|| F(X;\theta)-Y|| _2^2
$$

则网络训练的目的可以看做是：希望ground truth Y尽可能近似于网络的输出$F(X;\theta)$加上独立同分布的高斯白噪声$N(0,\sigma^2)$，即可以将Y的条件概率密度表示为：

$$
p(y| x)=N(y| F(x;\theta), \sigma^2I)
$$

和

$$
p(y_i| x_i)=N(y_i| F(x_i;\theta), \sigma^2)
$$

则在观测序列（即标注，ground truth）$Y=[y_1,y_2,...y_N]$的条件下， 最大似然函数：

$$
\begin{split}
l(\theta)&=\prod_{i=1}^Np(y_i| x_i)\\
&=\prod_{i=1}^N N(y_i| F(x_i;\theta), \sigma^2)
\end{split}
$$

则MLE可以表示为：

$$
\begin{split}
\theta&=\max_\theta l(\theta)\\
&=\max_\theta \prod_{i=1}^N N(y_i| F(x_i;theta), \sigma^2)\\
&=\max_\theta \prod_{i=1}^N \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(y_i-F(x_i;\theta))^2}{2\sigma^2}}\\
&=\min_\theta\sum_{i=1}^N(y_i-F(x_i;\theta))^2
\end{split}
$$

最终又回到MSE。