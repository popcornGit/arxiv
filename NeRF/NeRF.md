# NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis

## 摘要
我们提出了一种方法，通过使用稀疏的输入视图集优化underlying continuous volumetric scene function函数，实现了最先进的结果，用于合成复杂场景的新视图。

我们的算法使用全连接(非卷积)深度网络表示一个场景，其输入是一个连续的5D坐标(空间位置(x, y, z)和观看方向(θ, φ))，其输出是该空间位置的volume density和view-dependent emitted radiance。

我们通过沿着相机光线querying5D坐标来合成视图，并使用经典的体渲染技术将输出的颜色和密度投影到图像中。

因为体素渲染是自然可微的，所以优化我们的表示所需要的唯一输入是一组具有已知相机位姿的图像。

我们描述了如何有效地优化神经辐射场，以渲染具有复杂几何和外观的场景的逼真的新视图，并演示了优于先前在神经渲染和视图合成方面的工作的结果。

视图合成结果最好作为视频观看，因此我们敦促读者查看我们的补充视频，以便进行令人信服的比较。
