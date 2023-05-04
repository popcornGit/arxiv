# LIMOT：一种用于激光雷达-惯性里程计和多目标跟踪的紧耦合系统

![image](https://user-images.githubusercontent.com/48575896/236087911-cf81211d-a8a3-479b-b3bc-3fcbf8f2c69b.png)


# 摘要
本文介绍了一种用于激光雷达-惯性里程计和多目标跟踪的紧耦合系统。

同时定位和建图（SLAM）对于自动驾驶实现是至关重要的。

大多数激光雷达-惯性SLAM算法假定环境是静态的，导致在动态环境中定位不可靠。

此外，运动物体的精确跟踪对于自动驾驶汽车运行的控制和规划具有重要意义。

本项研究提出LIMOT，它是一种紧耦合的多目标跟踪和激光雷达-惯性SLAM系统，其能够精确地估计自车和目标位姿。

首先，本文使用目标检测器生成的3D边界框来表示所有可运动物体，并且使用惯性测量单元（IMU）预积分结果执行激光雷达里程计。

基于滑动窗口中被跟踪目标的历史轨迹，我们执行鲁棒的目标关联。

本文提出一种基于轨迹的动态特征滤除方法，它通过利用跟踪结果来滤除属于运动目标的特征。

然后，进行因子图优化来优化滑动窗口中IMU偏置以及自车和周围目标的位姿。

在KITTI数据集上进行的实验表明，本文方法比我们先前的工作DL-SLOT以及其它SLAM和多目标跟踪基准方法，实现了更好的位姿和跟踪精度。

# 主要贡献
本文的主要贡献总结如下：

1）本文开发了一种紧耦合多目标跟踪和激光雷达-惯性SLAM系统，允许联合估计自车和周围目标的位姿；

2）本文引入了一种方法，其利用近似的目标轨迹来识别和剔除属于运动物体的特征点，同时仍然利用静态目标上的特征点来为扫描匹配提供约束。

![image](https://user-images.githubusercontent.com/48575896/236089007-a10c2bcb-d567-402d-b0be-b87298e9c87d.png)

![image](https://user-images.githubusercontent.com/48575896/236089708-e04c6c5d-8067-418b-ad3d-6ebb6335b9b1.png)

![image](https://user-images.githubusercontent.com/48575896/236089726-6f7b4f8e-659a-4e0a-ac3a-6ecd72467aa7.png)

![image](https://user-images.githubusercontent.com/48575896/236096697-6eb6c2a1-9c0d-43cb-970d-54a34d916ac3.png)

![image](https://user-images.githubusercontent.com/48575896/236096725-357f8f12-7b3d-4aa0-976d-99eb41359e43.png)

![image](https://user-images.githubusercontent.com/48575896/236097076-f771c17f-5bd0-40aa-be7d-cfd487b99738.png)

![image](https://user-images.githubusercontent.com/48575896/236097094-a5d8953a-dcd2-402d-ba19-0af94e166531.png)

![image](https://user-images.githubusercontent.com/48575896/236097136-cdc72edc-6151-4cd9-b2dd-0d9002b74f60.png)

![image](https://user-images.githubusercontent.com/48575896/236097186-5541891c-d025-4fa5-8ce7-41461fffda75.png)

# 总结
本文提出LIMOT，一种用于激光雷达-惯性SLAM和多目标跟踪的紧耦合系统，其能够在滑动窗口内联合优化自车和周围目标的位姿。

此外，该方法可以基于近似的目标轨迹剔除属于运动目标的特征点，同时静态物体上的剩余特征点用于为扫描匹配提供约束，这增强了系统的鲁棒性，并且提高了其在动态环境中的性能。

实验结果表明，本文方法提高了自车和目标的位姿精度以及跟踪精度，这表明激光雷达-惯性SLAM和多目标跟踪可以相互发挥优势。

在未来，引入针对环境中运动目标的动态模型以获取更精确的目标位姿将是非常有利的。未来的工作还可能涉及利用视觉数据来辅助目标检测和激光雷达-惯性SLAM。
