---
title: 【论文翻译】Structure-from-Motion Revisited
date: 2023-05-29 21:29:09
excerpt: SfM
tags: 
- CV/SfM
- 文献类型/综述
rating: ⭐⭐⭐
status: inprogress
---


!!! info "相关论文"
	[Structure-from-Motion Revisited](https://doi.org/10.1109/CVPR.2016.445)


## 摘要 

增量式结构从运动是从无序图像集合中实现三维重构的普遍策略。
尽管增量式重建系统在所有方面都有了极大的进步，包括鲁棒性，准确性，完整性和可扩展性，但是对于构建一个真正的通用管道来说，这些都仍然是关键问题。我们提出了一种新的 SfM 技术，其中改进了现有技术，使其更接近这一最终目标。完整的重建流程作为开源实现公开发布。

## 1. 导言

多年来，无序图像的结构从运动(SfM)已经得到了极大的发展。早期的自标定度量重建系统【42, 6, 19, 16, 46】为无序互联网照片集合【47, 53】和城市场景【45】的第一批系统奠定了基础。受到这些作品的启发，已经开发了越来越大规模的重建系统，以重建成数百万【1】和数以百万计【20, 62, 51, 50】到最近的一亿互联网照片【30】。已经提出了各种 SfM 策略，包括增量式【53, 1, 20, 62】，分层式【23】和全局方法【14, 61, 56】。可以说，增量式 SfM 是重建无序照片集合最流行的策略。尽管它的使用非常广泛，我们仍然没有设计出一个真正通用的 SfM 系统。尽管现有系统在最先进技术方面已经取得了巨大进步，但是稳健性、准确性、完整性和可扩展性仍然是增量式 SfM 中的关键问题，这些问题妨碍了其作为通用方法的应用。在本文中，我们提出了一种新的 SfM 算法，以接近这一最终目标。该新方法在一系列具有挑战性的数据集上进行了评估，并将代码作为开源实现 COLMAP 共享给研究社区，可在 https://github.com/colmap/colmap 上获得。
## 2. SfM 回顾

SfM 是从不同视角拍摄的图像的投影中重建三维结构的过程。增量式的 SfM（在本文中称为 SfM）是一个顺序处理管道，具有迭代重建部分（图 2）。它通常从特征提取和匹配开始，然后进行几何验证。生成的场景图作为重建阶段的基础，使用精心选择的两视图重建为模型种子，随后逐渐注册新图像，三角化场景点，过滤异常值，并使用光束法平差（BA）来细化重建。以下各节详细介绍了这个过程，定义了本文中使用的符号，并引介相关研究。

### 2.1. 重叠图像搜索

第一阶段是重叠图像搜索，其找到了输入图像 $\mathcal{I}=\{I_i~|~i=1\dots N_I\}$ 中的场景重叠并确定了在重叠的图像中相同点的投影。输出为几何验证的图像对 $\bar{\mathcal{C}}$  和每个点的图像投影的图。

**特征提取**。对于每个图像 $I_i$，SfM 检测到在位置 $\mathrm{x}_j\in \mathbb{R}^2$ 处具有外观描述符 $\mathrm{f}_j$  的局部特征集合 $\mathcal{F}_\mathrm{i} = \{(\mathrm{x}_j，\mathrm{f}_j)~|~j = 1\dots N_{F_i}\}$。特征应该对辐射和几何变换具有不变性，以便 SfM 可以在多幅图像中唯一识别它们【41】。SIFT【39】，其改进方法 【59】和最近的特征【9】是鲁棒性方面的黄金标准。与此同时，二进制特征提供更好的效率，但鲁棒性较差【29】。

**匹配**。接下来，SfM 通过利用特征 $\mathcal{F}_i$ 作为图像外观描述来发现拍摄到相同场景部分的图像。原始方法会测试每对图像是否存在场景重叠；它通过找到图像 $I_a$ 上与图像 $I_b$ 中的每个特征最相似的特征，并利用相似性度量比较特征 $\mathrm{f}_j$ ，来搜索特征对应。该方法的计算复杂度为 $\mathcal{O}(N^2_IN_{F_i})$，对于大规模图像集合来说代价太大。因此，有多种方法解决可扩展和高效的匹配问题【1,20,37,62,28,49,30】。输出为一组潜在重叠的图像对 $\mathcal{C}= \{\{I_a, I_b\}~|~ I_a, I_b \in \mathcal{I}, a < b\}$ 和它们相应的特征对 $\mathcal{M}_{ab}\in \mathcal{F}_a\times \mathcal{F}_b$。

**几何验证**。第三阶段验证了潜在重叠的图像对 $\mathcal{C}$。由于匹配仅基于外观，因此不能保证对应的特征实际上映射到相同的场景点。因此，SfM 通过尝试使用投影几何来估计在图像之间映射特征点的变换来验证匹配。针对图像对的不同空间配置，使用不同的映射描述它们的几何关系。一个单应变换 $\mathrm{H}$ 描述了纯旋转或移动相机捕捉平面场景的变换【26】。基本矩阵 $\mathrm{E}$ (已标定）或基本矩阵 $\mathrm{F}$ （未标定）描述了移动相机的对极几何关系【26】，并可以使用三基张量【26】扩展到三个视图。如果一个有效的变换在图像之间映射了足够数量的特征，那么它们被认为是几何上验证的。由于匹配得到的对应通常存在离群值，需要使用鲁棒估计技术，如 RANSAC【18】。此阶段的输出是经过几何验证的图像对 $\bar{\mathcal{C}}$，它们的内点对应关系 $\bar{\mathcal{M}}_{ab}$，以及可选的它们的几何关系 $G_{ab}$ 的描述。为了决定适当的关系，可以使用决策准则，如 GRIC 【57】，或使用 QDEGSAC【21】等方法。这一阶段的输出是一个称为场景图【54, 37, 48, 30】的结构，其中图像作为节点，经过验证的图像对作为边。

### 2.2. 增量重建

重建阶段的输入是场景图。输出是已注册图像的姿态估计 $\mathcal{P} = \{\mathrm{P}_c \in \mathrm{SE}(3) ~|~ c = 1\dots N_P\}$ 和重建场景结构，作为点集 $\mathcal{X} = \{\mathrm{X}_k \in \mathbb{R}^3 ~|~ k = 1\dots N_X\}$。

**初始化**。SfM 使用精心选定的双视重建【7, 52】初始化模型。选择适合的初始图像对至关重要，因为重建过程可能无法从错误的初始化中恢复。此外，重建的鲁棒性、精度和性能取决于增量过程的种子位置。从图像图中具有许多重叠相机的密集位置进行初始化通常会产生更鲁棒和准确的重建结果，因为这增加了冗余信息。相反，从较稀疏的位置进行初始化可以降低运行时间，因为在重建过程中光束法平差处理的是累积的整体稀疏问题。

**图像注册**。从度量重建开始，可以通过解决透视 n 点（PnP）问题【18】来将新图像注册到当前模型中，使用特征对应关系将已注册图像中的三角化点（ 2D-3D 对应关系）用于计算。PnP 问题涉及估计姿态 $\mathrm{P}_c$，对于未标定相机还包括其内参数。因此，集合 $\mathcal{P}$ 通过新注册图像的姿态 $\mathrm{P}_c$ 进行扩展。由于 2D-3D 对应关系通常包含离群值，对于已标定相机，通常使用 RANSAC 和最小化求解器（例如【22, 34】）来估计姿态。对于非标定相机，存在各种最小化求解器（例如【10】）或基于采样的方法（例如【31】）。在第 4.2 节中，我们提出了一种新颖且鲁棒的下一个最佳图像选择方法，用于准确的姿态估计和可靠的三角化。

**三角测量**。新注册的图像必须观测到已有的场景点。此外，通过三角测量，它还可以通过扩展点集 $\mathcal{X}$ 来增加场景覆盖范围。只要至少有另一个图像也覆盖新的场景部分但从不同的视角注册，就可以将新的场景点 $\mathrm{X}_k$ 进行三角测量并添加到 $\mathcal{X}$ 中。三角测量是 SfM 中的关键步骤，它通过冗余性【58】增加了现有模型的稳定性，并通过提供额外的 2D-3D 对应关系来实现新图像的注册。存在许多多视图三角测量的方法 【27, 5, 25, 35, 40, 3, 44, 32】。这些方法在 SfM 中受到限制的鲁棒性或高计算成本的影响，我们在第 4.3 节中提出了一种鲁棒且高效的三角测量方法来解决这个问题。

**光束法平差**。图像配准和三角测量是独立的过程，尽管它们的结果高度相关 - 相机姿态的不确定性会传播到三角化的点，反之亦然，并且通过额外的三角化，可以通过增加冗余性来改善初始相机姿态。如果没有进一步的优化，SfM 通常会迅速漂移到不可恢复的状态。光束法平差【58】是对相机参数 $\mathrm{P}_c$ 和点参数 $\mathrm{X}_k$ 进行联合非线性优化的过程，通过最小化重投影误差来实现，其中使用函数 $π$ 将场景点投影到图像空间，同时使用损失函数 $\rho_j$ 来减弱离群值的影响。

$$E=\sum_j\rho_j(\|\pi(\mathrm{P}_c,\mathrm{X}_k)-\mathrm{x}_j\|^2_2))$$

*Levenberg-Marquardt* 方法【58，26】是解决光束法平差问题的方法。光束法平差问题中的参数结构特殊，可利用 *舒尔补* 技巧【8】，即先解算简化的相机参数，然后通过反代法更新点的参数。这种方案通常更高效，因为相机的数量通常比点的数量少。解算方程组有两种选择：精确解法和非精确的迭代算法。精确方法通过将方程组存储和分解为稠密或稀疏矩阵【13, 38】来解决，其空间复杂度为 $\mathcal{O}(N^2_P)$，时间复杂度为 $\mathcal{O}(N^3_P)$ 。非精确方法通常通过使用迭代方法（例如 *预条件共轭梯度法* ，PCG）来近似解算方程组，其时间和空间复杂度为 $\mathcal{O}(N_P)$ 【4, 63】。直接算法适用于少数几百个相机的情况，但在大规模场景中成本过高。虽然稀疏直接方法可以大大减少稀疏问题的复杂度，但对于大规模非结构化的照片集合来说，由于连接图通常更密集，它们的使用是不可行的【54, 4】。在这种情况下，间接算法是首选方法。特别是对于互联网照片，光束法平差花费了大量时间来优化许多近似重复的图像。在第 4.5 节中，我们提出了一种方法来识别和参数化高度重叠的图像，以实现对密集图像集合的高效光束法平差。

## 3 挑战

虽然目前最先进的 SfM 算法可以处理大规模互联网照片集合中多样且复杂的图像分布，但在完整性和稳健性方面，它们经常无法产生令人满意的结果。通常情况下，系统无法注册大部分应该可以注册的图像【20, 30】，或者由于错误的注册或漂移而产生破碎的模型。首先，这可能是由于重叠图像搜索产生了不完整的场景图，例如由于匹配近似，因此既无法提供完整模型所需的连通性，也无法提供可靠估计所需的足够冗余。其次，这可能是由于重建阶段由于缺失或不准确的场景结构而无法注册图像 - 图像配准和三角测量之间存在一种共生关系，即图像只能注册到现有的场景结构，而场景结构只能通过注册的图像进行三角测量【64】。在增量重建过程中，在每个步骤中最大化两者的准确性和完整性是SfM中的一个关键挑战。在本文中，我们解决了这一挑战，并在完整性方面显著改善了当前技术水平的结果（第 5 节）。

## 4 贡献

本节介绍了一种改进 SfM 中主要挑战的新算法。首先，我们引入了一种几何验证策略，通过增加场景图的信息来改善初始化和三角化组件的稳健性。其次，我们提出了一种最佳视角选择方法，以最大化增量重建过程的稳健性和准确性。第三，我们提出了一种鲁棒的三角化方法，以较低的计算成本产生比现有技术更完整的场景结构。第四，我们采用迭代的 BA（Bundle Adjustment）方法、重新三角化和异常值过滤策略，通过减轻漂移效应，显著提高了完整性和准确性。最后，我们提出了一种更高效的 BA参数化方法，适用于密集的照片集合，通过冗余视图挖掘。这样就得到了一个在稳健性和完整性方面明显优于当前技术水平的系统，同时保持了其效率。我们将我们的贡献与当前最先进的系统 *Bundler*（开源）【52】和 *VisualSFM*（闭源）【62】进行了对比。所提出的系统以开源实现的形式发布。

### 4.1 场景图增强

我们提出了一种多模型几何验证策略，通过适当的几何关系来增强场景图。首先，我们估计基础矩阵。如果至少有 $N_F$ 个内点被找到，我们将图像对视为几何验证通过。接下来，我们通过确定同一图像对的单应矩阵内点数 $N_H$ 来对变换进行分类。为了近似 GRIC 等模型选择方法，我们假设如果 $N_H/N_F <\epsilon_{HF}$，则在一般场景中存在移动相机。对于校准图像，我们还估计本质矩阵及其内点数$N_E$。如果 $N_E/N_F >\epsilon_{EF}$，则假设校准正确。在校准正确且 $N_H/N_F <\epsilon_{HF}$ 的情况下，我们对本质矩阵进行分解，通过内点对三角化点，并确定中位数三角化角度 $\alpha_m$。利用 $\alpha_m$，我们区分纯旋转（全景）和平面场景（平面）的情况。此外，互联网照片经常出现水印、时间戳和边框（WTFs）【60, 30】，它们错误地链接了不同位置的图像。我们通过在图像边界估计具有 $N_S$ 个内点的相似变换来检测此类图像对。任何 $N_S/N_F >\epsilon_{SF} \lor N_S/N_E >\epsilon_{SE}$ 的图像对都被视为 WTF，并不插入到场景图中。对于有效的图像对，我们在具有最大支持 $N_H$、$N_E$ 或 $N_F$ 的模型的内点旁标记场景图的模型类型（一般、全景、平面）。模型类型用于仅从非全景且最好是校准的图像对开始重建过程。已经增强的场景图使得能够高效地找到一个优化的初始化，用于稳健的重建过程。此外，我们不从全景图像对进行三角化，以避免退化点，从而提高三角化和随后的图像注册的稳健性。

### 4.2 最佳下一视图的选择

在计算机视觉、摄影测量和机器人领域中，已经研究了最佳下一视图规划【12】。在稳健的 SfM 中选择最佳下一视图的目标是最小化重建误差【17, 24】。在这里，我们提出了一种高效的下一视图策略，采用基于不确定性的方法，以最大化重建的稳健性。

选择下一视图非常关键，因为每个决策都会影响剩余的重建过程。一个错误的决策可能导致一系列相机误注册和错误的三角化。此外，选择下一视图极大地影响姿态估计的质量以及三角化的完整性和准确性。准确的姿态估计对于稳健的 SfM 至关重要，因为如果姿态不准确，点的三角化可能会失败。选择下一视图的决策具有挑战性，因为对于互联网照片集合，通常没有有关场景覆盖和相机参数的先验信息，因此决策完全基于从外观【17】、两视图对应和增量重建的场景推导的信息【53, 24】。

一种常用的策略是选择看到最多三角化点的图像【52】，以最小化相机复位的不确定性。Haner 等人【24】提出了一种以不确定性为驱动的方法，最小化重建误差。通常情况下，选择看到最多三角化点的相机，除非观测配置不良。为此，Lepetit 等人【34】通过实验证明，使用 PnP 求解相机姿态的准确性取决于观测数量及其在图像中的分布。对于互联网照片，标准的 PnP 问题在先验标定缺少或不准确的情况下被扩展到估计内参数。大量的 2D-3D 对应关系为该估计提供了冗余性【34】，而点的均匀分布避免了糟糕的配置，并实现了内参的可靠估计【41】。

下一个最佳视角的候选图像是尚未注册的看到至少 $N_t > 0$ 个三角化点的图像。通过使用特征轨迹图，可以有效地实现对此统计数据的跟踪。对于互联网数据集，由于许多图像可能看到相同的结构，因此该图可以非常密集。因此，在重建的每一步中，有许多候选视角可供选择。Haner 等人提出的详尽的协方差传播方法是不可行的，因为需要为每个候选视角计算和分析协方差。我们提出的方法使用高效的多分辨率分析来近似他们基于不确定性的方法。

我们必须同时跟踪每个候选图像中可见点的数量和分布。可见点更多且更均匀地分布应导致更高的得分 $\mathcal{S}$【31】，这样具有更好条件的可见点配置的图像将首先进行注册。为实现这一目标，我们将图像离散化为一个固定大小的网格，每个维度有 $K_l$ 个箱子。每个单元格有两种不同的状态：*空* 和 *满*。在重建过程中，当一个点在一个 *空* 单元格内变为可见时，该单元格的状态变为 *满*，并且该图像的得分 $\mathcal{S}_i$ 增加权重 $w_l$。通过这种方案，我们量化可见点的数量。由于单元格只对总得分有贡献一次，我们更喜欢点的均匀分布，而不是点集聚在图像的一个部分（即只有少数单元格包含所有可见点）的情况。然而，如果可见点的数量为 $N_t\ll K^2_l$，这种方案可能无法很好地捕捉到点的分布，因为每个点很可能落入一个单独的单元格。因此，我们将前面描述的方法扩展到具有 $l=1\dots L$ 级的多分辨率金字塔，通过在每个相继的级别使用更高的分辨率 $K_l =2^l$ 来对图像进行分割。得分在所有级别上累积，使用分辨率相关的权重 $w_l = K^2_l$。这种数据结构及其得分可以在线上高效地更新。图 3 显示了不同配置的得分，第 5 节展示了使用该策略改善的重建的稳健性和准确性。

### 4.3 鲁棒和高效的三角测量

特别是对于匹配稀疏的图像集合，利用传递性对应可以提升三角测量的完整性和准确性，从而改善后续的图像注册。近似匹配技术通常偏向于外观相似的图像对，因此两个视图之间的对应通常来自具有较小基线的图像对。利用传递性可以建立基线较大的图像之间的对应关系，从而实现更准确的三角测量。因此，我们通过连接两个视图之间的对应关系来形成特征轨迹。

已经提出了多视图从噪声图像观测中进行三角测量的各种方法【27，40，5】。尽管某些提出的方法对一定程度的异常值污染具有鲁棒性【25，35，3，44，32】，但据我们所知，这些方法中没有一种能够处理特征轨迹中经常出现的高异常值比例（图6）。请参考 Kang 等人的工作【32】，了解现有多视图三角测量方法的详细概述。在本节中，我们提出了一种高效的基于采样的三角测量方法，可以在存在异常值的特征轨迹中鲁棒地估计所有点。

由于模糊匹配沿着极线的两个视图的验证可能会产生错误的结果，特征轨迹通常包含大量异常值。单个不匹配会合并两个或多个独立点的轨迹。例如，将长度相等的四个特征轨迹错误地合并，结果会产生 75% 的异常值比例。此外，由于不准确的相机姿态可能导致轨迹元素的重投影误差较大，因此在进行多视图细化之前，需要找到一组一致的轨迹元素。此外，为了从错误合并中恢复可能的多个点的特征轨迹，需要使用递归三角测量方案。

*Bundler* 对所有轨迹元素的两两组合进行采样，进行双视图三角测量，然后检查是否至少有一个解具有足够的三角测量角度。如果找到了一个良好条件的解，则对整个轨迹进行多视图三角测量，并且只有当所有观测值都满足视差约束时，才接受该解【26】。这种方法对异常值不具有鲁棒性，因为无法恢复合并到一个轨迹中的独立点。此外，由于需要进行穷举的两两三角测量，计算成本较高。我们的方法克服了这两个限制。

为了处理任意水平的异常值污染，我们使用 RANSAC 方法来进行多视图三角测量问题的建模。我们将特征轨迹 $\mathcal{T} = \{Tn ~|~ n = 1\dots N_T\}$ 视为一组具有先验未知比例 $\epsilon$ 的测量。一个测量 $T_n$ 包括归一化的图像观测 $\bar{\mathrm{x}}_n \in \mathbb{R}^2$ 和相应的相机姿态 $\mathrm{P}_n \in \mathrm{SE}(3)$，其中相机从世界坐标系到相机坐标系的投影由$\mathrm{P} = \begin{bmatrix}\mathrm{R}^T &−\mathrm{R^T}\mathrm{t}\end{bmatrix}$ 定义，其中 $\mathrm{R} \in \mathrm{SO}(3)$ ，$\mathrm{t} \in \mathbb{R}^3$。我们的目标是最大化符合良好条件的双视图三角测量的测量支持。 

$$\mathrm{X}_{ab}\sim \tau(\bar{\mathrm{x}}_a,\bar{\mathrm{ x}}_b,\mathrm{P}_a,\mathrm{P}_b)~\text{with}~a\neq b$$

其中，$\tau$ 是任意选择的三角测量方法（在我们的情况下是 DLT 方法【26】），而 $\mathrm{X}_{ab}$ 是三角测量得到的点。需要注意的是，我们不从全景图像对进行三角测量（第4.1节），以避免由于姿态估计不准确而导致的错误三角测量角度。一个良好条件的模型满足两个约束条件。首先，存在足够的三角测量角度 $\alpha$，满足 

$$\cos\alpha=\dfrac{\mathrm{t}_a-\mathrm{X}_{ab}}{\|\mathrm{t}_a-\mathrm{X}_{ab}\|_2}\cdot\dfrac{\mathrm{t}_b-\mathrm{X}_{ab}}{\|\mathrm{t}_b-\mathrm{X}_{ab}\|_2}$$

其次，对于视图 $\mathrm{P}_a$ 和 $\mathrm{P}_b$，具有正的深度 $d_a$ 和 $d_b$（视差约束），其中深度定义为

$$d = \begin{bmatrix}p31 & p32 &p33 &p34 \end{bmatrix}\begin{bmatrix}\mathrm{X}^T_{ab} &1\end{bmatrix}^T$$

其中 $p_{mn}$ 表示 $\mathrm{P}$ 的第 $m$ 行第 $n$ 列的元素。如果测量 $T_n$ 满足正的深度 $d_n$，并且其投影误差 

$$e_n = \|\bar{\mathrm{x}}_n − \begin{bmatrix}x^\prime/z^\prime \\y^\prime/z^\prime  \end{bmatrix}\|~\text{with}~\begin{bmatrix}x^\prime\\y^\prime \\z^\prime\end{bmatrix}=\mathrm{P}\begin{bmatrix}\mathrm{X}_{ad}\\1\end{bmatrix}$$

小于某个阈值 $t$，则认为该测量 $T_n$ 符合模型。RANSAC 作为一个迭代方法，通过最大化 $\mathcal{K}$ 进行求解，并且通常会随机均匀地对最小的两个样本进行采样。然而，由于对于小的 $N_T$，可能会多次采样相同的最小集合，因此我们的随机采样器只生成唯一的样本，以确保具有置信度 $\eta$ 的至少采样到一个无异常值的最小集合，RANSAC 必须运行至少 $K$ 次迭代。由于先验的内点比例未知，我们将其设置为较小的初始值 $\epsilon 0$，并在找到更大的共识集合时调整 $K$（自适应停止准则）。由于一个特征轨迹可能包含多个独立的点，我们通过从剩余的测量中删除共识集合来递归地运行此过程。当最新的共识集合的大小小于三时，递归停止。在第 5 节的评估中，我们证明了该方法在降低计算成本的同时增加了三角测量的完整性。

### 4.4 光束法平差

为了减轻积累误差，我们在图像配准和三角测量之后进行 BA（光束法平差）。通常，在每个步骤之后都没有必要进行全局 BA，因为增量式 SfM 只对模型进行局部影响。因此，在每次图像配准之后，我们对最连接的图像集合进行局部 BA。类似于 VisualSFM，我们只在模型增长一定百分比之后进行全局 BA，从而实现 SfM 的线性摊销运行时间。

**参数化**。为了考虑潜在的异常值，我们在局部 BA 中使用 Cauchy 函数作为鲁棒损失函数 $\rho_j$。对于少量相机的问题，我们使用稀疏直接解算方法，对于较大的问题，我们依赖于 PCG。我们使用 Ceres Solver【2】，并提供将任意复杂性的相机模型在任意图像组合之间共享的选项。对于无序的互联网照片，我们依赖于一个简单的相机模型，其中只有一个径向畸变参数，因为估计是基于纯自标定的。

**滤除**。在 BA 之后，一些观测结果不符合模型。因此，我们过滤掉具有较大投影误差的观测结果【53, 62】。此外，对于每个点，我们通过强制所有视线对的三角测量角度最小来检查其几何条件是否良好【53】。在全局 BA 之后，我们还检查退化的相机，例如由全景图像或人工增强图像引起的退化。通常，这些相机只有异常值观测结果，或者其内参数收敛到一个虚假的最小值。因此，我们不将焦距和畸变参数限制在先验固定范围内，而是允许它们在 BA 中自由优化。由于主点标定是一个不适定的问题【15】，我们将其固定在未标定相机的图像中心。在全局 BA 之后，将具有异常视场或大畸变系数幅值的相机视为错误估计，并进行过滤。

**重新三角化**。类似于 VisualSfM，我们在全局 BA 之前进行重新三角化（pre-BA RT，光束法平差前三角化）以考虑漂移效应。然而，BA 通常会显著改善相机和点参数。因此，我们建议在非常有效的 pre-BA RT 步骤之后，再增加一个额外的 post-BA RT（光束法平差后三角化）步骤。此步骤的目的是通过继续无法进行三角测量的点的轨迹（例如由于姿态不准确等原因）来改善重建的完整性（参见第 4.3 节）。我们只继续那些其误差低于滤除阈值的观测结果的轨迹。此外，我们尝试合并轨迹，从而为下一步 BA 提供增加的冗余性。

**迭代细化**。Bundle r和 VisualSfM 仅执行单个 BA 和滤除步骤。由于漂移或 pre-BA RT，通常在 BA 中的观测结果中有相当大的部分是异常值，并且随后被过滤。由于 BA 严重受到异常值的影响，第二步 BA 可以显著改善结果。因此，我们建议在迭代执行 BA、RT 和滤除，直到被过滤的观测结果数量和 post-BA RT 点数量减少为止。在大多数情况下，经过第二次迭代后，结果显著改善且优化收敛。第 5 节证明了提出的迭代细化方法显著提高了重建的完整性。

### 4.5 冗余视图挖掘
光束法平差在 SfM 中是一个主要的性能瓶颈。在本节中，我们提出了一种方法，利用增量 SfM 和密集照片集合的固有特性，通过将冗余相机聚类成高场景重叠的组，以更高效地对 BA 进行参数化。

由于兴趣点的受欢迎程度不同，互联网照片集合通常具有高度不均匀的可见性模式。此外，无序集合通常被聚类成在许多图像中共视的点片段。之前的一些工作利用了这一事实来提高 BA 的效率，包括 Kushal 等人【33】通过分析可见性模式以便于对减少的相机系统进行有效预处理。Ni 等人【43】将相机和点分成子图，并通过在相机和点参数的图上提出分割问题来连接这些子图，然后 BA 在固定相机和点以及细化分割变量之间交替进行。Carlone 等人【11】的另一种方法是将多个低秩点合并为一个因子，并对相机施加高秩约束，从而在相机共享多个点时提供计算上的优势。

我们的方法受到这些先前工作的启发。与 Ni 等人类似，我们将问题划分为子图，其中内部参数被因子化。我们有三个主要贡献：首先，一种有效的相机分组方案，利用 SfM 的固有特性，并替代 Ni 等人使用的昂贵的图割方法。其次，我们将许多相机聚类为许多小的、高度重叠的相机组，而不是将许多相机聚类为一个子图，从而降低了解决减少的相机系统的成本。第三，由于较小且高度重叠的相机组，我们通过跳过 Ni 等人的分割变量优化来消除了交替方案。

SfM 根据它们的参数是否受到最新增量模型扩展的影响，将图像和点分成两组。对于大型问题，由于模型通常只在局部扩展，大部分场景保持不变。BA 自然更多地优化新扩展的部分，而其他部分只在漂移情况下得到改善【62】。此外，互联网照片集合通常具有不均匀的相机分布，存在许多冗余视点。基于这些观察，我们将未受影响的场景部分划分为高度重叠的图像组 $\mathcal{G}=\{G_r~|~r = 1\dots N_G\}$，并将每个组 $G_r$ 参数化为单个相机。受最新扩展影响的图像被独立分组，以便对其参数进行最优化。请注意，这导致了标准的 BA 参数化（Eq. 1）。对于未受影响的图像，我们创建具有大小为 $N_{G_r}$ 的组。我们认为如果图像在最新模型扩展期间被添加，或者其超过比率 $\epsilon_r$ 的观测具有大于 $r$ 个像素的重投影误差（用于改进重新三角化的相机），则该图像受到影响。

组内的图像应尽可能冗余【43】，图像之间共视点的数量是描述它们相互作用程度的度量【33】。对于具有 $N_X$ 个点的场景，每个图像可以用一个二进制可见性向量 $v_i\in\{0,1\}^{N_X}$  来描述，其中 $v_i$ 的第 $n$ 个条目在点 $\mathrm{X_n}$ 在图像 $i$ 中可见时为 1，否则为 0。图像 $a$ 和 $b$ 之间的相互作用程度通过对它们的向量 $v_i$ 进行位操作来计算。

$$V_{ab}=\|\mathrm{v}_a\land\mathrm{v}_b\|/\|\mathrm{v}_a\lor\mathrm{v}_b\|$$

为了构建组，我们将图像按照 $\bar{\mathcal{I}}=\{I_i~|~\|v_i\|\geq\|v_{i+1}\|\}$ 进行排序。我们通过从 $\bar{\mathcal{I}}$ 中删除第一个图像$I_a$，并找到最大化 $V_{ab}$ 的图像 $I_b$ 来初始化一个组 $G_r$。如果 $V_{ab} > V$ 并且 $\vert G_r\vert < S$，则将图像 $I_b$ 从 $\bar{\mathcal{I}}$ 中移除并添加到组 $G_r$ 中。否则，初始化一个新的组。为了减少查找 $I_b$ 的时间，我们采用启发式方法，将搜索限制在 $K_r$ 个空间上最近邻中，其具有 $\pm\beta$ 度范围内的共视方向，这是由于这些图像具有共享许多点的高可能性。

然后，组内的每个图像相对于一个共享的组局部坐标系进行参数化。对于组合图像的光束法平差代价函数为

$$E_g=\sum_j\rho_j=\left(\|\pi_g(\mathrm{G}_r,\mathrm{P}_c,\mathrm{X}_k)-\mathrm{x}_j\|^2_2\right)$$

使用外参组参数 $\mathrm{G}_r\in \mathrm{SE}(3)$ 和固定的 $\mathrm{P}_c$。然后，组中图像的投影矩阵定义为组和图像姿态的乘积 $\mathrm{P}_{cr} = \mathrm{P}_c\mathrm{G}_r$。总体代价 $\bar{E}$ 是分组和未分组代价的总和贡献。为了有效地串联 $\mathrm{G}_r$ 和 $\mathrm{P}_i$ 的旋转分量，我们使用四元数。较大的组大小会带来更大的性能优势，因为计算 $\pi_g$ 相对于 $\pi$ 的开销更小。请注意，即使对于两个图像的组大小，我们也观察到计算上的优势。此外，性能优势取决于问题的大小，因为摄像机数量的减少对直接方法的立方计算复杂度影响大于对间接方法的线性计算复杂度（第2节）。

## 5 实验

我们在各种各样的数据集上进行实验，评估了所提出的组件和整个系统与现有技术的增量式（*Bundler* 【53】, *VisualSFM*【62】）和全局 SfM 系统（*DISCO* 【14】，*Theia* 【55】）的比较。这17个数据集共包含 144,953 张无序的互联网照片，分布在一个大范围内，并具有高度变化的相机密度。此外，Quad 【14】具有地面真实相机位置。在所有实验中，我们使用 RootSIFT 特征，并使用在不相关数据集上训练的词汇树将每个图像与其 100 个最近邻进行匹配。为了确保不同方法之间的可比性，计时不包括对 2.7GHz 主频、256GB RAM 的机器进行的重叠图像搜索。

**下一最佳视角选择**。一个合成实验（图 4）评估了分数 $\mathcal{S}$ 如何反映点的数量和分布。我们使用  $L = 6$ 的金字塔级别，并生成具有标准差 $\sigma$ 和均值 $\mu$ 的高斯分布图像点。较大的 $\sigma$ 和较中心的 $\mu$ 对应于更均匀的分布，并且正确产生较高的分数。同样，当点的数量较少时，分数主要由它们在图像中的分布决定。另一个实验（图 5）将我们的方法（*Pyramid*）与现有策略进行了重建误差的比较。其他方法是 *Number* 【53】，它最大化了三角化点的数量，和 *Ratio*，它最大化了可见点与可能可见点的比率。在每次图像配准之后，我们测量不同策略之间共享的注册图像数（交并比）和重建误差（相对于地面真实相机位置的中位距离）。虽然所有策略都会收敛到相同的注册图像集，但我们的方法通过为图像选择更好的配准顺序，产生了最准确的重建结果。

**鲁棒高效的三角化**。在 Dubrovnik 数据集上进行的实验（图 6 和表 2）评估了我们的方法在由 47M 个经过验证的匹配组成的 2.9M 个特征轨迹上的表现。我们与 *Bundler* 和一种穷举策略进行比较，该策略在轨迹中采样所有两两组合。我们设置 $\alpha=2°$，$t =8\mathrm{px}$，$\epsilon_0 =0.03$。为了避免组合爆炸，我们将穷举方法限制在 10K 次迭代，即 $\epsilon_{min} \approx 0.02$，$\eta =0.999$。通过递归穷举方法确定的多样性内点比例分布证明了对于鲁棒的三角化方法的需求。我们提出的递归方法恢复了比非递归方法更长的轨迹和更多的轨迹元素。请注意，递归 RANSAC 方法得到的更多点对应于稍微较短的轨迹长度。RANSAC 方法产生的轨迹略逊于非递归方法，但速度更快（10-40 倍）。通过变化 $\eta$，可以很容易地在速度和完整性之间进行平衡。

**冗余视图挖掘**。我们在一个无序的密集图像集合上评估冗余视图挖掘。图8展示了在使用固定数量的 BA 迭代的全局 BA 中，参数化相机的增长率。根据强制的场景重叠度 $V$，我们可以显著减少解决缩减相机系统的时间。总运行时间的有效加速比分别为 5%（$V =0.6$），14%（$V =0.3$）和32%（$V =0.1$），而平均重投影误差从 0.26 像素（标准BA）分别降低到 0.27 像素，0.28 像素和 0.29 像素。在所有 $V>0.3$ 的选择中，重建质量是可比较的，并且对于较小的 $V$，重建质量逐渐降低。使用 $V =0.4$，Colosseum 的整个流程运行时间减少了36%，但结果仍然是一个等效的重建。

**系统**。表 1 和图 1 展示了整个系统的性能的表现，并评估了系统中各个提出的组件的性能。对于每个数据集，我们报告最大的重建组件。*Theia* 是最快的方法，而我们的方法的计时略差于 *VisualSFM*，并且比 *Bundler* 快 50 多倍。图7显示了各个模块的相对计时。对于所有数据集，我们在完整性方面显著优于其他任何方法，特别是对于较大的模型。重要的是，增加的轨迹长度导致 BA 中的更高冗余度。此外，我们在Quad 数据集的姿态精度方面取得最佳结果：*DISCO* 1.16m，*Bundler* 1.01m，*VisualSFM* 0.89m 和我们的方法 0.85m。图 9 显示了 *Bundler* 与我们方法的结果对比。我们鼓励读者查看补充材料，了解结果的其他视觉对比，展示了我们方法的优越鲁棒性、完整性和准确性。

## 6 总结

本文提出了一种 SfM 算法，克服了关键挑战，迈向通用的 SfM 系统。算法的提出组件在完整性、鲁棒性、精度和效率方面改进了现有技术的水平。针对具有挑战性的大规模数据集进行了全面的实验，展示了各个组件和整个系统的性能。整个算法以开源实现的形式向公众发布。

**致谢**。我们感谢 J. Heinly 和 T. Price 的校对工作。我们还要感谢 C. Sweeney 为 *Theia* 实验的制作。本研究部分得到了 NSF No. IIS1349074、No. CNS-1405847 和 MITRE 公司的支持。


## 参考文献

??? info "References"

	[1] S. Agarwal, Y. Furukawa, N. Snavely, I. Simon, B. Curless, S. Seitz, and R. Szeliski. "Building Rome in a Day." ICCV, 2009.

	[2] S. Agarwal, K. Mierle, and Others. "Ceres Solver." http://ceres-solver.org.

	[3] S. Agarwal, N. Snavely, and S. Seitz. "Fast algorithms for L∞ problems in multiview geometry." CVPR, 2008.

	[4] S. Agarwal, N. Snavely, S. Seitz, and R. Szeliski. "Bundle adjustment in the large." ECCV, 2010.

	[5] C. Aholt, S. Agarwal, and R. Thomas. "A QCQP Approach to Triangulation." ECCV, 2012.

	[6] P. Beardsley, P. Torr, and A. Zisserman. "3D model acquisition from extended image sequences." 1996.

	[7] C. Beder and R. Steffen. "Determining an initial image pair for fixing the scale of a 3D reconstruction from an image sequence." Pattern Recognition, 2006.

	[8] D. C. Brown. "A solution to the general problem of multiple station analytical stereotriangulation." 1958.

	[9] M. Brown, G. Hua, and S. Winder. "Discriminative learning of local image descriptors." IEEE PAMI, 2011.

	[10] M. Bujnak, Z. Kukelova, and T. Pajdla. "A general solution to the P4P problem for a camera with an unknown focal length." CVPR, 2008.

	[11] L. Carlone, P. Fernandez Alcantarilla, H.-P. Chiu, Z. Kira, and F. Dellaert. "Mining structure fragments for smart bundle adjustment." BMVC, 2014.

	[12] S. Chen, Y. F. Li, J. Zhang, and W. Wang. "Active Sensor Planning for Multiview Vision Tasks." 2008.

	[13] Y. Chen, T. A. Davis, W. W. Hager, and S. Rajamanickam. "Algorithm 887: Cholmod, supernodal sparse Cholesky factorization and update/downdate." ACM TOMS, 2008.

	[14] D. Crandall, A. Owens, N. Snavely, and D. P. Huttenlocher. "Discrete-Continuous Optimization for Large-Scale Structure from Motion." CVPR, 2011.

	[15] L. de Agapito, E. Hayman, and I. Reid. "Self-calibration of a rotating camera with varying intrinsic parameters." BMVC, 1998.

	[16] F. Dellaert, S. Seitz, C. E. Thorpe, and S. Thrun. "Structure from motion without correspondence." CVPR.

	[17] E. Dunn and J.-M. Frahm. "Next best view planning for active model improvement." BMVC, 2009.

	[18] M. A. Fischler and R. C. Bolles. "Random sample consensus: a paradigm for model fitting with applications to image analysis and automated cartography." ACM, 1981.

	[19] A. Fitzgibbon and A. Zisserman. "Automatic camera recovery for closed or open image sequences." ECCV, 1998.

	[20] J.-M. Frahm, P. Fite-Georgel, D. Gallup, T. Johnson, R. Raguram, C. Wu, Y.-H. Jen, E. Dunn, B. Clipp, S. Lazebnik, and M. Pollefeys. "Building Rome on a Cloudless Day." ECCV, 2010.

	[21] J.-M. Frahm and M. Pollefeys. "RANSAC for (quasi-) degenerate data (QDEGSAC)." CVPR, 2006.

	[22] X.-S. Gao, X.-R. Hou, J. Tang, and H.-F. Cheng. "Complete solution classification for the perspective-three-point problem." IEEE PAMI, 2003.

	[23] R. Gherardi, M. Farenzena, and A. Fusiello. "Improving the efficiency of hierarchical structure-and-motion." CVPR, 2010.

	[24] S. Haner and A. Heyden. "Covariance propagation and next best view planning for 3D reconstruction." ECCV, 2012.

	[25] R. Hartley and F. Schaffalitzky. "L∞ minimization in geometric reconstruction problems." CVPR, 2004.

	[26] R. Hartley and A. Zisserman. "Multiple view geometry in computer vision." 2003.

	[27] R. I. Hartley and P. Sturm. "Triangulation." 1997.

	[28] M. Havlena and K. Schindler. "Vocmatch: Efficient multiview correspondence for structure from motion." ECCV, 2014.

	[29] J. Heinly, E. Dunn, and J.-M. Frahm. "Comparative evaluation of binary features." ECCV.

	[30] J. Heinly, J. L. Schönberger, E. Dunn, and J.-M. Frahm. "Reconstructing the World* in Six Days *(As Captured by the Yahoo 100 Million Image Dataset)." CVPR, 2015.

	[31] A. Irschara, C. Zach, J.-M. Frahm, and H. Bischof. "From structure-from-motion point clouds to fast location recognition." CVPR, 2009.

	[32] L. Kang, L. Wu, and Y.-H. Yang. "Robust multi-view L2 triangulation via optimal inlier selection and 3D structure refinement." Pattern Recognition, 2014.

	[33] A. Kushal and S. Agarwal. "Visibility based preconditioning for bundle adjustment." CVPR, 2012.

	[34] V. Lepetit, F. Moreno-Noguer, and P. Fua. "EPnP: An accurate O(n) solution to the PnP problem." IJCV, 2009.

	[35] H. Li. "A practical algorithm for L∞ triangulation with outliers." CVPR, 2007.

	[36] Y. Li, N. Snavely, and D. P. Huttenlocher. "Location recognition using prioritized feature matching." ECCV, 2010.

	[37] Y. Lou, N. Snavely, and J. Gehrke. "MatchMiner: Efficient Spanning Structure Mining in Large Image Collections." ECCV, 2012.

	[38] M. I. Lourakis and A. A. Argyros. "SBA: A software package for generic sparse bundle adjustment." ACM TOMS, 2009.

	[39] D. G. Lowe. "Distinctive image features from scale-invariant keypoints." IJCV, 2004.

	[40] F. Lu and R. Hartley. "A fast optimal algorithm for L2 triangulation." ACCV, 2007.

	[41] C. McGlone, E. Mikhail, and J. Bethel. "Manual of photogrammetry." 1980.

	[42] R. Mohr, L. Quan, and F. Veillon. "Relative 3D reconstruction using multiple uncalibrated images." IJR, 1995.

	[43] K. Ni, D. Steedly, and F. Dellaert. "Out-of-core bundle adjustment for large-scale 3D reconstruction." ICCV, 2007.

	[44] C. Olsson, A. Eriksson, and R. Hartley. "Outlier removal using duality." CVPR, 2010.

	[45] M. Pollefeys, D. Nister, J.-M. Frahm, A. Akbarzadeh, P. Mordohai, B. Clipp, C. Engels, D. Gallup, S.-J. Kim, P. Merrell, et al. "Detailed real-time urban 3D reconstruction from video." IJCV, 2008.

	[46] M. Pollefeys, L. Van Gool, M. Vergauwen, F. Verbiest, K. Cornelis, J. Tops, and R. Koch. "Visual modeling with a hand-held camera." IJCV, 2004.

	[47] F. Schaffalitzky and A. Zisserman. "Multi-view matching for unordered image sets, or How do I organize my holiday snaps?" ECCV, 2002.

	[48] J. L. Schönberger, A. C. Berg, and J.-M. Frahm. "Efficient two-view geometry classification." GCPR, 2015.

	[49] J. L. Schönberger, A. C. Berg, and J.-M. Frahm. "PAIGE: PAirwise Image Geometry Encoding for Improved Efficiency in Structure-from-Motion." CVPR, 2015.

	[50] J. L. Schönberger, D. Ji, J.-M. Frahm, F. Radenović, O. Chum, and J. Matas. "From Dusk Till Dawn: Modeling in the Dark." CVPR, 2016.

	[51] J. L. Schönberger, F. Radenović, O. Chum, and J.-M. Frahm. "From Single Image Query to Detailed 3D Reconstruction." CVPR, 2015.

	[52] N. Snavely. "Scene reconstruction and visualization from internet photo collections." PhD thesis, 2008.

	[53] N. Snavely, S. Seitz, and R. Szeliski. "Photo tourism: exploring photo collections in 3D." ACM TOG, 2006.

	[54] N. Snavely, S. Seitz, and R. Szeliski. "Skeletal graphs for efficient structure from motion." CVPR, 2008.

	[55] C. Sweeney. "Theia multiview geometry library: Tutorial & reference." http://theia-sfm.org.

	[56] C. Sweeney, T. Sattler, T. Hollerer, M. Turk, and M. Pollefeys. "Optimizing the viewing graph for structure-from-motion." CVPR, 2015.

	[57] P. H. Torr. "An assessment of information criteria for motion model selection." CVPR, 1997.

	[58] B. Triggs, P. F. McLauchlan, R. I. Hartley, and A. Fitzgibbon. "Bundle adjustment: a modern synthesis." 2000.

	[59] T. Tuytelaars and K. Mikolajczyk. "Local invariant feature detectors: a survey." CGV, 2008.

	[60] T. Weyand, C.-Y. Tsai, and B. Leibe. "Fixing wtfs: Detecting image matches caused by watermarks, timestamps, and frames in internet photos." WACV, 2015.

	[61] K. Wilson and N. Snavely. "Robust global translations with 1dsfm." ECCV, 2014.

	[62] C. Wu. "Towards linear-time incremental structure from motion." 3DV, 2013.

	[63] C. Wu, S. Agarwal, B. Curless, and S. Seitz. "Multicore bundle adjustment." CVPR, 2011.

	[64] E. Zheng and C. Wu. "Structure from motion using structureless resection." ICCV, 2015.
