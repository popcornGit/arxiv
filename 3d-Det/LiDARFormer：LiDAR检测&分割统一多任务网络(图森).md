# LiDARFormer：LiDAR检测&分割统一多任务网络（图森）
# 摘要
最近激光雷达感知领域出现了一种趋势，即将多个任务统一在一个强大的网络中，从而提高性能，而不是为每个任务使用单独的网络。

本文介绍了一种新的基于Transformer的激光雷达多任务学习范式。

所提出的LiDARFormer利用跨空间全局上下文特征信息，并利用跨任务协同作用，在多个大规模数据集和基准上提高LiDAR感知任务的性能。

论文新颖的基于Transformer的框架包括跨空间Transformer模块，该模块学习2D密集鸟瞰图（BEV）和3D稀疏体素特征图之间的注意力特征。

此外，论文提出了一种用于分割任务的Transformer解码器，通过利用分类特征表示来动态调整学习的特征。

此外将共享Transformer解码器中的分割和检测特征与跨任务注意力层相结合，以增强和集成目标级和类级特征。

LiDARFormer在大规模nuScenes和Waymo Open数据集上进行了3D检测和语义分割任务的评估，并且在这两项任务上都优于所有先前发表的方法。

值得注意的是，对于单模型LiDAR方法，在具有挑战性的Waymo和nuScenes检测基准上，LiDARFormer实现了76.4%L2 mAPH和74.3%NDS的最新性能。

总结来说，本文的主要贡献如下：

1.在多任务网络中，当在稀疏体素特征和密集边界元特征之间转移特征时，论文提出了一个跨空间Transformer模块来改进特征学习；

2.论文提出了第一个LiDAR跨任务Transformer解码器，该解码器将学习到的信息跨目标级和类级特征嵌入进行桥接；

3.论文介绍了一种基于Transformer的粗到细网络，该网络利用Transformer解码器为激光雷达语义分割任务提取类级全局上下文信息；

4.论文在两个流行的大规模激光雷达基准上实现了最先进的3D检测和语义分割性能。

# 方法
本节将介绍激光LiDARFormer的设计。

如图2所示，框架由三部分组成：

1.使用3D稀疏卷积的3D编码器-解码器骨干网络；

2.一种跨空间Transformer（XSF）模块，用于提取BEV中的大规模和上下文特征；

3.一种跨任务Transformer（XTF）解码器，其从体素和BEV特征图聚合类和对象全局上下文信息。

本文的网络采用了LidarMultiNet[62]的多任务学习框架，但通过共享的跨任务注意力层进一步将分割和检测之间的全局特征相关联。

![image](https://user-images.githubusercontent.com/48575896/229677314-29d06085-7c02-46d9-94d3-1460959a9258.png)

## 基于体素的LiDAR感知
$$ \mathcal{V}_{j}=\max _{\mathcal{I}_{i}=\mathcal{I}_{j}}\left(\operatorname{MLP}\left(p_{i}\right)\right), j \in(1 \ldots M)$$
