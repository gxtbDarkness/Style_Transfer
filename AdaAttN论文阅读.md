# AdaAttN论文阅读

## 一、标题

​	`AdaAttN: Revisit Attention Mechanism in Arbitrary Neural Style Transfer  ` 

​	即：$$AdaAttN$$：重新审视任意神经风格迁移中的注意力机制。从标题中可以知道，本论文的核心便是风格迁移任务中的注意力机制。

## 二、概要（Abstract）

### 2.1 内容

​	先介绍了当前风格迁移任务的方法：要么专注地将深度风格特征融合到深度内容特征中，而不考虑特征分布，要么根据风格自适应地规范化深度内容特征，以使它们的全局统计数据匹配（`Existing solutions either attentively fuse deep style feature into deep content feature without considering feature distributions, or adaptively normalize deep content feature according to the style such that their global statistics are matched.`）。

​	再指出存在的问题：尽管这些方法有效，但对浅层特征未加探索，并且没有局部考虑特征统计，容易导致带有局部失真的不自然输出（`Although effective, leaving shallow feature unexplored and without locally considering feature statistics, they are prone to unnatural output with unpleasing local distortions. `）。

​	论文提出的解决方案：提出了一个新颖的注意力和规范化模块，名为自适应注意力规范化（`AdaAttN`），以逐点自适应地执行注意力规范化（`To alleviate this problem, in this paper, we propose a novel attention and normalization module, named Adaptive Attention Normalization (AdaAttN), to adaptively perform attentive normalization on per-point basis.`）。

  	方案的具体内容：

- 空间注意力分数是从内容和风格图像的浅层和深层特征中学习得来的（`Specifically, spatial attention score is learnt from both shallow and deep features of content and style images.  `）。
- 然后，通过将风格特征点视为所有风格特征点的注意力加权输出的分布，计算每个点的加权统计数据（`Then perpoint weighted statistics are calculated by regarding a style feature point as a distribution of attention-weighted output of all style feature points.`）。
- 最后，对内容特征进行规范化，以使其展现出与计算得到的每个点加权风格特征统计数据相同的局部特征统计（`Finally, the content feature is normalized so that they demonstrate the same local feature statistics as the calculated per-point weighted style feature statistics.  `）。
- 除此之外，基于`AdaAttN`提出了一种新颖的局部特征损失，用以增强局部视觉质量（`Besides, a novel local feature loss is derived based on AdaAttN to enhance local visual quality.  `）。

  	论文还对`AdaAttN`进行了轻微修改，以便应用于视频风格转移。

### 2.2 相关概念理解

#### 2.2.1 浅层特征与深层特征（`shallow feature, deep feature`）







#### 2.2.2 统计（`statistics`）







#### 2.2.3 注意力机制（`Attention Mechanism`）









#### 2.2.4 规范化（`normalize`）











## 三、介绍（`Introduce`）

### 3.1 内容

​	首段介绍了风格迁移的大致发展过程。风格迁移任务的目标是将风格图像的模式应用到内容图像中，并同时保留内容图像的结构。最为影响深远的工作由`Gatys  `等人完成，他们提出了一种图像优化方法，该方法在预训练深度神经网络的特征空间中迭代地最小化内容损失和风格损失。然而，这种耗时的优化过程促使研究人员探索更有效的方法。`Johnson  `等人提出使用前向网络直接生成渲染图像，实现了实时风格转移。因为学习好的模型只能对一种特定风格生效，这个方法与其相关的工作被分类为`Per-Style-Per-Model   `方法。除此之外，还有`Multiple-Style-Per-Model   `方法和`Arbitrary-Style-Per-Model   `方法。在后一种情况下，一旦模型训练完成，该模型可以接受任何风格图像作为输入，并在单次前向传递中生成风格化结果。

​	第二段介绍风格迁移中存在的问题，即：增加灵活性牺牲了对于任意风格转移网络的局部风格模式建模能力。比如$$AdaIN$$方法，它在特征空间将风格图像的全局均值和方差转移给内容图像，以支持任意输入的风格图像。由于特征的均值和方差是全局计算的，局部细节和逐点模式很大程度上被忽略，因此局部样式化性能大幅下降。在`[5, 22, 15, 24, 10]`中也存在类似的灵活性和能力之间的权衡，其中内容图像的所有局部特征点都通过相同的基于风格图像的变换函数进行处理。为了增强任意风格转移模型对局部特征的认知，最近多项工作（`[28, 6, 43]`）在这一任务中采用了注意力机制。它们的共同直觉是，模型在将内容图像区域进行风格化时应更多关注风格图像中特征相似的区域。这种注意力机制已被证明对于在任意风格转移中生成更多的局部风格细节是有效的。不幸的是，虽然提升了性能，但并未完全解决这个问题，局部失真仍然会发生。

​	第三段解释为什么会有这种困境。论文认为揭示上述给注意力机制带来困境的原因并不是一件很困难的事情。深入研究当前基于注意力的任意风格转移解决方案的细节，很容易发现：                                                                                                                                	              	1）设计的注意力机制通常基于深度卷积神经网络在较高抽象级别上的特征，忽略了低层次的细节；                                                     	2）注意力分数通常用于重新加权风格图像的特征图，然后将重新加权的风格特征简单地融合到内容特征中进行解码。                                       基于深度卷积神经网络特征的注意力策略忽略了浅层网络中图像的低层模式。因此，注意力分数可能很少关注低级纹理，并且受到高级语义的主导。与此同时，就像`SANet`中做的一样，对风格特征进行空间重新加权，然后将重新加权的风格特征与内容特征融合，而不考虑特征分布。

​	第四段提出了论文的解决方案。论文尝试解决这些问题，并在风格模式转移和内容结构保留之间取得更好的平衡。受到以上分析所得的经验教训的启发，论文提出了一个新颖的注意力和规范化模块，名为自适应注意力规范化（$$AdaAttN$$），用于任意风格转移。它可以根据特征分布进行逐点基础的自适应注意力规范化，更详细地说，它从内容和风格图像的浅层和深层特征中学习空间注意力分数。然后，通过将风格特征点视为所有空间特征点的注意力加权输出的分布，计算每个点的加权统计数据。最后，对内容特征进行规范化，使其局部特征统计与每个点加权风格特征的统计数据相同。这种方式下，注意力模块考虑了风格图像和内容图像的浅层和深层卷积神经网络特征。同时，实现了从内容特征到调制风格特征的逐点特征统计对齐。基于$$AdaAttN$$模块，提出了一种新的优化目标，名为局部特征损失，并推导出了一个新的任意图像风格转移流程。论文的贡献主要如下：

- 我们引入了一种新颖的AdaAttN模块，用于任意风格转移。它考虑了浅层和深层特征用于注意力分数的计算，并适当地对内容特征进行规范化，使特征统计与基于注意力加权的风格特征的均值和方差图在每个点上都能很好地对齐。
- 提出了一种新的优化目标，称为局部特征损失。它有助于模型训练，并通过规范生成图像的局部特征来提高任意风格转移的质量。
- 进行了大量实验并与其他最先进的方法进行了比较，以展示提出的方法的有效性。
- 通过引入基于余弦距离的注意力和基于图像相似度的损失，进一步扩展我们的模型用于视频风格转移可以产生稳定且吸引人的结果。

### 3.2 相关概念理解



















## 四、相关工作（`Arbitrary Style Transfer  `）

### 4.1 任意风格迁移（`Arbitrary Style Transfer  `）

















### 4.2































