# Block-NeRF: Scalable Large Scene Neural View Synthesis

## Abstract
我们提出了Block-NeRF，神经辐射的变体可以表示大规模环境。

具体来说，我们演示了在缩放NeRF以渲染跨越多个街区的城市规模场景时，将场景分解为单独训练的NeRF是至关重要的。

这种分解将渲染时间与场景大小分离，使渲染能够扩展到任意大的环境，并允许对环境的每个块进行更新。

我们采用了几项架构更改，使NeRF对不同环境条件下几个月捕获的数据具有健壮性。

我们为每个单独的NeRF添加了appearance embeddings、 learned pose refinement和可控曝光，并引入了在相邻NeRF之间对齐外观的过程，以便它们可以无缝组合。

我们从280万张图像中构建了一个block - nerf网格，以创建迄今为止最大的神经场景表示，能够渲染整个旧金山社区。

## Introduction
神经渲染方面的最新进展，如神经辐射场[42]，已经能够在给定一组相机图像的情况下实现照片逼真重建和新颖的视图合成[3,40,45]。

早期的工作侧重于小规模和以对象为中心的重建。

虽然现在有些方法可以解决单个房间或建筑物大小的场景，但这些方法通常仍然是有限的，不能完全扩展到城市规模的环境。

由于模型容量有限，将这些方法应用于大型环境通常会导致远古效果和较低的视觉保真度。

重构大规模环境可以在自动驾驶[32,44,68]和航空测量[14,35]等领域实现几个重要的用例。一个例子是建图，其中创建了整个操作域的高保真地图，作为各种问题的强大先验，包括机器人定位、导航和避免碰撞。

此外，大规模场景重建可用于闭环机器人仿真[13]。

自动驾驶系统通常通过重新模拟之前遇到的场景来评估;然而，任何偏离记录的遭遇都可能改变车辆的轨迹，需要沿着改变的路径绘制高保真的新视图。

除了基本的视图合成外，基于场景的nerf还能够改变环境照明条件，如相机曝光、天气或一天中的时间，这可用于进一步增强模拟场景。

重建如此大规模的环境带来了额外的挑战，包括瞬态对象(汽车和行人)的存在，模型容量的限制，以及内存和计算的限制。

此外，在一致的条件下，在一次捕获中收集如此大环境的训练数据是极不可能的。

相反，环境不同部分的数据可能需要来自不同的数据收集工作，在场景几何(例如，建筑工作和停放的汽车)和外观(例如，天气条件和一天的时间)中引入差异。

我们用appearance embeddings和learned pose refinement来扩展NeRF，以解决收集数据中的环境变化和姿态误差。

我们还添加了曝光调节，以提供在推断过程中修改曝光的能力。

我们将这个修正模型称为Block-NeRF。

扩大Block-NeRF的网络容量使其能够表示越来越大的场景。

然而，这种方法也有一些局限性;随着网络的大小呈现耗时增加，网络不再适合单个计算设备，更新或扩展环境需要重新训练整个网络。

为了解决这些挑战，我们建议将大型环境划分为单独训练的block - nerf，然后在推理时动态呈现和组合。

独立地对这些block - nerf建模允许最大的灵活性，可以扩展到任意大的环境，并提供以分段方式更新或引入新区域的能力，而无需重新训练整个环境，如图1所示。

为了计算目标视图，只渲染block - nerf的一个子集，然后根据它们相对于相机的地理位置进行合成。

为了实现更无缝的合成，我们提出了一种外观匹配技术，通过优化不同block - nerf的外观嵌入，将其带入视觉对齐。