---
title: CE, MSE, MLE and KLD
date: 2018-12-04 11:00:00
categories:
- MachineLearning
tags:
- MLE
mathjax: true
---

<center>"A Note for CE, MSE, MLE and KLD"</center>

<!-- more -->

### KLD 和 CE

假设有随机变量y的真实概率分布$g(y)$和非真实（预测）概率分布$f(y\vert\theta)$。

根据真实分布$g$编码服从真实分布$g$的样本所需的期望编码长度（即平均编码长度）为：

$$
H(g)=\sum_yg(y)\times ln(\frac{1}{g(y)})
$$

根据非真实分布$f$编码服从真实分布$g$的样本所需的期望编码长度为：

$$
H(g,f)=\sum_yg(y)\times ln(\frac{1}{f(y\vert\theta)})
$$

其中$H(g,f)$称为Cross Entropy(交叉熵，CE)。且根据gibbs不等式，恒有

$$
H(g,f)\ge H(g)
$$

当$g$与$f$是分布时等号成立。故CE在ML，DL的分类问题中经常作为损失函数，minimum ce就是在使交叉熵$H(g,f)$逼近$H(g)$。

定义两个编码长度的差为相对熵(Relative Entropy)，也称Kullback-Leibler Divergence，即KLD：

$$
KLD(g\Vert f)=H(g,f)-H(g)=\sum_y g(y)\times ln(\frac{g(y)}{f(y\vert\theta)})
$$

KLD的其他表达：

$$
KLD(g\Vert f)=E_{y\sim g(y)}[ln(g(y))-ln(f(y\vert\theta))]
$$

### forward KLD 和 reverse KLD

假设有随机变量y的真实概率分布$g(y)$和非真实（预测）概率分布$f(y\vert\theta)$。

则forward KLD定义为：

$$
\begin{split}
KLD(g\Vert f)
&=E_{y\sim g(y)}[ln(g(y))-ln(f(y\vert\theta))]
\\
&=\sum_yg(y)ln(\frac{g(y)}{f(y\vert\theta)})
\end{split}
$$

backward KLD定义为：

$$
\begin{split}
KLD(f\Vert g)
&=E_{y\sim f(y\vert\theta)}[ln(f(y\vert\theta)-ln(g(y))]
\\
&=\sum_yf(y\vert\theta)ln(\frac{f(y\vert\theta)}{g(y)})
\end{split}
$$

下面分析forward KLD与backward KLD的区别。forward KLD和backward KLD优化的目的都是使预测分布$f$逼近真实分布$g$。但是两者在具体问题的表现上又有所不同。

假设$g$服从混合高斯分布（可以近似看做多个高斯曲线的叠加然后归一化？），而$f$的建模却局限于单高斯。则forward KLD的优化会使$f$去逼近$g$的非零部分，而$g$有多个高斯峰，此时$f$很容易去拟合成一条$sigma$很大的平滑的高斯曲线，在$g$的每一个峰处都有所取值。而backward KLD会使$f$在$g$的某一个峰值处得到高响应而忽视其他峰值，即$f$的$\mu$与$g$中某一个高斯的$\mu$重合且$\sigma$较小。

有文献认为优化backward KLD未必能有更低的loss，但可以给人带来更好的视觉感受（不过于平滑）。有理由认为观察者已经学习到了自然图像的真实分布$g$，可以视作是先验信息。对一张生成图像的评估则是衡量该图像的预测分布$f$与$g$的近似程度，即$KLD(f\Vert g)$，即backward KLD。

### MLE和KLD

假设随机变量$Y$有条件概率密度函数$f(y\vert \theta)$。

若有$Y$的N个独立随机观测值$y_1,y_2,y_3,...y_N$，则有似然函数
$$
l(\theta)=\prod_{i=1}^{N}f(y_i\vert\theta)
$$

则MLE(maximum likelihiid estimation)即为

$$
\theta=\max_\theta l(\theta)=\max_\theta \prod_{i=1}^Nf(y_i\vert\theta)=\max_\theta\sum_{i=1}^Nln(f(y_i\vert\theta))
$$

若假设$N\rightarrow+\infty$，且Y有真实分布的概率密度函数$g(y)$，则

$$
\theta=\max_\theta\lim_{N\rightarrow+\infty}\frac{1}{N}\sum_{i=1}^Nln(f(y_i\vert\theta))=\max_\theta E(ln(f(y\vert\theta)))
$$

其中

$$
E(ln(f(y\vert\theta)))=\int g(y)ln(f(y\vert\theta))dy
$$

则衡量随机变量Y的真实概率分布$g(y)$与假设的基于$\theta$的条件概率密度函数之间$f(y\vert \theta)$的相对熵(relative entropy)或KL散度(Kullback-Leibler Divergence)为：

$$
\begin{split}
KL(g\|f)&=E(ln(g(y)))-E(ln(f(y\vert\theta)))\\&=\int{g(y)ln(g(y))}dy-\int{g(y)ln(f(y\vert\theta))}dy\\&=\int{g(y)ln\frac{g(y)}{f(y\vert\theta)}}dy
\end{split}
$$

当$g(y)$等于$f(y\vert \theta)$时，KL散度为0。若随机变量给定，则$E(ln(g(y)))$是常量，因此极大似然估计(MLE)是当$N\rightarrow+\infty$特殊情况下的KL散度的最小化求解。

### MSE和MLE

设有长度为N的标注序列(可以认为是ground truth) $Y=[y1,y2,...y_N]$，和长度为N的随机变量序列(通常是prediction) $X=[x_1,x_2,...x_N]$，则$X$和$Y$之间的MSE(mean square error)或$l_2 loss$为：

$$
MSE(X,Y)=\frac{1}{N}\sum_{i=1}^N(x_i-y_i)^2=\|X-Y
\|_2^2
$$

可以认为是有可加的独立同分布高斯白噪声$N(\epsilon\vert 0,\sigma^2)​$存在于序列Y中，



设模型的长度为N的输入序列为$X=[x_1,x_2,...x_N]$，长度为N的标注序列(可以认为是ground truth) $Y=[y1,y2,...y_N]$，模型为$F$，模型参数$\theta$。则最小化MSE(mean square error)或$l_2 loss$可以表示为：

$$
\min_\theta(MSE(F(X;\theta),Y))=\min_\theta(\frac{1}{N}\sum_i(F(x_i;\theta)-y_i))=\min_\theta\| F(X;\theta)-Y\| _2^2
$$

则网络训练的目的可以看做是：希望ground truth Y尽可能近似于网络的输出$F(X;\theta)$加上独立同分布的高斯白噪声$N(0,\sigma^2)$，即可以将Y的条件概率密度表示为：

$$
p(y\vert x)=N(y\vert F(x;\theta), \sigma^2I)
$$

和

$$
p(y_i\vert x_i)=N(y_i\vert F(x_i;\theta), \sigma^2)
$$

则在观测序列（即标注，ground truth）$Y=[y_1,y_2,...y_N]$的条件下， 最大似然函数：

$$
\begin{split}
l(\theta)&=\prod_{i=1}^Np(y_i\vert x_i)\\
&=\prod_{i=1}^N N(y_i\vert F(x_i;\theta), \sigma^2)
\end{split}
$$

则MLE可以表示为：

$$
\begin{split}
\theta&=\max_\theta l(\theta)\\
&=\max_\theta \prod_{i=1}^N N(y_i\vert F(x_i;theta), \sigma^2)\\
&=\max_\theta \prod_{i=1}^N \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(y_i-F(x_i;\theta))^2}{2\sigma^2}}\\
&=\min_\theta\sum_{i=1}^N(y_i-F(x_i;\theta))^2
\end{split}
$$

最终又回到MSE。

### 总结

MSE是MLE在高斯白噪声前提下的特殊情况（laplace白噪声前提下MSE变成$l_1loss$），而MLE又是样本$N\rightarrow+\infty$时KLD的特殊情况，因此逐像素误差与MLE在概率、分布的角度上来说是共通的，又从KLD的信息论角度给MLE做出了解释。



### 参考

[1] https://www.zhihu.com/question/41252833

[2] Yang W, Zhang X, Tian Y, et al. Deep Learning for Single Image Super-Resolution: A Brief Review[J]. 2018.