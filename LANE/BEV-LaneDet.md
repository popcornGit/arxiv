# BEV-LaneDet: a Simple and Effective 3D Lane Detection Baseline
## Abstract
3D车道检测在自动驾驶轨控中起着至关重要的作用，最近成为自动驾驶领域一个快速发展的话题。

3D车道检测在自动驾驶轨控中起着至关重要的作用，最近成为自动驾驶领域一个快速发展的话题。

面对这些问题，本文提出了一个高效且简单的单目3D车道线检测方法，称为BEV-LaneDet，有三个主要贡献。

首先，引入了Virutal Camera，统一了安装在不同车辆上的相机的内/外参数，以保证相机之间空间关系的一致性。

由于统一的视觉空间，它可以有效地促进网络的学习过程。

其次，提出了一个简单而有效的三维车道表示法，称为关键点表示法(Key-Points Representation), 这个模块更适合于表示复杂多样的三维车道结构。

最后，本文提出了一个轻量级和芯片友好的空间转换模块，名为空间转换金字塔(Spatial Transformation Pyramid)，用于将多尺度的前视特征转换成BEV特征。

实验结果表明，BEV-LaneDet在F-Score方面优于最先进的方法，在OpenLane数据集上高出10.6%，在Apollo 3D合成数据集上高出6.2%，V-100上速度为185FPS。

代码链接：https://github.com/gigo-team/bev_lane_det

总结一下:

1.本文提出了一种简单实用，几乎适用于所有芯片的3D车道线检测方法 BEV-LaneDet。

2.本文提出了Virtual Camera、Spatial Transformation Pyramid、Key-Points Representation三个模块，创新性没有那么大，但是组合起来非常好用。

3.在速度飞快的同时，能做到在公开集OpenLane(+10.6%)和Apollo 3D synthetic上大幅度涨点(+6.2%)，尤其是Apollo 3D synthetic上F-Score几乎已经刷满了。

在总体来讲，这篇文章的工程性大于其学术性。

## Methodology
![image](https://user-images.githubusercontent.com/48575896/227884686-9680d098-0845-4a69-9e9f-463d049624e2.png)

如上图所示，整个网络架构由五个部分组成：

1）虚拟相机（Virtual Camera）：统一相机内参和外参的预处理方法；

2）Backbone：前视特征提取器；

3）空间转换金字塔（Spatial Transformation Pyarmid）。将前视特征投射到BEV特征；

4）关键点表示（Key-Points Representation）：基于关键点的三维车头检测器；

5）2D图像监督：2D车道检测检测头，提供辅助监督。

