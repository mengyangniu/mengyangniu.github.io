---
title: MLE and KLD
date: 2018-12-04 11:00:00
categories:
- MachineLearning
tags:
mathjax: true
---

<center>"A Note for MLE and MinimumKLD"</center>

<!-- more -->

假设随机变量Y有条件概率密度函数$f(y|\theta)$。若有Y的N个独立随机观测值$y_1,y_2,y_3,...y_N$，则有似然函数

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

当$g(y)$等于$f(y|\theta)$时，KL散度为0。若随机变量给定，则$E(ln(g(y)))$是常量，因此极大似然估计(MLE)是当$N\rightarrow+\infty​$特殊情况下的KL散度的最小化求解。