![image](https://github.com/popcornGit/arxiv/assets/48575896/54001597-bb04-4d53-8844-2d0649b9f8f3)

Gst nvstreammux插件从多个输入源形成一批帧。

当将source源连接到nvstreammux(复用器)时，必须使用gst_element_get_request_pad()和pad模板sink_%u从复用器请求一个新的pad。

有关更多信息，请参阅DeepStream应用程序源代码中的link_element_to_streammux_sink_pad()。

该muxer形成一个batch size的帧的批处理缓冲区。(batch size是使用GST对象属性指定的。)

muxer复用器转发来自该source源的帧，作为muxer复用器output batched buffer的一部分。

当muxer返回其输出缓冲区时，帧将返回到源。BK问号

当batch被填充或从提供的streammux配置文件中的总体和流特定的“fps”控制配置键计算出的批处理形成__超时__时，muxer复用器将batch推向下游。

当收集新批处理的第一个缓冲区时，超时开始运行。

batch批量生成的默认最大和最小fps分别为120和5。

muxer的默认批处理使用循环算法从源收集帧。

它尝试从每个源(如果所有源都是活动的并且它们的帧速率都是相同的)中收集每批(批大小/ numsource)帧的平均值。

但是，每个源的数字不同，这取决于源的帧速率。

复用器将NvDsBatchMeta元数据结构附加到输出批处理缓冲区。

这个元数据包含了关于复制到批处理中的帧的信息(例如帧的源ID，输入帧的原始分辨率，输入帧的原始缓冲PTS)。

连接到Sink_N pad的源将在NvDsBatchMeta中具有pad_index N。

该复用器支持在运行时添加和删除源。

当复用器从一个新源接收到一个缓冲区时，它发送一个GST_NVEVENT_PAD_ADDED事件。

当一个复用器接收垫被移除时，复用器发送一个GST_NVEVENT_PAD_DELETED事件。

两个事件都包含正在添加或删除的源的源ID(参见sources/includes/gst-nvevent.h)。

下游元素在接收到这些事件时可以重新配置。

此外，复用器还发送GST_NVEVENT_STREAM_EOS来指示来自源的EOS。

复用器支持计算源帧的NTP时间戳。

它支持两种模式。

在系统时间戳模式下，复用器将当前系统时间附加为NTP时间戳。

在RTCP时间戳模式下，复用器使用RTCP发送端报告(RTCP Sender Report)计算帧在源端产生时的NTP时间戳。

NTP时间戳在NvDsFrameMeta的ntp_timestamp字段中设置。

可以通过设置attach-sys-ts属性来切换该模式。

更多详细信息，请参考DeepStream中的NTP时间戳。

![image](https://github.com/popcornGit/arxiv/assets/48575896/183fab58-c549-4bcc-82b0-8c0f07da859b)

![image](https://github.com/popcornGit/arxiv/assets/48575896/b3563ba1-e6f8-42e4-ae51-778afe80de6b)

默认情况下将使用当前的nvsteammux。

用户将能够通过设置环境变量export USE_NEW_NVSTREAMMUX=yes来使用新的nvstreammux。

新的nvstreammux不再是一个测试版功能。

在即将发布的DeepStream版本中，默认情况下将加载新的nvstreammux。

