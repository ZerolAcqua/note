---
---

# Sample4Geo: Hard Negative Sampling for Cross-View Geo-Localisation

!!! info "论文链接"
	原文：Fabian Deuser, Konrad Habel, Norbert Oswald. [Sample4Geo: Hard Negative Sampling For Cross-View Geo-Localisation](https://doi.org/10.1109/ICCV51070.2023.01545)

!!! warning "注意"
    机翻后整理


## 摘要
交叉视图地理定位仍然是一项具有挑战性的任务，需要额外的模块、特定的预处理或缩放策略来确定图像的准确位置。由于不同的视图具有不同的几何形状，因此极坐标变换等预处理有助于合并它们。然而，这会导致图像失真，然后必须进行校正。向训练批次添加硬负值可以提高整体性能，但由于地理定位中使用了默认的损失函数，因此很难包含它们。在这项工作中，我们提出了一个简化但有效的架构，该架构基于对比学习和对称的 InfoNCE 损失，其性能优于当前最先进的结果。我们的框架由一个狭窄的训练管道组成，无需使用聚合模块，避免了进一步的预处理步骤，甚至提高了模型泛化到未知区域的能力。我们介绍了两种类型的硬否定抽样策略。第一种明确地利用了地理上相邻的位置，以提供一个很好的起点。第二种利用图像嵌入之间的视觉相似性来挖掘硬的负样本。我们的工作在 CVUSA、CVACT、University1652 和 VIGOR 等常见的交叉视图数据集上显示出出色的性能。跨区域设置和相同区域设置之间的比较证明了我们的模型具有良好的泛化能力。

## 1 介绍

从没有元数据的图像中确定地理位置是计算机视觉社区中尚未解决的难题之一。解决这个问题可以在农业和汽车等领域有所帮助。例如，农业中用于喷洒肥料的机器人代理需要高精度的定位。这可以通过实时运动学 （RTK） GPS 来实现，但这些传感器价格昂贵，而且短时间信号中断可能会阻碍工作流程。因此，在这些高度重复的环境中，基于航空图像的定位可以进一步增强定位【4】。城市面临的另一个挑战是所谓的城市峡谷效应，它会阻挡GPS等信号或降低其准确性。特别是在大城市，由于高层建筑，GPS 信号很嘈杂。Brosh 等人对纽约市交通中 250,000 小时驾驶的评估显示，40% 的 GPS 信号存在 10 米的误差。因此，提出了一种基于图像检索的计算机视觉解决方案【2】来校正这些信号。

虽然经典方法试图通过视觉线索来做到这一点，如太阳位置和由此产生的阴影【7, 19, 17】或天气【16, 15】，但目前的方法越来越关注基于深度学习的图像检索【1, 20, 21】。在交叉视图图像检索中，必须匹配图像的不同视图，例如地面图像和卫星图像，以确定搜索的位置。将地面图像查询与具有已知地理位置的卫星图像数据库进行比较。

以前的方法大多使用基于 CNN 的方法【38, 13, 3, 30, 36, 35, 28, 29】，而目前的研究主要集中在 Transformer 或 MLP Mixer 架构上，作为地理定位的骨干【40, 43, 46, 48】。在后一种情况下，只有当两个特定于视图的编码器之间的权重不共享时，才能实现合理的性能，从而导致模型变大。此外，极坐标变换通常用于弥合视图之间的几何差距【28】。三重态损失作为交叉视角地理定位中的标准损失，每批仅使用一个负例，并且在使用硬负值时容易出现模型崩溃【39】。

在本文中，我们提出了一个权重共享的孪生 CNN，它学习基于 InfoNCE 损失的与类无关的嵌入【25, 11, 5】，并展示了 CNN 架构对我们方法的优势。我们使用多模态预训练的技术【26】对称地计算损失，以进一步帮助理解视点中的不同领域。在深度度量学习中，硬负样本，即模型难以区分的样本和正样本，是实现卓越性能的关键因素，因此我们提出了两种采样方法。在早期，我们利用GPS坐标来对比近在咫尺的地理邻居，作为我们第二次采样的一个很好的初始化。在基于相似性度量（如余弦相似度）的后期时期中，我们收集视觉上相似的样本进行批量构建，以专注于硬负值示例。总结一下我们在这项工作中的贡献：
- 我们展示了我们的对比训练在对称 InfoNCE 损失方面的卓越性能，同时对两个视图使用单个编码器。
- 在训练开始时，我们建议将 GPS 采样作为一种针对硬负片的特定任务采样技术。
- 我们提出了动态相似性采样 （DSS） 技术，以在训练过程中根据街道和卫星视图之间的余弦相似性选择硬负值。
- 我们的框架由一个简单的训练管道组成，无需特殊的聚合模块或复杂的预处理步骤，同时在性能和泛化能力方面优于当前的方法。

## 2 相关工作
Workman 等【38】的首批研究之一表明，CNN 提取的特征远远优于手工制作的特征。在他们的工作中，他们还介绍了 CVUSA，这是当今地理定位的主要基准之一。为了将他们的双流 CNN 训练为孪生网络【6, 31】，使用了一个简单的 L2 目标来减少视图特定特征之间的距离。在接下来的研究中，Zhai 等【41】在鸟瞰图中使用了嘈杂的语义标签，并最小化了这些标签之间的距离，并对街景进行了学习变换。特别是，这应该更好地反映视图中的常见语义布局。Vo 等【34】引入了软边际三元组损失作为交叉视角地理定位的标准。三元组损失减小了正样本到锚点的距离，并增加了到负样本的距离。胡等【13】的后续工作在其架构中加入了 NetVLAD 层【1】，以进一步增强全局描述生成。输出的特征图基于差分聚类进行聚合，而不是简单的平均或最大池化。由于软边际三元组损失的收敛速度缓慢，他们提出了一种加权软边距排序损失法，其中正样本和负样本之间的距离被额外缩放。

最初研究还关注图像中的方向，如在 CVUSA【38】等数据集中，对齐是先验已知的，并且可以作为附加特征，正如 Liu 等人【21】所展示的那样。为了帮助网络更好地利用方向，扩大了输入频道的数量，并且从上到下和从左到右对街景和卫星视图都使用了颜色编码的方向。此外，他们还引入了 CVACT 作为数据集。

在随后的几篇出版物【34, 3, 29, 14】中，方向进一步用于创建辅助目标。街景图像作为增强而移动，并且必须预测移动的程度。

由于卫星图像和街景的几何形状不同，Shi 等【28】对卫星图像进行了极性变换。得到的拉伸图像类似于街景图像的域，这在今天仍然是一种常见的预处理技术。此外，他们还使用空间感知特征聚合 （SAFA） 模块扩展了他们的网络，以可学习的方式从特征图中汇集重要特征。

极性变换的缺点是畸变会向图像注入干扰。Toker 等【32】开发了一种方法，该方法学会了在 GAN【10】的帮助下消除这些干扰。街景图像被用作 GAN 判别器的真值。在推理时，不再使用生成器和鉴别器，仅使用潜在表示来查找与街景的相似性。

Yang 等【40】介绍了将 ResNet 骨干网与 Transformer 结合使用来完成该任务。CNN 输出的特征图采用位置编码，然后输入到 Transformer 中。

由于前面提到的基准测试正在慢慢饱和，因此 Zhu 等【47】创建了一个更具挑战性的数据集，称为 VIGOR。虽然 CVUSA、CVACT 和 VIGOR 仅使用街景来拍摄卫星图像，但 Zheng 等人用无人机图像而不是街景创建了 University-1652【44】。

以前的方法只限于一个无法纠正的预测步骤，因此 TransGeo【46】和 SIRNet【24】强调了额外的细化。在 SIRNet 中，额外的优化模块被添加到 CNN 主干网中。基于 Softmax 的决策过程使用可变数量的模块，最多四个。使用 TransGeo 时，可以使用其 Transformer 架构中的注意力贴图执行额外的缩放步骤。这导致以更高的分辨率观看较小的对象。为了提高其方法的通用性，TransGeo 还使用自适应锐度感知最小化（ASAM）【18】。ASAM 大大减慢了训练过程，因为对于所有训练数据，需要通过网络进行额外的前向传递。由于几何视角会因不同的视角而发生强烈偏移，因此 GeoDTR【42】尝试使用基于 Transformer 的提取器提取其他几何属性。首先，特征是使用 CNN 视图独立生成的。但是，由于缺少这些几何特征的真值标签，因此使用额外的反事实损失来对比 CNN 的输出特征和几何提取器的输出特征，以确保相似性。然后计算依赖于视图的几何提取器的特征之间的三元组损失。Zhu 等【48】提出的另一种架构，即 SAIG-D，利用了MLPMixer【33】。他们用卷积词干替换了补丁词干，以支持局部特征学习。此外，他们还提出了一种自己的特征聚合模块，该模块基于两个全连接层，具有 GELU 激活函数和跟随向下投影。与 TransGeo 类似，他们利用锐度感知最小化（SAM）【9】来进一步平滑损失表面。


## 3 方法
用于跨视图地理定位的深度度量学习默认使用三元组损失【27】，并带有各种扩展，例如软边距三元组损失【34】或加权软边距三元损失【13】。三元组损失旨在减小正样本和锚点之间的距离，同时增加负样本和锚点之间的距离。要使用三元组损失，必须事先对合适的三元组进行采样。根据这个公式，只有一个正和锚总是与一个负示例形成对比。Arandjelovic 等【1】已经证明了使用两个阴性作为四倍损失而不是三倍损失的优势。如果没有上述扩展，三元组损失中的硬负值会导致模型崩溃【39】。考虑到这一点，我们设计了我们的框架来利用多个硬否定因素。
### 3.1 对称 InfoNCE 损失

对比学习的另一种方法利用了批处理中所有可用的负数，而不是三元组损失，是所谓的 InfoNCE【25, 26】或 NTXent【11, 5】损失:
$$\mathcal{L}(q,R)_{\mathrm{InfoNCE}}=-\log\frac{\exp(q\cdot r_+/\tau))}{\sum_{i=0}^R\exp(q\cdot r_i/\tau))}$$
$q$ 表示编码的街景，即所谓的查询，$R$ 是一组称为引用的编码卫星图像。只有一个正 $r_i$ ，即 $r_+$ 与 $q$ 匹配。InfoNCE 损失使用点积来计算查询图像和参考图像之间的相似度，当查询和正匹配相似时，损失值较低，当负匹配项与 $q$ 不同时，损失值较高。作为视图之间相似性的损失函数，计算了交叉熵。温度参数 $\tau$ 是一个超参数，可以学习【26】或设置为静态值。到目前为止，InfoNCE 损失主要以非对称方式用于图像的无监督表示学习【11, 5】。对称公式在多模态预训练中被证明是有用的【26】，以弥合模态之间的差距。因此，我们以相同的对称方式利用这种损失函数来利用两个方向的信息流：卫星视图到街视图，反之亦然。在 InfoNCE 损失中，正样本总是与 $N-1$ 负样本形成对比，其中 $N$ 表示批量大小，因此一次分隔多个样本。但是，在有多个阳性示例的情况下，例如 University-1652，这需要一个自定义采样器，以防止同一真值标签在一批中出现多个阳性。我们提供了对称性在 InfoNCE 损失中的重要性的消融研究，并与我们的补充材料中的三重态损失进行了比较。

### 3.2 模型架构

我们在本文中的主要目标之一是使用一种现成的架构，该架构不需要对图像进行任何特殊调整，以便以有意义的方式解决地理定位问题。由于最近的出版物越来越多地使用 Vision-Transformer【46,40,43】或MLP混频器架构【48】，因此我们比较了两种架构。为此，我们使用 ConvNeXt 【22】作为最先进的 CNN 和 Vision Transformer 【8】。我们根据第 4.3.1 节中的结果决定支持 ConvNeXt。与之前的方法一致，我们使用孪生网络，如图 2 所示。我们使用均值池化的特征向量，而没有任何注意力池化或模块进行细化。ConvNeXt 是 ResNet【2】 的现代化变体，并使用了许多改进，例如修补的词干、更高的内核大小、GELU 激活函数、LayerNorm 和深度卷积。虽然由于视图域的不同，目前的工作没有使用权重共享，但我们强调使用权重共享的 ConvNeXt 作为两个视图的单一编码器。

### 3.3 基于 GPS 的近邻采样

在大多数情况下，由于模型未拟合到训练数据的域，因此无法在训练之前选择困难示例。为了绕过这个缺点，我们利用了跨视角地理定位的地理性质。在 VIGOR 数据集中，来自同一区域甚至同一街道的图像共享共同的属性，如植被、路标或房屋类型。由于这些是将乡村景观与城市景观进行对比的简单特征，因此它们对目标函数没有太大贡献。因此，我们提出了一种简单的基于 GPS 的采样策略，用于在训练开始之前进行负挖掘初始化。对于 CVUSA 和 VIGOR，数据集中存在 GPS 位置，并且根据正弦距离选择近邻。在 CVACT 数据集中，位置位于 UTM 坐标系中，这就是我们使用欧几里得距离来确定邻居的原因。GPS 坐标仅在早期时期的采样策略训练期间使用。对于 University-1652 数据集，无法进行此初始化，因此使用了简单的随机排序。

### 3.4 动态相似采样

经过一定的时期后，基于 GPS 的采样被我们的动态相似性采样 （DSS） 所取代。在对训练数据进行一个完整的推理周期中，所有样本之间的视觉距离是使用余弦相似度计算的。为了对未来的批次进行采样，将选择每个查询图像的前 $K$ 个最近邻。我们将 $K$ 设置为小于或等于批量大小，以在一个批次中有多个区域或城市。根据它们的相似性对这些 $K$ 个邻居进行排序，然后为该批次选择 $k/2$ 个最近的样本，其余 $k$ 个样本是从 $K$ 中的剩余样本中随机选择的。随机选择过程确保了硬负值的足够多样性，因为我们只计算每 $e$ 个时期的新距离以缩短训练过程。我们按如下方式设置超参数：$k = 64$、$K = 128$ 和 $e = 4$。在我们的补充材料中可以找到对 $k$ 最佳选择的消融。在我们将 $k$ 个样本添加到批次中之前，先进行查找以避免在一个纪元内出现重复输入。对于 $k$ 的不同设置，即使 $k = K$，我们也没有观察到【39】中描述的折叠模型问题。


## 4 评估
### 4.1 实验和结果
### 4.2 实现细节
### 4.3. 消融研究

在我们的消融研究中，我们深入研究了这项工作中所做的设计选择，例如模型架构或采样的有效性。为了提供额外的见解，我们提取了卫星的激活热图和模型的街景图像，以可视化两个视图中的重要区域。

#### 4.3.1 架构评估
长期以来，在交叉视地理定位的研究社区中，CNN 是学习有用表示的最重要组成部分【28,13,41,32,42】。最近的出版物探讨了其他架构，如 transformers【46,40,43】或 MLP 混合器【48】。为了确定我们方法的正确选择，我们比较了两种标准架构：表 4 中的视觉转换器（ViT）【8】和当前最先进的 CNN【22】。

CNN 的优点是它可以处理不同的输入图像大小。视觉 Transformer 由于其固定大小的位置编码，因此无法处理多种输入大小。Transformer 架构的另一个限制是其可扩展性，即由于注意力机制导致的二次内存消耗。以前的方法建议对卫星和街景图像使用单独的编码器。因此，我们还测试了没有权重共享的 ConvNeXt，但当对两个视图使用相同的编码器时，我们获得了更好的结果。在表 4 中，我们比较了具有两个独立编码器（用于卫星和街景）的模型，我们的方法只需要一个编码器即可用于两种视图。

由于 SAIG-D 是迄今为止在 VIGOR、CVUSA 和 CVACT 等数据集上的最佳方法之一，其参数数量要少得多，因此有必要进行比较。问题是我们的模型在使用相似数量的参数时表现如何。如表 4 所示，我们的工作结果非常一致，参数数量越多，对性能的影响越大。我们不是为每个视图提供两个单独的主干，而是使用单个编码器，并表明它的性能优于两个单独的编码器，即使在具有相似模型大小的设置中也是如此。

#### 4.3.2 抽样策略的有效性

在没有任何抽样策略的情况下，我们在利用 InfoNCE 损失使用对比训练时仍然可以获得有竞争力的结果。但是，一个好的采样策略可以大大提高性能，如表 5 所示。我们还测试了 CVUSA 和 CVACT 的抽样策略，结果包含在我们的补充材料中。

由于 VIGOR SAME 分割中存在地理邻居，因此在整个训练过程中，该模型都受益于这种方法。但是，由于所有四个城市的位置都可能出现在 SAME 分割中，因此观察 GPS 采样是否对交叉区域分割有影响也很有趣。这些附近的位置不在测试集中，并且在跨区域分割上借助 GPS 采样提高性能将表明这种采样的普遍性。正如我们成功展示的那样，在我们的同区域和跨区域实验中，结果是一致的。即使在训练期间仅使用 GPS 采样，我们也能获得与视觉相似性采样几乎相同的 $R$@$1$。当使用基于 GPS 的采样而不使用 DSS 时，可以假设，短地理距离内的区域也非常相似，并且根据视觉特征更难区分。与同一区域设置相比，两种策略组合在跨区域设置中的性能更大。基于这一分析，我们决定对报告的 CVUSA 和 CVACT 结果使用两种抽样策略。

#### 4.3.3 泛化能力

关于不同方法的一个非常有趣的问题是泛化能力，即模型在一个区域或场景上训练然后用于另一个区域的能力。正如我们在表 3 中已经特别显示的那样，我们的方法很好地推广到未知城市，但还没有达到与同一区域设置几乎相同的性能。对于我们的消融，我们还评估了在 CVUSA 和 CVACT 上训练的模型，以显示这些数据集之间的转移。从表 6 可以看出，我们的方法得分很高，特别是与没有极变换的方法相比。在我们的补充材料中，我们提供了模型在此设置下的行为的进一步可视化。这表明，被错误识别的卫星图像在道路走向方面通常非常相似。该模型偏向于图像中的道路路线，并将其编码到特征向量中。这就是为什么以前的方法受益于极变换的原因之一，极变换导致街景和变形的卫星影像更好地对齐。

#### 4.3.4 可视化 
人类对交叉视角地理定位的推理通常基于强大的特征，例如道路的走向或独特的建筑物。因此，了解哪些区域对模型很重要特别有趣。在我们的补充材料中，我们介绍了 CVUSA 的多个示例。这些示例表明，在 CVUSA 中，街道是图像中最重要的特征之一。对于VIGOR数据集，情况并非如此。在图 3 中，可以观察到 semi-positive hit （半正命中？），周围景观的激活对于预测更为重要。但是，VIGOR 数据集包含每个位置的多个 semi-positives （半正定？）。由于两个卫星图像中都存在多个重要特征，例如图 3 中的树木，因此很难进行明确的预测。
## 5 结论

在我们的工作中，我们提出了一种简单而有效的方法来解决地理定位任务。我们提出的模型由一个基于现代CNN的用于卫星和地面视图的单一图像编码器组成。这种轻量级方法通过使用 InfoNCE 损失作为训练目标来利用对比学习。我们进一步证明，有效的抽样策略可以在 VIGOR、CVUSA、CVACT 和 University-1652 数据集上获得更好的结果。与其他最先进的方法相比，我们的模型不需要复杂的预处理步骤或任何特定于任务的聚合模块，也不需要使用像SAM或ASAM这样的损失表面平滑来在跨区域设置中实现出色的泛化性能。
## 6 讨论 
在我们的工作过程中，我们发现了几个问题以及未来的挑战。虽然 CVUSA、CVACT 和 VIGOR 等传统数据集仅包含 360◦ 街景图像，但 University-1652 是为数不多的包含无人机图像有限视野的数据集之一。但是，所有图像都是在建筑物附近拍摄的。在这两种情况下，这都极大地简化了地理定位任务，因为必须匹配非定向绑定的特征。在CVUSA和CVACT上表现出色的另一个原因是街景图像的位置与卫星图像的中心完美对齐。这在 VIGOR 中是固定的，但在这里，基准也逐渐饱和。为了提高适用性，需要额外的数据集，这些数据集的中心位置明显未对齐，卫星和街景图像之间的地理方向未知。此外，FoV 应限制在 120 度以下，因为这反映了当前标准智能手机的 FoV。前述数据集的另一个共同特征是对城市环境的关注。CVUSA在这里提供了更广泛的范围，但地面视图图像显然都是在街上拍摄的。这并不能反映野外场景的多样性，正如我们的补充材料中所示，我们的方法大部分时间都集中在街道和十字路口以实现匹配，尤其是在 CVUSA 数据集上。未来的数据集还应包括并非完全取自道路的地面景观，以增加多样性、实用性和实用性。

## 参考文献

??? info "References"

	[1] Relja Arandjelovic, Petr Gronat, Akihiko Torii, Tomas Pajdla, and Josef Sivic. Netvlad: Cnn architecture for weakly supervised place recognition. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 5297–5307, 2016.

	[2] Eli Brosh, Matan Friedmann, Ilan Kadar, Lev Yitzhak Lavy, Elad Levi, Shmuel Rippa, Yair Lempert, Bruno Fernandez-Ruiz, Roei Herzig, and Trevor Darrell. Accurate visual localization for automotive applications. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops, pages 0–0, 2019.

	[3] Sudong Cai, Yulan Guo, Salman Khan, Jiwei Hu, and Gongjian Wen. Ground-to-aerial image geo-localization with a hard exemplar reweighting triplet loss. In 2019 IEEE/CVF International Conference on Computer Vision (ICCV), pages 8390–8399, 2019.

	[4] Nived Chebrolu, Philipp Lottes, Thomas Läbe, and Cyrill Stachniss. Robot localization based on aerial images for precision agriculture tasks in crop fields. In 2019 International Conference on Robotics and Automation (ICRA), pages 1787–1793, 2019.

	[5] Ting Chen, Simon Kornblith, Mohammad Norouzi, and Geoffrey Hinton. A simple framework for contrastive learning of visual representations. In International conference on machine learning, pages 1597–1607. PMLR, 2020.

	[6] S. Chopra, R. Hadsell, and Y. LeCun. Learning a similarity metric discriminatively, with application to face verification. In 2005 IEEE Computer Society Conference on Computer Vision and Pattern Recognition (CVPR’05), volume 1, pages 539–546 vol. 1, 2005.

	[7] F. Cozman and E. Krotkov. Robot localization using a computer vision sextant. In Proceedings of 1995 IEEE International Conference on Robotics and Automation, volume 1, pages 106–111 vol.1, 1995.

	[8] Alexey Dosovitskiy, Lucas Beyer, Alexander Kolesnikov, Dirk Weissenborn, Xiaohua Zhai, Thomas Unterthiner, Mostafa Dehghani, Matthias Minderer, Georg Heigold, Sylvain Gelly, et al. An image is worth 16x16 words: Transformers for image recognition at scale. arXiv preprint arXiv:2010.11929, 2020.

	[9] Pierre Foret, Ariel Kleiner, Hossein Mobahi, and Behnam Neyshabur. Sharpness-aware minimization for efficiently improving generalization. arXiv preprint arXiv:2010.01412, 2020.

	[10] Ian Goodfellow, Jean Pouget-Abadie, Mehdi Mirza, Bing Xu, David Warde-Farley, Sherjil Ozair, Aaron Courville, and Yoshua Bengio. Generative adversarial networks. Communications of the ACM, 63(11):139–144, 2020.

	[11] Kaiming He, Haoqi Fan, Yuxin Wu, Saining Xie, and Ross Girshick. Momentum contrast for unsupervised visual representation learning. In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition, pages 9729–9738, 2020.

	[12] Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun. Deep residual learning for image recognition. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 770–778, 2016.

	[13] Sixing Hu, Mengdan Feng, Rang M. H. Nguyen, and Gim Hee Lee. Cvm-net: Cross-view matching network for image-based ground-to-aerial geo-localization. In 2018 IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 7258–7267, 2018.

	[14] Wenmiao Hu, Yichen Zhang, Yuxuan Liang, Yifang Yin, Andrei Georgescu, An Tran, Hannes Kruppa, See-Kiong Ng, and Roger Zimmermann. Beyond geo-localization: Fine-grained orientation of street-view images by cross-view matching with satellite imagery. In Proceedings of the 30th ACM International Conference on Multimedia, MM ’22, page 6155–6164, New York, NY, USA, 2022. Association for Computing Machinery.

	[15] Nathan Jacobs, Kylia Miskell, and Robert Pless. Webcam geo-localization using aggregate light levels. In 2011 IEEE Workshop on Applications of Computer Vision (WACV), pages 132–138, 2011.

	[16] Nathan Jacobs, Nathaniel Roman, and Robert Pless. Toward fully automatic geo-location and geo-orientation of static outdoor cameras. In 2008 IEEE Workshop on Applications of Computer Vision, pages 1–6, 2008.

	[17] Imran N. Junejo and Hassan Foroosh. GPS coordinates estimation and camera calibration from solar shadows. Computer Vision and Image Understanding, 114(9):991–1003, 2010.

	[18] Jungmin Kwon, Jeongseop Kim, Hyunseo Park, and In Kwon Choi. Asam: Adaptive sharpness-aware minimization for scale-invariant learning of deep neural networks. In International Conference on Machine Learning, pages 5905–5914. PMLR, 2021.

	[19] Jean-François Lalonde, Srinivasa Narasimhan, and Alexei Efros. What do the sun and the sky tell us about the camera? International Journal of Computer Vision, 88:24–51, 05 2010.

	[20] Tsung-Yi Lin, Yin Cui, Serge Belongie, and James Hays. Learning deep representations for ground-to-aerial geo-localization. In 2015 IEEE Conference on Computer Vision and Pattern Recognition (CVPR), pages 5007–5015, 2015.

	[21] Liu Liu and Hongdong Li. Lending orientation to neural networks for cross-view geo-localization. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 5624–5633, 2019.

	[22] Zhuang Liu, Hanzi Mao, Chao-Yuan Wu, Christoph Feichtenhofer, Trevor Darrell, and Saining Xie. A convnet for the 2020s. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 11976–11986, 2022.

	[23] Ilya Loshchilov and Frank Hutter. Decoupled weight decay regularization. arXiv preprint arXiv:1711.05101, 2017.

	[24] Xiufan Lu, Siqi Luo, and Yingying Zhu. It’s okay to be wrong: Cross-view geo-localization with step-adaptive iterative refinement. IEEE Transactions on Geoscience and Remote Sensing, 60:1–13, 2022.

	[25] Aaron van den Oord, Yazhe Li, and Oriol Vinyals. Representation learning with contrastive predictive coding. arXiv preprint arXiv:1807.03748, 2018.

	[26] Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh, Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark, et al. Learning transferable visual models from natural language supervision. In International conference on machine learning, pages 8748–8763. PMLR, 2021.

	[27] Florian Schroff, Dmitry Kalenichenko, and James Philbin. Facenet: A unified embedding for face recognition and clustering. In Proceedings of the IEEE conference on computer vision and pattern recognition, pages 815–823, 2015.

	[28] Yujiao Shi, Liu Liu, Xin Yu, and Hongdong Li. SpatialAware Feature Aggregation for Cross-View Image Based Geo-Localization. Curran Associates Inc., Red Hook, NY, USA, 2019.

	[29] Yujiao Shi, Xin Yu, Dylan Campbell, and Hongdong Li. Where am i looking at? joint location and orientation estimation by cross-view matching. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 4064–4072, 2020.

	[30] Yujiao Shi, Xin Yu, Liu Liu, Tong Zhang, and Hongdong Li. Optimal feature transport for cross-view image geolocalization. In Proceedings of the AAAI Conference on Artificial Intelligence, volume 34, pages 11990–11997, 2020.

	[31] Yaniv Taigman, Ming Yang, Marc’Aurelio Ranzato, and Lior Wolf. Deepface: Closing the gap to human-level performance in face verification. In 2014 IEEE Conference on Computer Vision and Pattern Recognition, pages 1701–1708, 2014.

	[32] Aysim Toker, Qunjie Zhou, Maxim Maximov, and Laura Leal-Taixé. Coming down to earth: Satellite-to-street view synthesis for geo-localization. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 6488–6497, 2021.

	[33] Ilya O Tolstikhin, Neil Houlsby, Alexander Kolesnikov, Lucas Beyer, Xiaohua Zhai, Thomas Unterthiner, Jessica Yung, Andreas Steiner, Daniel Keysers, Jakob Uszkoreit, et al. Mlp-mixer: An all-mlp architecture for vision. Advances in neural information processing systems, 34:24261–24272, 2021.

	[34] Nam N Vo and James Hays. Localizing and orienting street views using overhead imagery. In Computer Vision–ECCV 2016: 14th European Conference, Amsterdam, The Netherlands, October 11–14, 2016, Proceedings, Part I 14, pages 494–509. Springer, 2016.

	[35] Tingyu Wang, Zhedong Zheng, Chenggang Yan, Jiyong Zhang, Yaoqi Sun, Bolun Zheng, and Yi Yang. Each part matters: Local patterns facilitate cross-view geo-localization. IEEE Transactions on Circuits and Systems for Video Technology, 32(2):867–879, 2021.

	[36] Ting Wang, Zhedong Zheng, Zunjie Zhu, Yu-Fei Gao, Yi Yang, and Chenggang Yan. Learning cross-view geo-localization embeddings via dynamic weighted decorrelation regularization. ArXiv, abs/2211.05296, 2022.

	[37] Scott Workman and Nathan Jacobs. On the location dependence of convolutional neural network features. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR) Workshops, June 2015.

	[38] Scott Workman, Richard Souvenir, and Nathan Jacobs. Wide-area image geolocalization with aerial reference imagery. In 2015 IEEE International Conference on Computer Vision (ICCV), pages 3961–3969, 2015.

	[39] Chao-Yuan Wu, R Manmatha, Alexander J Smola, and Philipp Krahenbuhl. Sampling matters in deep embedding learning. In Proceedings of the IEEE international conference on computer vision, pages 2840–2848, 2017.

	[40] Hongji Yang, Xiufan Lu, and Yingying Zhu. Cross-view geo-localization with layer-to-layer transformer. In M. Ranzato, A. Beygelzimer, Y. Dauphin, P.S. Liang, and J. Wortman Vaughan, editors, Advances in Neural Information Processing Systems, volume 34, pages 29009–29020. Curran Associates, Inc., 2021.

	[41] Menghua Zhai, Zachary Bessinger, Scott Workman, and Nathan Jacobs. Predicting ground-level scene layout from aerial imagery. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pages 867–875, 2017.

	[42] Xiaohan Zhang, Xingyu Li, Waqas Sultani, Yi Zhou, and Safwan Wshah. Cross-view geo-localization via learning disentangled geometric layout correspondence. arXiv preprint arXiv:2212.04074, 2022.

	[43] Jianwei Zhao, Qiang Zhai, Rui Huang, and Hong Cheng. Mutual generative transformer learning for cross-view geo-localization. arXiv preprint arXiv:2203.09135, 2022.

	[44] Zhedong Zheng, Yunchao Wei, and Yi Yang. University-1652: A multi-view multi-source benchmark for drone-based geo-localization. In Proceedings of the 28th ACM international conference on Multimedia, pages 1395–1403, 2020.

	[45] Runzhe Zhu, Mingze Yang, Ling Yin, Fei Wu, and Yuncheng Yang. UAV’s status is worth considering: A fusion representations matching method for geo-localization. Sensors, 23(2):720, Jan 2023.

	[46] Sijie Zhu, Mubarak Shah, and Chen Chen. Transgeo: Transformer is all you need for cross-view image geo-localization. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 1162–1171, 2022.

	[47] Sijie Zhu, Taojiannan Yang, and Chen Chen. Vigor: Cross-view image geo-localization beyond one-to-one retrieval. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, pages 3640–3649, 2021.

	[48] Yingying Zhu, Hongji Yang, Yuxin Lu, and Qiang Huang. Simple, effective and general: A new backbone for cross-view image geo-localization. arXiv preprint arXiv:2302.01572, 2023.

