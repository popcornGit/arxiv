# Block-NeRF: Scalable Large Scene Neural View Synthesis

## Abstract
我们提出了Block-NeRF，神经辐射的变体可以表示大规模环境。

具体来说，我们演示了在缩放NeRF以渲染跨越多个街区的城市规模场景时，将场景分解为单独训练的NeRF是至关重要的。

这种分解将渲染时间与场景大小分离，使渲染能够扩展到任意大的环境，并允许对环境的每个块进行更新。

我们采用了几项架构更改，使NeRF对不同环境条件下几个月捕获的数据具有健壮性。

我们为每个单独的NeRF添加了appearance embeddings、 learned pose refinement和可控曝光，并引入了在相邻NeRF之间对齐外观的过程，以便它们可以无缝组合。

我们从280万张图像中构建了一个block - nerf网格，以创建迄今为止最大的神经场景表示，能够渲染整个旧金山社区。

