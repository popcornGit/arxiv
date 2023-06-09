![image](https://user-images.githubusercontent.com/48575896/226500717-142d0726-9a5b-4620-a6b4-3a9fb5fec320.png)

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

## Background
我们建立在NeRF[42]和它的扩展mip-NeRF[3]。

在此，我们对这些方法的相关部分进行总结, 详情请参阅论文原文。

### NeRF and mip-NeRF Preliminaries
神经辐射场(NeRF)[42]是一种基于坐标的神经场景表示，它通过可微分渲染损失进行优化，以重现一组来自已知相机姿势的输入图像的外观。

优化后，NeRF模型可以用来渲染以前未见过的视点。

NeRF场景表示是一对多层感知器(mlp)。第一个MLP fσ取三维位置x，输出体积密度σ和特征向量。

该特征向量与2D观看方向d连接，馈送到第二个MLP fc，输出RGB颜色c。

这种架构确保输出颜色可以在从不同角度观察时发生变化，允许NeRF表示反射和光滑材料，但σ表示的底层几何仅是位置的函数。

图像中的每个像素在三维空间中对应一条射线r(t) = o + td。

为了计算r的颜色，NeRF沿射线随机采样距离![image](https://user-images.githubusercontent.com/48575896/226497140-ba58548c-4396-4129-a91a-ccd5614fc3ff.png)，通过点r(ti)和方向d通过它的mlp进行计算![image](https://user-images.githubusercontent.com/48575896/226497182-df89ce72-cea7-4a46-9a7e-1713e9ac22bf.png)

结果输出颜色为:
![image](https://user-images.githubusercontent.com/48575896/226497210-cfe4d2a2-4b76-4b27-9d9c-b9a87fadddb2.png)

完整的NeRF实现迭代地重新采样点ti(通过将权重wi视为概率分布)，以便更好地将样本集中在高密度区域。

为了使NeRF mlp能够表示更高的频率细节[63]，输入x和d分别经过分量正弦位置编码γPE预处理:

![image](https://user-images.githubusercontent.com/48575896/226497592-1691931c-12b8-4979-a256-4abb55be7c9f.png)

其中L是位置编码的层数。

NeRF的MLP fσ以单个3D点作为输入.

然而，这忽略了  相应图像像素的相对足迹  和  沿着射线r的间隔[ti−1,ti]上可能包含着点但就不采用该点，导致在渲染新相机轨迹时产生混叠工件。

MipNeRF[3]通过 沿着射线  使用投影像素足迹  而不是间隔  对锥形截锥进行采样来解决这个问题。

![image](https://user-images.githubusercontent.com/48575896/226497948-fc73dd71-218c-42f1-9055-ed5fec0b78ab.png)

为了将这些截锥送入MLP, mip-NeRF将它们近似为带有参数µi， Σi的高斯分布，并将位置编码γPE替换为其在输入高斯上的期望，称为集成位置编码。

## Method
训练一个单一的NeRF在试图表现像城市这样大的场景时是不适合的。

相反，我们建议将环境拆分为一组block - nerf，这些block - nerf可以在推理期间并行地独立训练和合成。

这种独立性允许使用额外的block - nerf或更新块来扩展环境，而无需重新训练整个环境(参见图1)。

我们动态地选择相关的block - nerf进行渲染，然后在遍历场景时以平滑的方式进行合成。

为了辅助这种合成，我们优化了外观代码以匹配照明条件，并使用基于每个条件计算的插值权重Block-NeRF到新视图的距离。

### Block Size and Placement
单独的block - nerf应该安排在一起以确保对目标环境的完全覆盖。

我们通常在每个十字路口放置一个Block-NeRF，覆盖十字路口本身和任何连接街道的75%，直到它收敛到下一个十字路口
图1)。

这导致连接街道段上任何两个相邻街区之间有50%的重叠，使它们之间的外观更容易对齐。

遵循这个过程意味着块大小是可变的;在必要时，可以引入额外的块作为交叉路口之间的连接器。

通过应用地理过滤器，我们确保每个块的训练数据完全保持在其预期的范围内。

这个过程可以自动化，只依赖于基本的地图数据，如OpenStreetMap[22]。

请注意，只要整个环境至少被其中一个Block-NeRF覆盖，其他的放置启发式也是可能的。

例如，在我们的一些实验中，我们沿着单一的街道段以均匀的距离放置街区，并将街区大小定义为围绕街道的球体Block-NeRF Origin(参见图2)。

![image](https://user-images.githubusercontent.com/48575896/226500776-035a66b3-0c32-4d8d-bfff-fd602a38feac.png)


### Training Individual Block-NeRFs
#### Appearance Embeddings
考虑到我们数据的不同部分可能在不同的环境条件下被捕获，我们遵循NeRF-W[40]并使用生成潜优化[5]来优化周围图像外观嵌入向量，如图3所示。

![image](https://user-images.githubusercontent.com/48575896/226500852-6cccc583-3dd7-46a3-830a-cc4c1f2b5dd3.png)

这允许NeRF解释一些外观变化的条件，如变化的天气和照明。

我们还可以操作这些外观嵌入，在训练数据中观察到的不同条件之间进行插值(例如多云与晴朗的天空，或白天与夜晚)。

在图4中可以看到具有不同外观的渲染示例。

![image](https://user-images.githubusercontent.com/48575896/226501150-41f11c29-b7b4-4e72-b0d7-277e9bb43ddb.png)

在§4.3.3中，我们在这些嵌入上使用测试时间优化来匹配相邻block - nerf的外观，这在组合多个渲染时很重要。

#### Learned Pose Refinement
虽然我们假设提供了相机姿态，但我们发现学习正则化姿态偏移量对进一步对齐是有利的。

姿态细化已经在之前的研究中进行了探索基于NeRF的模型[34,59,66,70]。

这些偏移量是每个驾驶段学习的，包括平移和旋转组件。

我们与NeRF本身一起优化这些偏移量，在训练的早期阶段显著地正则化偏移量，以允许网络在修改姿势之前首先学习一个粗略的结构。

####  Exposure Input
训练图像可能会在大范围的曝光水平下被捕获，如果不加以解释，这可能会影响NeRF训练。

我们发现，将相机曝光信息输入到模型的外观预测部分可以使NeRF补偿视觉差异(参见图3)。

具体来说，曝光信息被处理为γPE(快门速度×模拟增益/t)，其中γPE是4级正弦位置编码，t是缩放因子(我们在实践中使用1000)。

在图5中可以找到不同的习得暴露的例子。

![image](https://user-images.githubusercontent.com/48575896/226502568-b4367c1e-9289-4954-908c-a757719465d6.png)

#### Transient Objects
虽然我们的方法使用外观嵌入来解释外观的变化，但我们假设场景几何形状在整个训练数据中是一致的。

任何可移动的物体(如汽车、行人)通常都违反了这一假设。

因此，我们使用语义分割模型[10]来生成常见可移动物体的掩码，并在训练过程中忽略掩码区域。

虽然这不能解释环境中其他静态部分的变化，例如结构，但它可以容纳最常见的几何不一致类型。

#### Visibility Prediction
当合并多个block -NeRF时，在训练过程中了解特定的空间区域是否对给定的NeRF可见是很有用的。

我们用一个额外的小MLP fv扩展了我们的模型，它被训练来学习采样点可见性的近似值(见图3)。

对于沿着训练射线的每个样本，fv取位置和视图方向，并回归相应的点的透射率(公式2中的Ti)。

该模型与提供监督的fσ一起训练。

透光率表示一个点从一个特定的输入相机可见的程度:自由空间中的点或第一个相交物体表面上的点的透光率将接近1，第一个可见物体内部或后面的点的透光率将接近0。

如果从某些视点看到一个点，而不是从其他视点看到，则回归的透过率值将是所有训练相机的平均值，并且位于0到1之间，表明该点被部分观测到。

我们的能见度预测与Srinivasan等人[58]提出的能见度场相似。

然而，他们使用MLP来预测环境照明的可视性，以恢复一个relightable NeRF模型，而我们则预测训练光线的可视性。

能见度网络很小，可以独立于颜色和密度网络运行。

当合并多个NeRF时，这被证明是有用的，因为它可以帮助确定特定的NeRF是否可能为给定的位置产生有意义的输出，如§4.3.1所述。

能见度预测还可以用于确定两个nerf之间进行外观匹配的位置，详见§4.3.3。

### Merging Multiple Block-NeRFs
#### Block-NeRF Selection
环境可以由任意数量的block - nerf组成。

为了提高效率，我们使用了两种过滤机制来只呈现给定目标视点的相关块。

我们只考虑在目标视点设定半径内的block - nerf。

此外，对于每个候选对象，我们计算相关的可见性。

如果平均可见性低于阈值，则丢弃Block-NeRF。

图2提供了一个可见性筛选的示例。

可见性可以快速计算，因为它的网络独立于颜色网络，并且不需要以目标图像分辨率渲染。

过滤后，通常有一到三个block - nerf需要合并。

#### Block-NeRF Compositing
我们渲染来自每个过滤后的blocknerf的彩色图像，并在它们之间使用相机原点c和每个Block-NeRF的中心xi之间的逆距离加权进行插值。

具体来说，我们计算各自的权重为![image](https://user-images.githubusercontent.com/48575896/226504998-5b8cb93c-0507-4906-b810-c2c444d93a5f.png)，其中p会影响Block-NeRF渲染之间的混合速率。

插值是在二维图像空间中完成的，并在block - nerf之间产生平滑过渡。

我们还在§5.4中探索了其他插值方法。

#### Appearance Matching
在训练Block-NeRF之后，我们学习的模型的外观可以由 appearance latent code控制。

这些代码在训练过程中是随机初始化的，因此当输入不同的block - nerf时，相同的代码通常会导致不同的外观。

这在合成时是不可取的，因为它可能导致视图之间的不一致。

给定一个block - nerf中的目标外观，我们的目标是匹配它在其余块中的外观。

为了实现这一点，我们首先在相邻的block - nerf对之间选择一个3D匹配位置。

对于两个block - nerf，该位置的能见度预测都应该很高。

给定匹配位置，我们冻结目标的Block-NeRF的网络权重, 并仅仅优化目标块的appearance code，以减少各自区域渲染之间的L2损失。

这种优化是快速的，收敛于100次迭代以内。

虽然不一定会产生完美的对齐，但这个过程会对齐场景的大多数全局和低频属性，例如一天中的时间、颜色平衡和天气，这是成功合成的先决条件。

图6显示了一个优化示例，其中外观匹配将白天场景转换为夜间场景，以匹配相邻的Block-NeRF。

![image](https://user-images.githubusercontent.com/48575896/226506953-3f1875e7-651c-4553-99fe-fe6b140bfb83.png)

优化后的外观在场景中迭代传播。从一个根Block-NeRF开始，我们优化邻近的块的外观，并从那里继续这个过程。

如果一个目标Block-NeRF周围的多个块已经被优化了，我们在计算损失时考虑每个块。

## Limitations and Future Work
该方法通过使用分割算法在训练过程中通过屏蔽将瞬态对象过滤掉.

如果对象没有被正确地屏蔽，它们可能会在最终的渲染中产生工件。

例如，即使汽车本身被正确地移除，汽车的阴影通常仍然存在。

植被也打破了这一假设，因为树叶随季节变化，随风移动;这导致了树木和植物的模糊表现。

类似地，训练数据中的时间不一致，例如建筑工作，不会自动处理，需要手动重新训练受影响的块。

此外，目前无法渲染包含动态对象的场景限制了Block-NeRF对机器人技术中的闭环模拟任务的适用性。

在未来，这些问题可以通过在优化[40]过程中学习瞬态对象或直接建模动态对象来解决[44,67]。

特别是，场景可以由环境的多个e Block-NeRFs 和 单个可控对象nerf组成。

使用分割掩码或bounding框可以促进分离。

在我们的模型中，场景中远处物体的采样密度与附近物体不同，这导致重建更加模糊。

这是采样无界体积表示的一个问题。

在nerf++[74]和并发mip - nerf360[4]中提出的技术可能被用于产生更清晰的远处物体渲染。

在许多应用程序中，实时渲染是关键，但是nerf渲染的计算成本很高(每张图像最多需要数秒)。

一些NeRF缓存技术[20,25,72]或稀疏体素网格[36]可用于实现实时块NeRF渲染。类似地，多个并发工作已经演示了将NeRF风格表示的训练速度提高多个数量级的技术[43,60,71]。

## Conclusion
在本文中，我们提出了Block-NeRF，一种使用nerf重建任意大环境的方法。

我们通过从2.8万张图像中构建旧金山的整个社区，形成迄今为止最大的神经场景表示，证明了该方法的有效性。

我们通过将我们的表示分成多个可以独立优化的块来实现这个规模。

在这样的规模下，收集到的数据必然具有瞬态对象和外观变化，我们通过修改底层NeRF体系结构来解释这一点。

我们希望这可以启发未来使用现代神经渲染方法进行大规模场景重建的工作。
