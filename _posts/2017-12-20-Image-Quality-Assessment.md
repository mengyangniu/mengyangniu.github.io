---
title: Image Quality Assessment
data: 2017-12-20 21:00:00
categories:
- SuperResolution
- DeepLearning
tags:
- SuperResolution
- DeepLearning
mathjax: true
---

<center><i>受到程博启发，把一些常见的图像评估标准都了解下。</i></center>

<!-- more -->

## PSNR

PSNR（Peak Signal to Noise Ratio），峰值信噪比。MSE（Mean Square Error），均方误差。

有图像X和参考图像Y，H、W分别为图像的高度和宽度，n为每像素的比特数，一般的灰度图像取8，即像素灰阶数为256。有

$$
MSE=\frac{1}{H\times{W}}\sum_{i=1}^H{\sum_{j=1}^{W}({X(i,j)-Y(i,j)})^2}
$$

$$
PSNR=10log_{10}(\frac{(2^n-1)^2}{MSE})
$$

其中PSNR的单位为dB，数值越大表示失真越小。

PSNR是最普遍和使用最为广泛的一种图像客观评价指标，然而它是基于对应像素点间的误差，即基于误差敏感的图像质量评价。由于并未考虑到人眼的视觉特性（人眼对空间频率较低的对比差异敏感度较高，人眼对亮度对比差异的敏感度较色度高，人眼对一个区域的感知结果会受到其周围邻近区域的影响等），因而经常出现评价结果与人的主观感觉不一致的情况。在超分辨率领域有被Perceptual Loss取代的趋势。

## SSIM

SSIM（structural similarity index）是一种基于结构相似性的图像质量评价。

#### 结构相似性（structural similarity）

自然图像具有极高的结构性，表现在图像的像素间存在着很强的相关性，尤其是在空间相似的情况下。这些相关性在视觉场景中携带着关于物体结构的重要信息。我们假设人类视觉系统（HSV）主要从可视区域内获取结构信息。所以通过探测结构信息是否改变来感知图像失真的近似信息。

大多数的基于误差敏感度（error sensitivity）的质量评估方法（如MSE,PSNR）使用线性变换来分解图像信号，这不会涉及到相关性。SSIM就是要找到更加直接的方法来比较失真图像和参考图像的结构。

#### SSIM指数

![]()

SSIM指数的测量可由三种对比模块组成，分别是：亮度，对比度，结构。带比对测量的图分别为x, y。

##### 亮度

用平均灰度来作为亮度测量的估计：

$$
\mu_x=\frac{1}{N}\sum_{i=1}^N{x_{i}}, \mu_y=\frac{1}{N}\sum_{i=1}^N{y_{i}}
$$

亮度对比函数l(x,y)：

$$
l(x,y)=\frac{2\mu_x\mu_y+C_1}{\mu_x^2+\mu_y^2+C_1}
$$

常数$C_1$是为了避免$\mu_x^2+\mu_y^2$接近0时造成的不稳定，通常取：

$$
C_1=(K_1L)^2, K_1=0.01
$$

##### 对比度

用标准差来做对比度估量值：

$$
\sigma_x=(\frac{1}{N-1}\sum_{i=1}^N(x_i-\mu_x)^2)^{\frac{1}{2}}
$$

$$
\sigma_y=(\frac{1}{N-1}\sum_{i=1}^N(y_i-\mu_y)^2)^{\frac{1}{2}}
$$

对比度对比函数：

$$
c(x,y)=\frac{2\sigma_x\sigma_y+C_2}{\sigma_x^2+\sigma_y^2+C_2}, 其中通常C_2=(K_2L)^2, K_2=0.03
$$

##### 结构

结构对比函数：

$$
s(x,y)=\frac{\sigma_{xy}+C_3}{\sigma_x\sigma_y+C_3}, 其中\sigma_{xy}=\frac{1}{N-1}\sum_{i=1}^N(x_i-\mu_x)(y_i-\mu_y), 通常C_3=\frac{C_2}{2}
$$

上述对于8-bit灰度图像，L=255。

##### 相似测量函数前提条件

$$
S(x,y)=f(l(x,y),c(x,y),s(x,y))
$$

需要满足

$$
对称性：S(x,y)=S(y,x)
$$

$$
有界性：S(x,y)\le1
$$

$$
最大值唯一性：当且仅当x=y时，S(x,y)=1
$$

上述$l(x,y), c(x,y), s(x,y)$皆满足上述3个条件。

##### 完整的相似测量函数

$$
SSIM(x,y)=[l(x,y)]^\alpha[c(x,y)]^\beta[s(x,y)]^\gamma
$$

令$\alpha=\beta=\gamma=1$，对于8-bit灰度图像可以得到：

$$
SSIM(x,y)=\frac{(2\mu_x\mu_y+C_1)(2\sigma_x\sigma_y+C_2)(\sigma_{xy}+C_3)}{(\mu_x^2+\mu_y^2+C_1)(\sigma_x^2+\sigma_y^2+C_2)(\sigma_x\sigma_y+C_3)}
$$

$$
(C_1=6.5025, C_2=58.5225, C_3=29.26125)
$$

SSIM的取值范围为[0,1]值越大则失真越小。

## MSSIM

> 在图像质量评估之中，局部求SSIM指数的效果要好于全局。第一，图像的统计特征通常在空间中分布不均；第二，图像的失真情况在空间中也是变化的；第三，在正常视距内，人们只能将视线聚焦在图像的一个区域内，所以局部处理更符合人类视觉系统的特点；第四，局部质量检测能得到图片空间质量变化的映射矩阵，结果可服务到其他应用中。
> 在实际应用中，可以利用滑动窗将图像分块，令分块总数为N，考虑到窗口形状对分块的影响，采用加权计算每一窗口的均值、方差以及协方差，权值$\omega_{ij}$满足$\sum_i\sum_j\omega{ij}=1$，通常采用高斯核，然后计算对应块的结构相似度SSIM，最后将平均值作为两图像的结构相似性度量，即平均结构相似性MSSIM： 

$$
\mu_x=\sum_{i=1}^N\omega_ix_i
$$

$$
\sigma_x=(\sum_{i=1}^N{\omega_i(x_i-\mu_x)^2})^{\frac{1}{2}}
$$

$$
\sigma_{xy}=\sum_{i=1}^N{\omega_i(x_i-\mu_x)(y_i-\mu_y)}
$$

> 应用这种加窗方法，映射矩阵就可展现出局部各向同性的性质。

$$
MSSIM(x,y)=\frac{1}{MN}\sum_{i=1}^N{SSIM(x_i,y_i)}
$$

其中N为局部窗口的数量。

## SROCC



## KROCC



## PLCC



## rMSE



## Perceptual Loss

