---
title:  Linux Deep Learning Enviroment Configuration
date: 2017-12-19 22:01:00
categories:
- Linux
tags:
- Linux
- DeepLearning
- Configuration
- Pytorch
- Mxnet
---

<center>Ubuntu1604, Cuda8/9, Cudnn7, Opencv2/3, Pytorch 0.2.0 / 0.4.0, Mxnet 0.12.*/1.0.0, Caffe</center>

<!-- more -->

# 遇上的坑

- [x] 在Intel i7 6700, GTX1070*2, MSI Z170 M7的平台上，使用启动U盘安装Ubuntu1604可能会遇到无法安装的情况，将两块1070全部拔出，然后使用主板输出视频信号即可装上系统，随后插上显卡安装驱动即可。

# Ubuntu1604
若在国内，换源，随后安装各种相关依赖项以及更新等等：

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler  
sudo apt-get install libboost-all-dev 
sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev 
sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
sudo apt-get install  libopencv-dev libdc1394-22 libdc1394-22-dev libjpeg-dev libpng12-dev libtiff5-dev libjasper-dev libavcodec-dev libavformat-dev libswscale-dev libxine2-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev libv4l-dev libtbb-dev libqt4-dev libfaac-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev x264 v4l-utils unzip
sudo apt-get install ffmpeg libopencv-dev libgtk-3-dev python-numpy python3-numpy libdc1394-22 libdc1394-22-dev libjpeg-dev libpng12-dev libtiff5-dev libjasper-dev libavcodec-dev libavformat-dev libswscale-dev libxine2-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libv4l-dev libtbb-dev qtbase5-dev libfaac-dev libmp3lame-dev libopencore-amrnb-dev
```

随时`sudo apt-get update`

# 显卡驱动

[下载](http://www.nvidia.cn/Download/index.aspx?lang=cn)（个人喜欢.run文件）。

```
sudo service lightdm stop
sudo -s
./*.run
```

即可安装。

若出现“The Nouveau kernel driver is currently in use by your system. This driver is incompatible with the NVIDIA driver, and must be disabled before proceeding.”而导致安装失败，禁用Nouveau kernel即可：

在 /etc/modprobe.d/blacklist.conf 的后面加入：blacklist nouveau，然后`sudo update-initramfs -u`

安装完之后运行`nvidia-smi`，若出现GPU信息，即安装成功。

# CUDA

选择合适的版本[下载](https://developer.nvidia.com/cuda-downloads)到本地后（个人喜欢.run文件），sudo ./*.run即可安装，当选择是否安装驱动时，选择否，其他都选择是。

`sudo vim ~/.bashrc`，添加：

```bash
export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda8.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
```

`sudo vim /etc/profile`，添加`export PATH = /usr/local/cuda/bin:$PATH`

`sudo vim /etc/ld.so.conf.d/cuda.conf`，添加`/usr/local/cuda/lib64`

执行`sudo ldconfig`，重启，执行以查看Cuda安装成功与否。

```bash
cd /usr/local/cuda-8.0/samples/1_Utilities/deviceQuery 
sudo make  
./deviceQuery
```

**若安装Cuda9，将上述所有8.0改成9.0。**

# Cudnn

解压下载后的tar.gz文件，进入解压后的目录

```bash
sudo cp ./include/cudnn.h /usr/local/cuda/include/
sudo cp -a ./lib64/* /usr/local/cuda/ib64/
```

安装完毕

#OpenCV2/3

找个地方建立一个工作目录，执行git，例如

```bash
cd ~
mkdir OpenCV
git clone https://github.com/opencv/opencv.git # 默认opencv3，如需2，设置branch
git clone https://github.com/opencv/opencv_contrib.git
# 需要一段时间，建议使用vps下载
cd opencv/build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local   ..
sudo make -j8 #-j8是指用8个线程
sudo make install
#如果在/usr/local里能看到Opencv的头文件和库文件，即安装完成
```

若卸载
```bash
cd Opencv/opencv/build
make uninstall -j8
```

# PyTorch

### 使用wheel安装

看http://pytorch.org/，建议whl文件先用下载工具下载到本地后再用pip安装，会比较快。

### 使用源码安装

Pytorch

```bash
git clone --recursive https://github.com/pytorch/pytorch
cd pytorch
sudo python setup.py install
```

torchvision

```bash
git clone --recursive https://github.com/pytorch/vision.git
cd vision
sudo python setup.py install
```

# Mxnet

https://mxnet.incubator.apache.org/install/index.html

# caffe

最近不用，懒得写，有空再补。

