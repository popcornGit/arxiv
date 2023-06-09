# SurroundOcc：面向自动驾驶的纯视觉3D占据预测网络

最近，多相机三维占据预测（3D Occupancy Prediction）受到了广泛关注。

作为自动驾驶中的基石任务，三维目标检测天然存在无法识别任意形状以类别的物体。

相较于三维目标检测，三维占据预测可以对周围环境进行稠密重建，从而更好地进行感知。

本文提出SurroundOcc方法，我们利用多帧稀疏LiDAR点云自动生成稠密三维占据标签，并以此作为监督信号训练得到基于多相机图像的稠密占据预测网络。

相关代码已经开源，除了公开数据集外，也支持个人数据的稠密占据预测和标签生成。相关demo如下：

## 简介
近年来，随着L2辅助驾驶的量产落地，自动驾驶相关技术飞速发展。

作为感知的重要一环，三维目标检测可以识别大部分常见物体并指导下游进行避障和路径规划。

但随着落地场景越来越复杂，从高速路到城市街道，三维目标检测逐渐暴露了一些弊端。

由于检测任务天然存在长尾效应，非常依赖训练集中的样本类型和分布，三维检测器只能识别有限类别的物体。

但真实世界中物体的种类是无穷的，因此，在没有激光雷达加持的纯视觉方案中，我们需要三维重建来辅助三维目标检测。

相较于实时三维重建常用手段之一的深度估计，三维占据预测这种表示形式有着一些优势。

首先，由于三维占据预测是直接在三维空间表示的，因此天然满足多视角/多相机一致性。

其次，网络可以通过周围语义信息预测出被遮挡物体的形状，这一点在深度估计中是无法做到的。

最后，三维占据预测易于拓展到其它任务上，例如三维语义分割和场景流估计，并且也方便与基于BEV的三维目标检测方案兼容。

为了得到三维占据预测的模型，我们首先需要生成稠密三维占据标签。

尽管TPVFormer[1]利用稀疏LiDAR作为监督信号也可以进行占据预测，但他们的预测结果比较稀疏，无法进行稠密感知，会漏掉一些障碍物。

而另一方面，在SemanticKITTI[2]中提到，对于稠密三维占据的人工标注是非常费时费力的，尤其是大规模工业数据集。

三维任务的标注本身就比较麻烦，稠密占据标注更是难上加难。

因此，本文提出了一套autolabel的方案，仅需要多帧LiDAR和数据集中已有的三维目标检测标签，就可以生成稠密三维占据标签。

有了稠密标签后，我们设计了2D-3D UNet结构的模型进行稠密占据预测。

SurroundOcc在nuScenes[3]在SemanticKITTI[2]数据集上SurroundOcc都达到了当前最佳性能，证明了方法的有效性。

## 三维占据预测方法
![image](https://user-images.githubusercontent.com/48575896/228441635-5bcca6e7-c4f9-4340-8c82-a8b7e582c8d9.png)

上图展示了本文方法的pipeline。

首先利用ResNet-101等网络对多相机图像提取多尺度特征，并利用FPN对二维图像特征进行多尺度融合。

之后在每个尺度上均利用空间注意力机制对多相机特征进行提取，并得到三维体素特征。

最后利用三维反卷积对小尺度体素特征上采样并与大尺度特征进一步融合。

不同尺度的特征会输出不同分辨率的三维占据预测，并利用loss权重衰减的稠密标签进行监督。

### 2D-3D 空间注意力机制
![image](https://user-images.githubusercontent.com/48575896/228449612-f01956d8-8562-4d2e-98bd-d659ea5fae31.png)

很多三维场景重建方法将多视角特征投影到三维空间，并以求和或者求平均的方式得到三维特征。

这类方法假设每个视角的贡献是一样的并且内外参是绝对准确的，但往往在真实场景中，由于物体遮挡，图像模糊，标定误差等因素的存在，这两个假设都很难成立。

为了解决这个问题，本文利用空间注意力机制进行多相机特征融合。

与BEV特征[4]不同的是，本文直接在三维空间进行相应的操作，从而可以更好地保留三维信息。

具体来说，在每一个2D-3D 空间注意力机制层中，对于三维体素空间中的每一个点，根据内外参将其投影到每个相机视角下，并利用可变形注意力机制在每个视角的投影点周围进行特征提取和聚合。

与此同时，本文利用三维卷积对相邻体素特征进行交互，避免了复杂的三维自注意力机制的使用。

### 多尺度占据预测
相较于三维目标检测，三维占据预测这个任务更需要细粒度的三维信息。

因此，我们将2D-3D 空间注意力机制拓展到多尺度形式。

在每个尺度中，本文方法使用不同层数的2D-3D 空间注意力机制层对不同尺度的二维特征进行聚合。

利用三维反卷积对上一级的三维特征进行上采样，并与这一级的特征进行相加得到当前尺度的特征。

在每一级中，网络都会输出一个占据预测结果，并且都会用相应分辨率的标签进行监督。

本文使用交叉熵损失函数和场景类别匹配度损失函数作为监督信号。

因为高分辨的预测是更重要的，因此本文在每一级中使用衰减的损失权重。

## 三维占据标签生成
![image](https://user-images.githubusercontent.com/48575896/228450327-75093eb8-f457-4b0a-8d2c-038db2457fb8.png)

为了得到稠密三维占据预测，我们需要使用稠密三维占据标签对网络进行训练。

上图展示了三维占据标签生成的pipeline，本文方法只需要数据集中已有的三维目标检测和三维分割标签，无需昂贵的稠密三维占据标注。

对于多帧时序激光雷达点云，本文通过三维目标检测标签将场景和物体点云分割开，并利用位姿信息将场景和物体点云分别拼接起来。

之后将两种点云合并起来得到稀疏占据标签。

为了填补空洞和进一步稠密化，本文使用泊松重建和最近邻算法得到稠密占据标签。

### 多帧点云拼接
一种简单的方式是根据每一帧的位姿信息直接将各帧点云拼接起来。

但这种做法忽略了动态物体，只能应用在完全静止的场景中。

为了解决这个问题，本文设计了一个two-stream的方法在体素化之前对多帧点云进行拼接。

具体而言，对于每帧点云，本文根据三维目标检测标签将其划分为场景和物体点云，之后通过每帧的外参和每个物体各自的位姿信息将二者分别变换到世界坐标系。

在世界坐标系下，利用一个序列中所有帧的点云，对物体和场景点云分别进行拼接。

为了得到当前帧的标签，需要将二者投影到当前帧坐标系，并将两个点云拼合起来。

### 语义标签稠密化
![image](https://user-images.githubusercontent.com/48575896/228451923-f58cbaf0-9082-409e-9f03-4831f08877f8.png)

通过多帧点云拼接操作已经可以得到比较稠密的点云，但点云中仍存在比较多的空洞，尤其对于低线数激光雷达点云而言。

因此，本文对点云中每个点计算法向量并利用泊松重建算法对拼接后的点云进一步地稠密化。

泊松重建算法输出三角网格，本文方法对三角网格的顶点进行体素化处理得到不含语义的稠密占据标签。

为了得到稠密语义标签，本文利用最近邻算法在稀疏占据标签中得到稠密占据的语义结果。

上图对比了稀疏激光雷达点云，稀疏占据标签和稠密占据标签的结果。

