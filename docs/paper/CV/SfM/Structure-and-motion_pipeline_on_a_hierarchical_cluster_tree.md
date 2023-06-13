---
title: Structure-and-motion pipeline on a hierarchical cluster tree
date: 2023-06-12 19:17:09
excerpt: 分层式 SfM，还好它的利用树的思路还不太一样。
tags: 
- CV/SfM
rating: ⭐⭐⭐
status: complete
---


!!! info "相关论文"
	[Structure-and-motion pipeline on a hierarchical cluster tree](https://doi.org/10.1109/ICCVW.2009.5457435)


## 摘要
摘要： 本论文介绍了一种计算结构和运动的新型分层方案。图像通过自下而上的聚类方法组织成一棵树，使用重叠度作为距离度量。重建过程从叶节点到根节点按照这棵树的顺序进行。因此，问题被分解为较小的子问题，分别解决后再进行合并。与标准的顺序方法相比，这个框架具有较低的计算复杂度，不依赖于初始视图对，并且更好地应对漂移问题。通过正式的复杂度分析和一些实验结果来支持这些说法。

## 1 引言
近年来，人们对于从图像中自动建模建筑/城市景观的兴趣迅速增加。

文献中涵盖了多种解决这个问题的方法。这些方法可以分为两个主要类别：第一类是专门针对城市环境设计并且在实时环境下运行的方法【6, 24】。这些系统通常依赖于大量的额外信息，例如 GPS/INS 导航系统和相机校准。第二类 - 我们的贡献所在的类别 - 包括对图像进行批处理的结构与运动（Structure and Motion，SaM）流程，该流程在重建过程中不对图像场景和采集设备做任何假设【2, 15, 16, 30, 36】。

在这个背景下需要解决的主要问题是 SaM 流程的可扩展性。这引发了对效率的追求，探索了多种不同的解决方案：其中最成功的是那些旨在减少光束法平差阶段影响的方法，光束法平差阶段在特征提取之后对计算复杂性起主导作用。

提出的解决方案类别之一是所谓的分区方法 【9】。它们将重建问题分解为更小且更好条件的子问题，可以有效地进行优化。在本文中，我们提出了一种新的 SaM 分层方案，它可以将计算复杂度降低一个数量级。图像被组织成一个分层的聚类树，如图 1 所示。重建过程按照这棵树的层次结构从叶节点到根节点进行。部分重建对应于内部节点，而图像存储在叶节点中。父节点包含与其子节点相关联的两个部分重建的合并结果。通过限制每个节点使用的视图数量，引入本地光束法平差策略进一步降低了整体复杂性。这种方法在某种程度上类似于 【27】，在其中构建了一棵生成树来确定图像的处理顺序。然而，在该情况下，图像是按照标准的增量方式处理的，而不是分层方式。除了在复杂性上的改进之外，该框架还更好地处理了初始化和漂移问题，这是顺序方案的典型问题。

在文献中已经研究了使用分区方法进行 SaM 的应用。可以区分出两种主要策略。

第一种策略是直接处理光束法平差算法，利用其属性和规律。思路是将优化问题分解为更小、更易处理的组件。这些子问题可以通过分析选择，如 【32】 中应用于 SaM 的谱分割，或者可以从问题的底层 3D 结构中得出，如 【22】 所述。这些方法的计算收益是通过限制算法复杂度的组合爆炸来处理图像和特征点数量的增加。

第二种策略是选择一个包含整个解决方案的输入图像和特征点的子集。分层子采样由 【9】 首先引入，使用平衡的三焦点张量树来处理视频序列。随后，该方法由 【23】 进行了改进，添加了用于抑制冗余帧和选择张量三元组的启发式方法。在 【28】 中，序列被分割成多个段，每个段都在本地解决。然后，它们按层次结构合并，最终使用段的代表性子集。【11】 中采用了类似的方法，着重于获得良好的段划分和后续合并步骤的鲁棒性。与顺序方法相比，这些方法的优势在于它们改善了整个数据集上的误差分布，并解决了退化配置问题。无论如何，它们只适用于视频序列，无法应用于无序的稀疏图像。

最近的一项工作 【31】 在稀疏数据集上描述了一种选择子集图像的方法，其重建可近似于使用整个数据集的重建结果。通过可控地从数据集中删除冗余项，可以显著降低计算要求。然而，即使在这种情况下，所选的图像也是按增量方式处理的。

我们的策略结合了大部分上述方法的优势：i）它适用于无组织的相当大的图像集合，ii）它将问题分割为较小的实例并进行层次化组合，iii）它高效且可并行化，iv）它对于顺序方法的典型问题更不敏感，即对初始化 【33】 和漂移 【6】 的敏感性。

本文的其余部分安排如下。下一节概述了匹配阶段，然后第 3 节描述了如何构建分层聚类树。第 4 节介绍了我们的分层方法，而第 5 节解释了本地光束法平差策略。第 6 节报告了实验结果，最后在第 7 节中得出结论。

## 2 关键点匹配

在本节中，我们描述了我们的 SaM 流程中专门用于自动提取和匹配所有 $n$ 个可用图像之间关键点的阶段。其输出将输入到几何阶段，该阶段将执行实际的结构和运动恢复。

尽管这个阶段的构建模块是相当标准的技术，但我们精心设计了一个完全自动、鲁棒（通过剔除尽可能多的异常值）和计算效率高的过程。

首先，目标是以计算效率高的方式识别潜在共享大量关键点的图像，而不是尝试在每对图像之间匹配关键点（复杂度为 $\mathcal{O}(n^2)$）。我们遵循 【1】 的方法。在所有 $n$ 个图像中提取 SIFT 【18】 关键点。在这个筛选阶段，我们每个图像只考虑固定数量的描述符（我们使用了 300 个，而典型的图像包含数千个 SIFT 关键点）。然后，每个关键点描述符在特征空间中与其 $\mathcal{l}$ 个最近邻进行匹配（我们使用 $\mathcal{l}=6$）。可以通过使用 k-d 树来找到近似最近邻（我们使用了 ANN 库 【20】），在 $\mathcal{O}(nlogn)$ 的时间内完成此操作。然后构建一个 2D 直方图，在每个 bin 中记录相应视图之间的匹配数。每个图像只会与与其匹配关键点数最多的 $m$ 个图像进行匹配（我们使用 $m=8$）。因此，要匹配的图像数量为 $\mathcal{O}(n)$，其中 $m$ 是常数。例如，在由 54 个图像组成的 Pozzoveggiani 数据集上，我们的 Matlab 实现中的匹配时间从 13 小时 40 分钟减少到 50 分钟。当然，通过利用现代 GPU 的计算能力，可以进一步缩短计算时间。

匹配遵循最近邻方法 【18】，对于最近邻距离与次近邻距离之比大于阈值（在我们的实验中设置为 1.5）的关键点进行拒绝。

然后，使用 MSAC 【35】 计算匹配图像对之间的单应性矩阵和基本矩阵。设 $e_i$ 为 MSAC 之后的残差，按照 【38】，最终的内点集合是满足以下条件的点：

$$|e_i−med_je_j|<3.5σ^∗$$

其中 $σ^∗$ 是噪声尺度的鲁棒估计量：

$$ σ^∗=1.4826med_i|e_i−med_je_j|$$

这个异常值拒绝规则在 【12】 中被称为 $X8_4$。

最终，通过最小二乘法对这些内点集合重新估计模型参数（最小化几何误差的一阶近似）【4, 19】。

根据几何鲁棒信息准则（GRIC）【34】 选择更可能的模型（单应性矩阵或基本矩阵）。最后，如果两个图像之间剩余的匹配数小于一个阈值（根据 【1】 中的统计测试计算），则它们将被丢弃。

之后，将多个图像中的关键点匹配连接成轨迹，拒绝那些存在多个关键点汇合的不一致轨迹 【30】 和长度小于三帧的轨迹。

## 3 视图聚类
我们流程的第二阶段是将可用视图组织成一个层次化的聚类结构，以指导重建过程。

在文献中已经提出了图像视图聚类的算法，用于重建 【27】、全景图 【1】、图像挖掘 【26】 和场景摘要 【29】 等领域。使用的距离和聚类算法是应用特定的。

在本文中，我们使用一种适用于结构和运动重建任务的图像相似度度量。它通过考虑共有关键点的数量以及它们在图像中的分布情况来计算。具体而言，假设 $S_i$ 和 $S_j$ 分别是图像 $I_i$ 和 $I_j$ 中的匹配关键点集合：

$$a_{i,j}=\dfrac{1}{2}\dfrac{|S_i\cap S_j|}{|S_i\cup S_j|}+\dfrac{1}{2}\dfrac{CH(S_i)+CH(S_j)}{A_i+A_j}$$

其中 $CH(\cdot)$ 表示一组点的凸包的面积，$A_i(A_j)$ 表示图像 $I_i(I_j)$的总面积。第一项是一组集合之间的相似度指数，也称为 Jaccard 指数。距离为 $(1−\alpha_{i,j})$，因为 $a_{i,j}$ 的取值范围在 $[0, 1]$ 之间。

视图通过凝聚聚类方法进行分组，该方法生成一棵层次二叉聚类树，称为树状图。通常的凝聚聚类算法是自下而上进行的：从所有单例开始，算法的每次迭代将具有最小距离的两个聚类合并。通过计算聚类之间的距离，可以产生不同的算法变体，包括简单链接、完全链接和平均链接 【7】。我们选择了简单链接规则：两个聚类之间的距离由不同聚类中两个最近邻对象（最近邻）的距离确定。

简单链接聚类对我们的情况很合适，因为：i）聚类问题本身相当简单，ii）最近邻信息可以通过 ANN 轻松获取，iii）它产生“拉长”或“细长”的聚类，非常适应图像在某个区域或建筑物上的典型空间排列方式。

如下一节所述，由两个视图组成的聚类是开始重建的聚类。这两个视图必须满足两个相互冲突的要求：既要有大量共有的关键点，又要有足够大的基线，以实现良好条件的重建。第一个要求可以自动验证，因为这些聚类是根据公式 (3) 中定义的相似度而组成的最近视图。第二个要求等价于说基本矩阵必须比其他模型（如单应矩阵）更好地解释数据，这可以通过考虑 GRIC（如 【25】 中所述）来实现。

因此，我们修改链接策略，只有当以下条件成立时，才允许两个视图 i 和 j 合并为一个聚类：

$$gric(F_{i,j})<\alpha \;gric(H_{i,j}),\;\text{with}\;\alpha \ge 1$$

其中 $gric(F_{i,j})$ 和 $gric(H_{i,j})$ 分别是通过基本矩阵和单应矩阵计算得到的 GRIC 得分（我们使用 $\alpha=1.2$）。如果测试失败，则考虑第二近的元素并重复此过程。

## 4 层次结构和运动
聚类阶段生成的树状图对视图进行了层次化组织，我们的 SaM 流程将按照这个层次化组织进行。在树状图的每个节点上，需要执行一个操作，增加重建的内容（相机 + 3D 点）。这些操作包括：创建聚类时，必须执行两个视图的重建；将视图添加到聚类时，必须进行重定位-交叉步骤（与标准顺序流程相同）；当两个聚类合并时，必须解决绝对定向问题。下面详细介绍每个步骤。

**两个视图的重建**

我们假设进行两个视图重建的相机至少已经标定。这可以通过离线标定或自标定 【10】 来实现。

给定两个视图的外参数通过因子分解本质矩阵得到，如 【14】 中所述。然后通过交叉（或三角测量）重建 3D 点，并使用 $X8_4$ 剪枝重投影误差。最后运行光束法平差以改善重建结果。

**添加一个视图**

可见于要添加的视图中的重建 3D 点提供了一组 3D-2D 对应关系，可以利用它们通过线性算法 【8】 解决外方位问题，或者在视图未标定的情况下使用 DLT 进行重定位 【13】。为了处理异常值，使用 MSAC。

然后，使用最后一个视图中可见的轨迹更新 3D 结构。通过交叉获得三维点，并通过在重投影误差上进行 X84 剪枝。为了进一步保证，删除交叉条件数不良的 3D 点，使用线性系统的条件数阈值（在我们的实验中为 104）。最后，对当前重建结果进行光束法平差。

**聚类合并**

要合并的两个重建结果存在于两个不同的参考坐标系中，因此必须使用相似变换（或共线性，如果至少一个重建结果未标定）将其注册到另一个重建结果上。由于构造时它们具有一些共同的 3D 点，可以使用 MSAC 解决绝对定向问题。一旦相机注册完毕，通过交叉重新计算共同的 3D 点，采取与之前相同的注意事项，即在重投影误差上进行 $X8_4$ 剪枝和条件数测试。合并后，还对任何在合并后可见的轨迹进行交叉计算。最后，通过三角测量长度为两个的轨迹，使用重投影误差的异常值剔除（$X8_4$），增加了最终重建结果中的重建点数。

### 4.1 复杂性分析

上述的层次化方法使得与顺序 SaM 流程相比，计算复杂性降低了。实际上，如果视图数为 $n$，并且每个视图都向重建中添加一个恒定数量的点 $\mathcal{l}$，则顺序 SaM 的计算复杂性是 $\mathcal{O}(n^5)$，而我们的层次化 SaM 的计算复杂性（在最佳情况下）是 $\mathcal{O}(n^4)$，如附录 A 所示。

最坏的情况是每次添加一个视图来增长单个聚类。在这种情况下，对应于顺序情况，树状图极不平衡，复杂性降至 $\mathcal{O}(n^5)$。经验上我们发现，树状图通常是相当平衡的，因此我们声称在实践中可以达到最佳情况的复杂性。

## 5 局部光束法平差

为了进一步降低复杂性，我们采用了一种策略，即从每个聚类 $C$ 中选择一个恒定数量的 $k$ 个视图，用于光束法平差，而不是使用整个聚类。然而，这些活动视图并非一劳永逸地固定，而是根据正在添加的对象（单个视图或另一个聚类 $C^\prime$）的情况灵活选择。这种策略是本地光束法平差的一种实例 【21, 37】，在视频序列中经常使用，其中活动视图是最近的视图。

让我们集中讨论聚类合并步骤，因为一个视图的添加是后者的特例。考虑属于聚类 $C$ 和 $C^\prime$ 的点集：我们首先确定在 $C$ 和  $C^\prime$ 中这些点可见的视图。在这些视图中，根据在第 3 节中已计算的距离矩阵，我们选择 $k$ 个最接近的视图对作为活动视图。

光束法平差涉及到活动视图和可以从它们重建的点作为变量，以及一些其他锚定视图，仅用于计算重投影误差。锚定视图是每个聚类中最接近活动视图的 $k$ 个视图；它们不会被光束法平差移动，但会对与其他结构相关的 3D 点进行锚定，起到使被光束法平差的结构部分更加刚性的作用。图 2 说明了这个思路。

在最后，可以通常运行一个包含所有视图和所有点的光束法平差，以获得最佳解。如果由于数据集的规模而无法实现这一点，这种策略仍然能够产生次优的结果。

### 5.1 复杂性分析

除了最后一个光束法平差外，现在每个光束法平差仅在恒定数量的视图上运行，因此其成本为 $\mathcal{O}(1)$。光束法平差的数量为 $\mathcal{O}(n)$，因此总成本由最后一个光束法平差主导，为 $\mathcal{O}(n^4)$。尽管渐近复杂性与之前相同，但本地光束法平差显然减少了总操作数。

顺序方法与本地光束法平差相结合也能够实现相同的复杂性 $\mathcal{O}(n^4)$。然而，层次化方法易于并行化，并且更加稳健和有效，下一节的实验将会展示这一点。

  
## 6 实验
我们在作者用已知内部参数的消费级相机拍摄的几个数据集上测试了我们的算法（以下简称为 SAMANTHA）。图 3 和图 4 分别展示了从“Piazza Erbe”和“Piazza Bra”数据集中进行的重建结果。

我们将我们的结果与 BUNDLER 【3】 产生的结果进行了比较，BUNDLER 是一个 C++实现的最先进的顺序 SaM 流水线。在我们的流水线中，我们使用了 【17】 中描述的 C++实现的束调整（BA）。只报告进行 BA 所花费的时间，以消除由于我们的软件部分使用 Matlab 编写而产生的差异，并与我们的复杂性分析保持一致。此外，BUNDLER 在匹配阶段非常慢，因为它将每个视图与其他所有视图进行匹配。所有实验都在相同的硬件上运行（Intel Core2 Duo E4600@2.4Ghz，2Gb RAM）。

表 1 报告了比较结果。结果显示，SAMANTHA 的时间比 BUNDLER 显著少，而在重建的视图和点数方面没有主要差异。

作为一个顺序算法，*BUNDLER* 对初始化非常敏感。实际上，对于一些数据集，有必要仔细选择初始对，以使其产生有意义的解。在“Piazza Bra”的情况下，共尝试了四个初始对：默认选择的一个和另外三个根据我们的聚类使用的相同标准选择的。在所有情况下，结果仅为部分重建（通过重建的点数较少来证明），且存在明显的不对齐问题（图 6）。对于“Tribuna”数据集也存在类似的结果。

在表 2 中，我们分析了本地 BA 策略在活动视图数量、计算时间和重建质量之间的权衡。正如预期的那样，随着活动视图数量的减少，计算时间逐渐减少，而在重建的点数和视图方面没有明显的损失。点数和视图数量的小变化在算法的相同运行中是预期的和正常的，因为存在非确定性的步骤。因此，与基线情况（所有活动视图）相比，与之平均对齐误差增加。

最后，使用很少的活动视图时，SAMANTHA 可能无法合并聚类。在出现这种情况之前，我们注意到由于在不理想的情况下需要更多迭代才能使束调整收敛，导致 BA 运行时间增加。这促使我们建议使用足够大的（20+）活动视图数量，以确保快速可靠的计算。

## 7 结论与未来工作

我们开发了一种新颖的结构与运动流水线，通过基于视图聚类的分层方案，可使计算效率提高一个数量级，从而更高效、更有效地解决了初始化和漂移问题。

未来的研究将致力于利用其固有的并行性，推动我们的方法在越来越大的数据集上的应用。

  
## A 复杂性分析 
对于具有 $m$ 个点和 $n$ 个视图的捆绑调整而言，其成本为 $\mathcal{O}(mn(m+2n)2)$ 【28】，因此当 $m=\mathcal{l}n$ 时，复杂度为 $\mathcal{O}(n_4)$。

在顺序的 SaM 中，添加视图 $i$ 需要进行一定数量的具有 $i$ 个视图的光束法平差（通常为一到两个），因此复杂度为

$$\sum^{n}_{i=1}=\mathcal{O}(i^4)=\mathcal{O}(n^5)$$

对于分层方法，考虑一个树状图节点，其中两个聚类被合并为一个大小为 $n$ 的聚类。调整该聚类的成本 $T(n)$ 等于 $\mathcal{O}(n^4)$ 加上对左子树和右子树执行相同操作的成本。在假设树状图平衡的情况下，即两个聚类具有相同大小时，该成本由 $2T(n/2)$ 给出。因此，在最佳情况下，渐近时间复杂度 $T$ 的解为以下递归的解：

$$T(n)=2T(n/2)+\mathcal{O}(n^4)$$

根据 Master 定理的第三种情况，$T(n)=\mathcal{O}(n^4)$

## 致谢 
感谢 Roberto Posenato 和 Cheng Dong Seon 提供的许多有益建议。感谢 A. Vedaldi 和 B. Fulkerson 的 VLFeat，David M. Mount 和 Sunil Arya 的 ANN，M. Lourakis 和 A. Argyros 的 SBA，以及 N. Snavely 的 Bundler。本研究由欧盟项目 FP7 SAMURAI 资助，授予号 FP7-SEC-2007–01 No. 217899。


## 参考文献

??? info "References"

	[1] M. Brown and D. Lowe. Recognising panoramas. In Proceedings of the 9th International Conference on Computer Vision, volume 2, pages 1218–1225, October 2003.

	[2] M. Brown and D. G. Lowe. Unsupervised 3D object recognition and reconstruction in unordered datasets. In Proceedings of the International Conference on 3D Digital Imaging and Modeling, June 2005.

	[3] http://phototour.cs.washington.edu/bundler/.

	[4] O. Chum, T. Pajdla, and P. Sturm. The geometric error for homographies. Computer Vision and Image Understanding, 97(1):86–102, 2005.

	[5] T. H. Cormen, C. E. Leiserson, R. L. Rivest, and C. Stein. Introduction to Algorithms. The MIT Press, Cambridge, MA, USA, 2001.

	[6] N. Cornelis, B. Leibe, K. Cornelis, and L. V. Gool. 3D urban scene modeling integrating recognition and reconstruction. International Journal of Computer Vision, 78(2-3):121–141, July 2008.

	[7] R. O. Duda and P. E. Hart. Pattern Classification and Scene Analysis, pages 98–105. John Wiley and Sons, 1973.

	[8] P. D. Fiore. Efficient linear solution of exterior orientation. IEEE Transactions on Pattern Analysis and Machine Intelligence, 23(2):140–148, 2001.

	[9] A. W. Fitzgibbon and A. Zisserman. Automatic camera recovery for closed and open image sequences. In Proceedings of the European Conference on Computer Vision, pages 311–326, 1998.

	[10] A. Fusiello, A. Benedetti, M. Farenzena, and A. Busti. Globally convergent autocalibration using interval analysis. IEEE Transactions on Pattern Analysis and Machine Intelligence, 26(12):1633–1638, December 2004.

	[11] S. Gibson, J. Cook, T. Howard, R. Hubbold, and D. Oram. Accurate camera calibration for off-line, video-based augmented reality. Mixed and Augmented Reality, IEEE / ACM International Symposium on, 2002.

	[12] F. Hampel, P. Rousseeuw, E. Ronchetti, and W. Stahel. Robust Statistics: the Approach Based on Influence Functions. Wiley Series in probability and mathematical statistics. John Wiley & Sons, 1986.

	[13] R. Hartley and A. Zisserman. Multiple View Geometry in Computer Vision. Cambridge University Press, 2nd edition, 2003.

	[14] R. I. Hartley. Estimation of relative camera position for uncalibrated cameras. In Proceedings of the European Conference on Computer Vision, pages 579–587, 1992.

	[15] A. Irschara, C. Zach, and H. Bischof. Towards wiki-based dense city modeling. In Proceedings of the 11th International Conference on Computer Vision, pages 1–8, 2007.

	[16] G. Kamberov, G. Kamberova, O. Chum, S. Obdrzalek, D. Martinec, J. Kostkova, T. Pajdla, J. Matas, and R. Sara. 3D geometry from uncalibrated images. In Proceedings of the 2nd International Symposium on Visual Computing, November 6-8 2006.

	[17] M. Lourakis and A. Argyros. The design and implementation of a generic sparse bundle adjustment software package based on the Levenberg-Marquardt algorithm. Technical Report 340, Institute of Computer Science - FORTH, Heraklion, Crete, Greece, August 2004.

	[18] D. G. Lowe. Distinctive image features from scale-invariant keypoints. International Journal of Computer Vision, 60(2):91–110, 2004.

	[19] Q.-T. Luong and O. D. Faugeras. The fundamental matrix: Theory, algorithms, and stability analysis. International Journal of Computer Vision, 17:43–75, 1996.

	[20] D. M. Mount and S. Arya. Ann: A library for approximate nearest neighbor searching. In http://www.cs.umd.edu/mount/ANN/, 1996.

	[21] E. Mouragnon, M. Lhuillier, M. Dhome, F. Dekeyser, and P. Sayd. Real-time localization and 3D reconstruction. In Proceedings of the International Conference on Computer Vision and Pattern Recognition, pages 363–370, 2006.

	[22] K. Ni, D. Steedly, and F. Dellaert. Out-of-core bundle adjustment for large-scale 3D reconstruction. In Proceedings of the International Conference on Computer Vision, pages 1–8, 2007.

	[23] D. Nist ́ er. Reconstruction from uncalibrated sequences with a hierarchy of trifocal tensors. In Proceedings of the European Conference on Computer Vision, pages 649–663, 2000.

	[24] M. Pollefeys, D. Nist ́ er, J. M. Frahm, A. Akbarzadeh, P. Mordohai, B. Clipp, C. Engels, D. Gallup, S. J. Kim, P. Merrell, C. Salmi, S. Sinha, S. Sinha, B. Talton, L. Wang, Q. Yang, H. Stew ́ enius, R. Yang, G. Welch, and H. Towles. Detailed real-time urban 3D reconstruction from video. International Journal of Computer Vision, 78(2-3):143–167, 2008.

	[25] M. Pollefeys, F. Verbiest, and L. V. Gool. Surviving dominant planes in uncalibrated structure and motion recovery. In Proceedings of the European Conference on Computer Vision, pages 837–851, 2002.

	[26] T. Quack, B. Leibe, and L. Van Gool. World-scale mining of objects and events from community photo collections. In Proceedings of the International Conference on Content-based Image and Video Retrieval, pages 47–56, 2008.

	[27] F. Schaffalitzky and A. Zisserman. Multi-view matching for unordered image sets, or "how do I organize my holiday snaps?". In Proceedings of the 7th European Conference on Computer Vision, pages 414–431, 2002.

	[28] H.-Y. Shum, Q. Ke, and Z. Zhang. Efficient bundle adjustment with virtual key frames: A hierarchical approach to multi-frame structure from motion. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, June 1999.

	[29] I. Simon, N. Snavely, and S. M. Seitz. Scene summarization for online image collections. In Proceedings of the International Conference on Computer Vision, 2007.

	[30] N. Snavely, S. M. Seitz, and R. Szeliski. Photo tourism: exploring photo collections in 3D. In SIGGRAPH: International Conference on Computer Graphics and Interactive Techniques, pages 835–846, 2006.

	[31] N. Snavely, S. M. Seitz, and R. Szeliski. Skeletal graphs for efficient structure from motion. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pages 1–8, 2008.

	[32] D. Steedly, I. Essa, and F. Dellaert. Spectral partitioning for structure from motion. In Proceedings of the International Conference on Computer Vision, pages 649–663, 2003.

	[33] T. Thorm ̈ ahlen, H. Broszio, and A. Weissenfeld. Keyframe selection for camera motion and structure estimation from multiple views. In Proceedings of the European Conference on Computer Vision, pages 523–535, 2004.

	[34] P. H. S. Torr. An assessment of information criteria for motion model selection. Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pages 47–53, 1997.

	[35] P. H. S. Torr and A. Zisserman. MLESAC: A new robust estimator with application to estimating image geometry. Computer Vision and Image Understanding, 78:2000, 2000.

	[36] M. Vergauwen and L. V. Gool. Web-based 3D reconstruction service. Machine Vision and Applications, 17(6):411–426, 2006.

	[37] Z. Zhang and Y. Shan. Incremental motion estimation through modified bundle adjustment. In Proceedings of the International Conference on Image Processing, pages II343–6, Sept. 2003.

	[38] M. Zuliani. Computational Methods for Automatic Image Registration. PhD thesis, University of California, Santa Barbara, Dec 2006.
