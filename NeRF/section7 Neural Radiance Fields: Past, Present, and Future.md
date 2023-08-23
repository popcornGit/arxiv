# Evaluation Metrics

因为，我们正在讨论一个全面的调查论文，这是至关重要的读者了解什么是用于评估辐射场的指标。

本次讨论的主要焦点是PSNR(峰值信噪比)，SSIM(结构相似指数测量)，LPIPS(学习感知图像补丁相似)， 倒角距离

PSNR (Peak Signal to Noise Ratio) 
SSIM(structural similarity index measure) 
LPIPS (Learned Perceptual Image PatchSimilarity)
Chamfer distance

## PSNR (Peak-Signal-to-Noise Ratio)

PSNR(峰值信噪比)是一个广泛用于评估图像质量的指标，特别是在图像压缩和恢复任务中。

它测量最大可能的信号功率(即像素强度的最大值)与破坏噪声的功率之间的比率(即原始图像与重建图像之间的均方误差)。

PSNR值越高，图像质量越好。

然而，PSNR可能是一个很差的感知相似性指标，因为它不能准确地捕捉人类对图像质量的感知。

## SSIM (Structural Similarity Index)

SSIM(结构相似指数)是一种更直观的度量，它根据亮度、对比度和结构对两幅图像进行比较。

它考虑了像素强度、空间依赖性和纹理对比度的变化。

SSIM的取值范围是-1 ~ 1，其中1表示原始图像与重建图像完全匹配。

与PSNR相比，SSIM提供了更好的关于人类感知的图像质量评估。

## LPIPS (Learned Perceptual Image Patch Similarity)

LPIPS(学习感知图像补丁相似性)[258]是一种度量，用于评估渲染视图/姿势与特定观看方向的特定图像的地面真实情况相比的感知相似性。

它是一种感知度量，使用深度学习来衡量两幅图像之间的相似性。

它基于从预训练的卷积神经网络(CNN)中提取的特征，旨在更好地与人类对图像相似性的判断保持一致。

较低的LPIPS值表明被比较的图像之间的感知相似性较高。

LPIPS对小的几何和纹理差异具有更强的鲁棒性，更适合于评估生成模型和图像合成任务。

## NIQE (Natural Image Quality Evaluator)

NIQE(自然图像质量评估器)是评估自然图像质量的一个度量。

它通过将统计数据与大型自然图像数据库的统计数据进行比较来衡量图像的自然程度。

NIQE的公式尚未公开，但它涉及计算各种图像统计数据，如平均值、标准差和高阶矩。

## Chamfer Distance

倒角距离是用于评估两个点集之间相似性的几何度量，通常用于3D重建任务。

它测量一个集合中每个点到另一个集合中最近的点之间的平均距离。

较小的倒角距离值表明点集之间的几何对齐更好。

虽然它不直接用于评估辐射场中的图像质量，但倒角距离可用于评估模型重建的底层3D几何形状的质量。

## KID (Kernel Inception Distance)

KID (Kernel Inception Distance)是一种度量，用于评估生成模型中生成图像的质量和多样性，如VAEs和GANs

KID通过测量这些图像在特征空间中的嵌入之间的距离来比较真实图像和生成图像的特征分布，通常从Inception网络(在大规模图像分类任务上预训练的深度卷积神经网络，如ImageNet)中获得。

KID背后的主要思想是通过比较其特征嵌入的统计量来衡量真实图像和生成图像分布之间的相似性。

这些分布之间的距离使用MMD(最大平均差异)度量和RBF(径向基函数)核来计算。

