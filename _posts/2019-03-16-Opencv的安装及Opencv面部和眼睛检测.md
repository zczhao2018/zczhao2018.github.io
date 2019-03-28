---
layout:     post                    # 使用的布局（不需要改）
title:      Opencv的安装及Opencv面部和眼睛检测               # 标题 
subtitle:   Hello World, Hello Blog #副标题
date:       2017-03-29              # 时间
author:     zc                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - OpenCV
---
这篇教程中，主要记录我花了三天三夜在Raspberry Pi Zero安装opencv和opencv的扩展模块的过程，这个过程是很痛苦的，因为Pi Zero的处理速度真是感人。。。，其他树莓派和Ubuntu系统是通用的，具体安装时间是和自己的机器速度有关系的。一般来说，一个小时差不多可以安装完成。楼主使用的机器是Pi Zero（欲哭无泪）。

（题外话：）
Ubuntu系统下：
cmake是用来编译opencv源码的，编译完源码，会生成很多的.so文件，以后使用opencv时，包含需要的头文件，链接到opencv的库文件就可以了，这样每次不用重新变异opencv的源文件
Windows系统下：
1.Windows系统下用可视化的Cmake编译opencv源码，会有很多的.dll文件，这些在你建工程时，需要添加到工程的属性配置中
2.当然也可以直接下载自解压缩包，就不用自己再编译了，建议自己编译一下

# 第1步：下载并安装Raspbian映像

 - 从raspberry pi网站下载带有桌面图像的raspbian stretch
 https://www.raspberrypi.org/downloads/raspbian
 - 然后将存储卡插入笔记本电脑并使用	烧录工具把镜像文件写入内存卡中
 Windows使用Win32 Disk Imager
Mac使用 https://etcher.io
 - 烧录后，将存储卡插入树莓派中，然后打开系统

# 第2步：安装Opencv
（树莓派因为有些源的问题，一定要确保每一步依赖包都已经安装过之后，再进行下一步，下一步，因为如果缺少了某一个包，你需要重新安装所有的，直接安装缺少的依赖包无效，切记，ubuntu系统则没有这种问题，不过，确认一下总是没错的，可以多装，不可以少装）
 ## 1.每次启动任何新安装时，最好升级现有软件包
```
sudo apt-get update
sudo apt-get upgrade
```
Tims：2m 30 sec

## 2.然后安装开发者工具
```
sudo apt-get install build-essential cmake pkg-config
```
Time: 50 sec

## 3.现在安装必要的图像I / O包
```
sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
```
Time: 37 sec

## 4.视频I / O包
```
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
```
Time: 36 sec

## 5.安装GTK开发
```
sudo apt-get install libgtk2.0-dev
```
Time: 2m 57s

## 6.优化包
```
sudo apt-get install libatlas-base-dev gfortran
```
Time: 1 min

## 7.如果不存在的话，最好安装python 2.7。 
```
sudo apt-get install python2.7-dev
```
Time: 55 sec

## 8.现在下载opencv源并解压缩
 https://github.com/opencv
 opencv3.3之前的代码和之后的代码会有一些区别，需要注意, 安装的过程中把 https://github.com/opencv/opencv_contrib 也安装上比较好
## 9.编译opencv
Contrib模块正在不断开发中，建议将它们与主分支或最新版本的OpenCV一起使用。

这是给你的CMake命令：

```
cd <opencv_build_directory>
cmake -D OPENCV_EXTRA_MODULES_PATH=<opencv_contrib>/modules <opencv_source_directory>
make -j5
```
结果，OpenCV将在<opencv_build_directory>中构建，其中包含来自opencv_contrib存储库的所有模块
在解压好的opencv中建立一个build文件夹，-D OPENCV_EXTRA_MODULES_PATH=（这里是contrib文件夹里modules的地址），之后不要忘了opencv源文件的路径，即上一级目录 .. , make后面加上cpu的核数
## 10.安装opencv
```
sudo make install 
```
查看opencv的版本号

```
pkg-config --modversion opencv
```
如果有版本号，则说明opencv安装成功

# 第3步：面部和眼部检测
https://docs.opencv.org/2.4/doc/tutorials/objdetect/cascade_classifier/cascade_classifier.html
opencv3.3代码有变化，需要注意
（后续会补充）
以下是运行上面代码并使用内置网络摄像头的视频流作为输入的结果：
![在这里插入图片描述](https://img-blog.csdn.net/20181005203600100?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW96aGljaGVuZ2hwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
复制haarcascade_frontalface_alt.xml和haarcascade_eye_tree_eyeglasses.xml文件。它们位于opencv / data / haarcascades中
![在这里插入图片描述](https://img-blog.csdn.net/20181005203832875?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW96aGljaGVuZ2hwdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
### 参考
(https://www.instructables.com/id/Face-and-Eye-Detection-With-Raspberry-Pi-Zero-and-/)
-(https://docs.opencv.org/2.4/modules/contrib/doc/facerec/facerec_tutorial.html)
-[OpenCV](https://docs.opencv.org/2.4/doc/tutorials/objdetect/cascade_classifier/cascade_classifier.html)
https://github.com/opencv
