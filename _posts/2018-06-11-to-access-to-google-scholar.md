---
title: 访问Google Scholar流量异常解决方案
date: 2018-06-11 22:00:00
categories:
- Linux
tags:
- Linux
mathjax: true
---

<center>使用shadowsocks通过vps访问Google Scholar有时会因为流量异常被拒绝访问。</center>

<!-- more -->

一直是自己租vps搭shadowsocks+kcptun来搭梯子的，陆续遇到过一些问题，都一一顺利解决了，于是从来没有做过笔记。今天又遇到了相似的问题，所以就记录一下。

#### 问题描述

1. 能正常访问Google，但是访问Scholar时显示有异常流量被禁止访问；
2. 或者能正常访问Scholar，但是无法打开Scholar的BibTex页面。

猜测：vps所在ip段频繁访问Scholar，可能是有人爬数据之类的。

#### 解决方案

很简单，强制使用ipv6访问对应域名即可。

1. 启用vps的ipv6，网上教程挺多，不加以赘述；
2. 更改host文件：以Ubuntu16.04为例，路径`/etc/hosts`；
3. 上https://raw.githubusercontent.com/lennylxx/ipv6-hosts/master/hosts，查看无法访问的域名的ipv6地址，将其添加到hosts文件中，例如`echo 2404:6800:4008:803::2001 scholar.googleusercontent.com >> /etc/hosts`；
4. 重启vps或者使hosts重新生效，重启shadowsocks、kcptun等即可。
