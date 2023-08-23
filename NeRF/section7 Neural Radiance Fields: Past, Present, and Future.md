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

