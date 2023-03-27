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

### Virtual Camera
![350eb82dad932daf28c2b7a35f0f01fd](https://user-images.githubusercontent.com/48575896/227886068-66f357f8-7205-45f4-a55c-b089b3d9a485.png)

不同车辆的内/外在参数是不同的，这对3D车道线的结果有很大影响。
不同于将摄像机内、外参数整合到网络特征中的方法不同(PersFormer、LSS等)，本文实现了一种统一相机内外参的预处理方法，即通过建立具有固定内外参数的虚拟相机(Virtual Camera)完成各个相机的图像内/外参数的快速统一。

论文假设![image](https://user-images.githubusercontent.com/48575896/227887048-6fbc7aa5-efb2-4091-8299-146de64c6afc.png)是与当地路面相切的平面。路面的切线。由于3D车道检测更关注![image](https://user-images.githubusercontent.com/48575896/227887071-39d221b8-b45c-4d3c-be36-843a41022116.png)平面，论文利用homography矩阵的共面性将当前摄像机的图像通过homography 投影到虚拟摄像机的图像中。

如下图所示：通过homography矩阵将当前摄像机的图像投射到虚拟像机的视图中。因此，Virutal Camera实现了不同像机的空间关系的一致性。

虚拟像机的内参和外参是固定的，这些参数是由训练数据集的内/外参数的平均值算出来的。

在训练和推理阶段，根据当前相机提供的相机![image](https://user-images.githubusercontent.com/48575896/227889503-bb9f7481-8271-438e-abd0-e824a6b00a62.png)以及虚拟相机的内/外参来计算![image](https://user-images.githubusercontent.com/48575896/227889447-97487d23-3373-4d90-bc7e-a069b9d492d2.png)

具体的，首先，论文在BEV平面![image](https://user-images.githubusercontent.com/48575896/227889606-12972ccf-64bd-4b27-b3bb-c2ab1fd8d823.png)上选择四个点![image](https://user-images.githubusercontent.com/48575896/227889647-f5fe1026-3101-454a-9117-9169c9b6666b.png)，其中![image](https://user-images.githubusercontent.com/48575896/227889719-855dd700-4176-4205-8285-681ea845d370.png)。

最后，通过最小二乘法得到![image](https://user-images.githubusercontent.com/48575896/227889840-1fd1b6ff-85c3-485a-9ed4-a4b831d837e3.png)，如公式1所示：

![image](https://user-images.githubusercontent.com/48575896/227889881-b87edec2-f84a-407a-b65c-c407146aa4e1.png)

实际上，在推理的时候，把原始相机的图像和传入到OpenCV的库函warpPerspective即可得到虚拟相机下的图像。

关于Virtual Camera的效果，可以通过消融实验看出，对x_error(-0.5m)和F1-Score(+3.3)都有着比较明显的效果。

![93a70b242bd7177d1496beae1b6ac52c](https://user-images.githubusercontent.com/48575896/227891841-49c46b55-8b2c-4067-aac5-6622411927a2.png)

### MLP Based Spatial Transformation Pyramid
