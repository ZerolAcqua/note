---
title: Skeletal graphs for efficient structure from motion
date: 2023-08-22 20:47:31
status: new
---


!!! info "论文链接"
	原文：Noah Snavely, Steven M. Seitz, Richard Szeliski. [Skeletal graphs for efficient structure from motion](https://doi.org/10.1109/CVPR.2008.4587678)

!!! warning "注意"
    由 claude 和 chatgpt 翻译整理


## 摘要

当前一代非结构化 SFM 方法根本无法扩展到成千上万的图像。此外，诸如子采样和分层分解 【9, 17】 等 SFM 缩放技术对有序视频序列有效，但将其应用于互联网收集更具挑战性，因为后者往往是无序的，在某些区域（热门观点）过度采样，在其他区域下采样。直观地说，重建一个场景的难度应该取决于场景本身的复杂性，而不是图像的数量。对于这样大型的冗余收集，一小部分图像可能就足够表示场景的大部分信息。如果我们能识别这样一个视图子集——一个骨架集——我们可以将重建过程集中在这些骨架图像上，并产生真正可扩展的算法。

我们优化不确定性而不是准确性，因为前者可以在没有地面真实几何知识的情况下计算。

其次，我们使用图像数量作为运行时间的代理，这使得一种算法无关的效率测量成为可能。通过约束骨架集必须“跨越”完整集并通过姿态估计和三角测量使完整集中的所有图像和 3D 点重建来确保完整性。我们通过将完整图像集的联合协方差表示为图来形成我们的问题，其中每个图像是一个节点，边编码了图像对之间的相对姿态不确定性（协方差）。通过组合沿着图中节点之间的路径估计的协方差，估计任意两张图像之间的相对姿态不确定性。然后问题是确定具有最小内部节点数的骨架子图，该子图跨越完整图，同时实现对完整协方差的所需边界。对于每对图像，我们将它们在原始图中的估计相对不确定性与骨架图中的不确定性进行比较，并要求后者不超过前者的固定常数 t。尽管这个问题是 NP 完全的，但我们开发了一种快速逼近方法，可保证对协方差的该约束，并且在实践中大大减少了问题的大小。

我们的实验结果表明，结果方法通过一个数量级以上增加了大规模问题的效率，几乎没有损失准确性；此外，我们重建了所有图像，而不仅仅是骨架集。骨架集仅用于使计算更高效。

## 概览

在本文中，重建是指一组恢复的 3D 相机和点参数。SFM 问题是使用来自一组图像的测量（以特征对应形式）来构建重建。我们的目标是使用更少的测量集，但仍然计算出尽可能高质量的重建。

如何衡量重建的质量

- 一些理想的属性是完整性和准确性，即重建应该跨越图像中可见的场景的所有部分，并且应该尽可能贴近地面真实的场景和相机位置。如果完整性和准确性是唯一的考虑，那么 SFM 应该使用所有可用的测量。
- 效率也很重要。对于互联网图像集，这种权衡尤其相关。在这些集合中，通常有大量流行的，因此冗余的视图，以及一些稀缺但重要（用于重建）的视图。如果我们能识别这小部分重要测量，那么我们可以潜在地更快地重建场景。

我们将为重建选择的图像集称为骨架集，并将我们的问题定义如下：给定无序图像集，找到一个小的子集，它产生的重建质量与完整图像集相比有限损失。这样的重建将是完整解决方案的近似。此外，它可能是一个最终捆绑调整的良好初始化，当运行所有测量时，通常会恢复任何丢失的质量。

定义质量

- 首先考虑准确性，即恢复的相机和点应该尽可能忠实于实际场景。没有真值，不可能直接测量准确性。然而，可以估计重建中的不确定性（协方差），这是准确性的统计估计。
    
    通过沿 $P$ 和 $Q$ 之间的路径链合协方差矩阵。要计算确切的协方差，我们需要整合所有从 $P$ 到 $Q$ 的路径上的信息。然而，从 $P$ 到 $Q$ 的最短路径为真协方差提供了一个上界。
    

权衡效率与准确度

- 我们删除的边越多（创建的叶子越多），重建任务越快，但估计的不确定性也会增长。我们用一个称为伸展因子的参数 t 来表达这种权衡。对于给定的 t 值，骨架图问题是找到具有最大叶数的子图 $G_\mathcal{S}$，受到任意一对相机 $(P，Q)$ 在 $G_\mathcal{S}$ 中的距离（最短路径的长度）最多比 $P$ 和 $Q$ 在 $G_\mathcal{I}$ 中的距离长 t 倍的约束。具有此属性的子图 $G_\mathcal{S}$ 称为 $G_\mathcal{I}$ 的 t-张力子图【2】，因此我们的问题是找到 $G_\mathcal{I}$ 的最大叶 t-张力子图。

多度重叠的必要性

- 该方法的一个问题是图像图 $G_\mathcal{I}$ 不是 SFM 的足够表达图像连接性的模型。为什么这么说，考虑三张图像 $A$、$B$ 和 $C$，其中 $A$ 和 $B$ 重叠，$B$ 和 $C$ 也重叠，但 $A$ 和 $C$ 不重叠。这些节点在 $G_\mathcal{I}$ 中形成一个连通集。然而，不能确定两帧重建 $(A,B)$ 和 $(B,C)$ 之间的尺度，并且无法从这些图像构建一致的重建。为了确定尺度， $(A,B)$ 和 $(B,C)$ 必须看到至少一个公共点。因此，任何按顺序通过节点 $A$、$B$、$C$ 的路径都不是可实现的重建链（我们称这样的路径在 $G_\mathcal{I}$ 中不可行）。
- 为了解决这个问题，我们定义另一个图，图像对图 $G_\mathcal{P}$。$G_\mathcal{P}$ 对每一对重建的图像有一个节点，并在共享公共特征的重建之间有一条边。$G_\mathcal{P}$ 还与每个图像的节点一起增强，并构建为图像 $P$ 和 $Q$ 之间的路径与 $G_\mathcal{I}$ 中的类似路径具有相同的权重；唯一的区别是只能在 $G_\mathcal{P}$ 中遍历可行路径。显示此构造的 $G_\mathcal{I}$ 和 $G_\mathcal{P}$ 的示例如图 2 所示。这种更高级别的连接性施加了骨架图的附加约束：它必须产生一个可行的单一重建。表达这一点的一种方法是将 $G_\mathcal{I}$ 的子图 $G_\mathcal{S}$ 嵌入 $G_\mathcal{P}$ 中，作为包含与 $G_\mathcal{S}$ 边对应的节点以及这些节点之间的任何边的 $G_\mathcal{P}$ 的子图。为了使骨架图可行，$G_\mathcal{S}$ 嵌入 $G_\mathcal{P}$ 必须是连通的。

## 构建 $G_\mathcal{I}$ 和 $G_\mathcal{P}$

分三个阶段计算 $G_\mathcal{I}$ 和 $G_\mathcal{P}$

- 为每对匹配图像创建两帧重建，并在过程中删除重复图像
- 为每对计算相对的相机位置协方差
- 检查哪些图像对的两帧重建是重叠的（因此在 $G_\mathcal{P}$ 中是边）。

具体步骤

- 通过从每个图像中提取 SIFT 特征【14】，在每对图像之间匹配特征，并形成匹配的连通组件以产生轨迹，获得对应关系。
- 使用 Nistér 的五点相对姿势算法【18】在 RANSAC 循环内为每个匹配图像对计算重建。五点算法需要两台相机都经过标定。
- 在重建一对图像后，检查图像是否是近似重复的，即图像内容是否非常相似，或者一个图像是否被其他图像包含。通过重复检测，我们通常可以避免处理许多对。
- 重建了一对图像 $(I,J)$ ，我们就估计两个相机位置的协方差。在捆绑调整过程中，SBA 使用 Schur 补来计算减小的相机系统的海森矩阵 $H_{CC}$ 【24】。我们可以通过求反 $H_{CC}$ 并选择对应于相机位置的子矩阵来估计相机中的协方差。但是，由于标度自由度，$H_{CC}$ 是奇异的，因此我们对重建添加约束。
- 计算协方差后，我们构造图像对图 $G_\mathcal{P}$。回想一下，$G_\mathcal{P}$ 的每个节点代表一个成对重建，每个重叠重建对 $(I,J)$ 和 $(J,K)$ 之间有一条边。剩下的主要任务是决定哪些节点对是连接的。

## 计算骨架集

计算一般图的最小 t-张力子图的问题是 NP 完全的【5】，所以找到最大叶 t-张力子图问题的精确解很可能无法高效地找到。

我们提出了一种逼近算法来计算骨架集，该算法由两步组成：

- 构造 $G_\mathcal{I}$ 的生成树  $T_\mathcal{S}$。 $T_\mathcal{S}$ 的构造在计算具有大量叶节点的树（最大叶生成树）与计算具有小伸展因子的树（t-张力子图）之间进行权衡。因为没有树可能具有伸展因子为 t。
- 第二步是向 $T_\mathcal{S}$ 添加额外的边以满足 t-张力子图的属性。

### 构造生成树

计算最大叶生成树（MLST）的简单贪心逼近算法【10】。该算法的思想是逐个添加节点来增长一棵树，从最大度数的节点开始。然后我们修改该算法以考虑边权重。

### 从 MLST 到 t-张力子图

为了保证骨架图的伸展因子至多为 t，我们可能需要添加额外的边，形成具有环的图 $G_\mathcal{S}$。为了确定要添加哪些边，我们需要一种方法来测试是否已经达到目标伸展因子。虽然乍一看我们需要计算所有节点对之间的路径来检查这一点，但只检查相邻对（I，J）之间的路径就足够了。

总结步骤

- 为图像计算特征对应关系。
- 为每对匹配计算重建和协方差，并删除重复项（第 3 节）。
- 修剪低重要性重建。
- 从 $G_\mathcal{I}$ 构造 MLST（第 4.1 节）。
- 添加边以保证伸展因子（第 4.2 节）。
- 识别和重建骨架集。
- 使用姿态估计添加剩余图像。
- 可选地，运行最终捆绑调整。

## 实验

我们算法的运行时间是针对整个流水线（计算成对重建、构建骨架图和重建场景）。对于最大的集合特拉法加（最大集合），基线方法在 50 天后仍在运行。结果显示，与基线方法相比，我们的方法花费的时间显著更少，随着数据集大小的增加，性能提升也急剧增加。加速度从圣彼得的 2 倍提高到特拉法加广场最大集合的 40 倍左右。同时，我们的算法恢复了基线方法重建的大多数图像。丢失了一些图像；这些图像大多与其余部分联系很弱，在构建骨架图时可能被错误地修剪为不可行。我们的方法在由单人拍摄的集合（Pisa2）上也效果很好，尽管骨架集中的图像比例比互联网集合略高。对于大多数数据集，我们的算法在重建上花费的时间比构建骨架图更长；对于一些特别密集的集合（例如万神庙），预处理时间更长。

## 结论

我们开发了一种通过计算骨架图来重建互联网照片收集的算法，并且表明这种方法可以提高一个数量级或更高的效率，几乎没有损失准确性。我们的工作为未来工作指明了许多有趣的方向。例如，我们希望找到使用更复杂不确定性模型的方法，例如考虑相机方向的不确定性，也许还考虑场景结构的不确定性，或者通过考虑图像对之间的多条路径。考虑三元组而不是对可能也会很有成果，因为它在表示连接性和提高鲁棒性方面更方便。将我们的模型适配为以更细粒度移除测量也很有趣，例如移除点和图像。我们最终希望将工作扩展到更大的集合，包括整个城市。

## 参考文献

??? info "References"

	[1] A. Akbarzadeh et al. Towards urban 3D reconstruction from video. In Proc. Int. Symp. on 3D Data Processing, Visualization, and Transmission, pages 1–8, 2006.
    
	[2] I. Alth ̈ ofer, G. Das, D. Dobkin, D. Joseph, and J. Soares. On sparse spanners of weighted graphs. Discrete Comput. Geom., 9(1):81–100, 1993.
    
	[3] O. Booij, Z. Zivkovic, and B. Kr ̈ ose. Sparse appearance-based modeling for robot localization. In Proc. Int. Conf. on Intelligent Robots and Systems, pages 1510–1515, 2006.
    
	[4] M. Brown and D. Lowe. Unsupervised 3d object recognition and reconstruction in unordered datasets. In Proc. 3DIM, pages 56–63, 2005.
    
	[5] L. Cai. NP-completeness of minimum spanner problems. Discrete Appl. Math., 48(2):187–194, 1994.
    
	[6] O. Chum, J. Philbin, J. Sivic, M. Isard, and A. Zisserman. Total recall: Automatic query expansion with a generative feature model for object retrieval. In Proc. ICCV, 2007.
    
	[7] N. Cornelis, K. Cornelis, and L. V. Gool. Fast compact city modeling for navigation pre-visualization. In Proc. CVPR, pages 1339–1344, 2006.
    
	[8] A. J. Davison. Active search for real-time vision. In Proc. ICCV, pages 66–73, 2005.
    
	[9] A. W. Fitzgibbon and A. Zisserman. Automatic camera recovery for closed or open image sequences. In Proc. ECCV, pages 311–326, 1998.
    
	[10] S. Guha and S. Khuller. Approximation algorithms for connected dominating sets. Algorithmica, 20(4):374–387, 1998.
    
	[11] R. I. Hartley and A. Zisserman. Multiple View Geometry. Cambridge University Press, Cambridge, UK, 2004.
    
	[12] B. K. P. Horn, H. M. Hilden, and S. Negahdaripour. Closed-form solution of absolute orientation using orthonormal matrices. J. Opt. Soc. of America A, 5:1127–1135, July 1988.
    
	[13] M. Lourakis and A. Argyros. The design and implementation of a generic sparse bundle adjustment software package based on the levenberg-marquardt algorithm. Tech. Report 340, Inst. of Computer Science-FORTH, Heraklion, Greece, 2004.
    
	[14] D. Lowe. Distinctive image features from scale-invariant keypoints. Int. J. of Computer Vision, 60(2):91–110, 2004.
    
	[15] D. Martinec and T. Pajdla. Robust rotation and translation estimation in multiview reconstruction. In Proc. CVPR, 2007.
    
	[16] K. Ni, D. Steedly, and F. Dellaert. Out-of-core bundle adjustment for large-scale 3D reconstruction. In Proc. ICCV, 2007.
    
	[17] D. Nist ́ er. Reconstruction from uncalibrated sequences with a hierarchy of trifocal tensors. In Proc. ECCV, pages 649–663, 2000.
    
	[18] D. Nist ́ er. An efficient solution to the five-point relative pose problem. IEEE Trans. on Pattern Analysis and Machine Intelligence, 26(6):756–777, 2004.
    
	[19] J. Repko and M. Pollefeys. 3d models from extended uncalibrated video sequences. In Proc. 3DIM, pages 150–157, 2005.
    
	[20] F. Schaffalitzky and A. Zisserman. Multi-view matching for unordered image sets, or “How do I organize my holiday snaps?”. In Proc. ECCV, volume 1, pages 414–431, 2002.
    
	[21] N. Snavely, S. M. Seitz, and R. Szeliski. Photo tourism: exploring photo collections in 3D. In SIGGRAPH Conf. Proc., pages 835–846, 2006.
    
	[22] D. Steedly, I. Essa, and F. Delleart. Spectral partitioning for structure from motion. In Proc. ICCV, pages 996–103, 2003.
    
	[23] D. Steedly and I. A. Essa. Propagation of innovative information in non-linear least-squares structure from motion. In Proc. ICCV, pages 223–229, 2001.
    
	[24] B. Triggs, P. McLauchlan, R. Hartley, and A. Fitzgibbon. Bundle adjustment – a modern synthesis. In Vision Algorithms: Theory and Practice, volume 1883 of Lecture Notes in Computer Science, pages 298–372, 2000.
    
	[25] M. Vergauwen and L. V. Gool. Web-based 3D reconstruction service. Mach. Vis. Appl., 17(2):321–329, 2006.
