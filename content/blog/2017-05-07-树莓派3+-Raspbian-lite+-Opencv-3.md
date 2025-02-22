---
layout: article
title: "树莓派3 + raspbian lite + OpenCV 3"
date: 2018-01-01
tags:
  - python-opencv
  - 环境搭建
  - 树莓派
lang: zh-Hans
---
**本教程仅针对已经成功刷写树莓派raspbian操作系统的树莓派3,且为raspbian lite系统,且仅针对opencv 3(在本人尝试中,opencv2.4.9并不适用于本教程,而Opencv2.4.9与Opencv3存在一些差异,例如:某些库文件的增减问题.请读者务必在刷写前注意,以免浪费大量时间)**

**因为编译安装opencv中存在耗时较长步骤,如果使用ssh对树莓派进行操作,推荐安装 screen 程序,以免发生掉线问题无法得知当前进度,本教程不赘述screen使用方法,如有需要,敬请google**

## 第一步 安装依赖环境

1.首先的首先是update和upgrade已经存在的一些packages:

	$ sudo apt-get update
	$ sudo apt-get upgrade

2.安装一些开发工具,比如说`cmake`

	$ sudo apt-get install build-essential cmake pkg-config

3.然后安装一些图片格式类型库,比如说jpeg,png等等

	$ sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev

4.图片库好了,当然需要安装一些视频库

	$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
	$ sudo apt-get install libxvidcore-dev libx264-dev

5.opencv的库建立于一个叫`highgui`的次模组(sub-module),为了完全编译`highgui`,需要安装GTK开发库

*此处耗时较长*

	$ sudo apt-get install libgtk2.0-dev

6.opencv里的许多操作可以通过安装下面的库来实现最优化

	$ sudo apt-get install libatlas-base-dev gfortran

7.最后是需要安装python2.7-dev来编译opencv

	$ sudo apt-get install python2.7-dev

## 第二步 下载opencv的源代码

*本教程使用opencv3.1.0版本,你可以使用最新版代替*

	$ cd ~
	$ mkdir opencv
	$ cd opencv
	$ wget -O opencv.zip https://github.com/Itseez/opencv/archive/3.1.0.zip
	$ unzip opencv.zip

为了完整安装opencv,还需要opencv_contrib

**注意:opencv和opencv_contrib的版本号必须一致**

	$ wget -O opencv_contrib.zip https://github.com/Itseez/opencv_contrib/archive/3.1.0.zip
	$ unzip opencv_contrib.zip

## 第三步 配置python

首先安装pip

	$ wget https://bootstrap.pypa.io/get-pip.py
	$ sudo python get-pip.py

然后通过pip安装numpy

*此处耗时较长*

	$ sudo pip install numpy

## 第四步 编译,安装opencv

通过CMake构建opencv

	$ cd ~/opencv/opencv-3.1.0/
	$ mkdir build
	$ cd build
	$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
	    -D CMAKE_INSTALL_PREFIX=/usr/local \
	    -D INSTALL_PYTHON_EXAMPLES=ON \
	    -D OPENCV_EXTRA_MODULES_PATH=~/opencv/opencv_contrib-3.1.0/modules \
	    -D BUILD_EXAMPLES=ON ..

完成后检查CMake输出中是否存在如下图所示的信息:

![](http://ww1.sinaimg.cn/large/005L13Yhgy1ffd8ap1fw6j30n402l3yh.jpg)

*如果没有没有构建成功,且确保之前的步骤正确,可以尝试重启树莓派*

**万事俱备,只欠东风**

	$ make -j4

*-j4可以使用4个核心提高速度,但是容易发生错误,如不成功,可以去掉后重试*

	$ make clean
	$ make

编译没出现错误成功完成后

	$ sudo make install
	$ sudo ldconfig

## 第五步 检验安装

	$ python
	>>> import cv2
	>>> cv2.__version__
	//此时如果出现'3.1.0'则安装成功


本文参考链接:

http://www.pyimagesearch.com/2016/04/18/install-guide-raspberry-pi-3-raspbian-jessie-opencv-3/

(原文包含多版本python环境时虚拟python环境的运用)

以上. 