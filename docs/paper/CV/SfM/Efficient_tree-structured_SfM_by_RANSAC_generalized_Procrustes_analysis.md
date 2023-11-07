---
title: Efficient tree-structured SfM by RANSAC generalized Procrustes analysis
status: new
---

# Efficient tree-structured SfM by RANSAC generalized Procrustes analysis

!!! info "论文链接"
	Yisong Chen, Antoni B. Chan, Zhouchen Lin, Kenji Suzuki, Guoping Wang [Efficient tree-structured SfM by RANSAC generalized Procrustes analysis](https://linkinghub.elsevier.com/retrieve/pii/S1077314217300334
	)

!!! warning "注意"
    在 chatgpt 辅助下翻译整理



## 1 引言

- 结构运动（Structure-from-motion，简称 SfM）技术已经变得流行，用于从非结构化和不受限制的图像集合中重建相机和3D场景。

- 主导 SfM 的方法有增量算法

	- 增量方法往往计算密集

		- 频繁使用捆绑调整以及去除不一致的测量
		- 可通过使用多核优化或将问题分割为更易处理的成分来缓解
		
	- 增量方法在视觉连接较弱的场景中容易产生漂移。

		- 使用分层重建
		- 组织分层聚类树并沿树合并部分重建，这些方法能够均匀分布错误，从而使它们对初始化和漂移更不敏感
		- 这种方案可以通过良好平衡的集群树减少一个数量级的计算复杂性
		- 分层方法中，必须注意避免由异常匹配引起的不良结构
		
- 另一种方法是全局优化

	- 方法

		- 使用两视图几何单独估计相机旋转。
		- 这些旋转被输入进一步的优化步骤，解决相机平移和结构
		
	- 全局姿态注册方法缺乏对噪声的内置鲁棒性，可能无法提供良好的初始化
	- 全局方法在应用于高度非结构化的数据时往往不稳定且不准确

- 噪声和计算效率是现代 SfM 方法的两个主要挑战

	- 不良的相机姿势和不匹配可能导致不良的原子重建，并且可能在小型数据集上显着偏向非线性优化。
	- 大量工作致力于加速关键 SfM 模块

		- 图像聚类
		- 特征匹配
		- 非线性优化

	- 从所有可用的匹配进行完全重建通常会受到高度非结构化数据集的干扰，并可能带来不必要的计算成本

		- 生成树结构对许多任务已经足够

- 本文提出了一种从无序图像构建生成树重建的新型 SfM 方法

	- 解决了鲁棒性和效率方面的挑战，能从高度非结构化的数据集中迅速恢复相机和场景结构
	- 过程

		- 计算一个包含所有图像并生成相关原子结构的生成树
		- 将结构分组成具有近似内部相似性的本地集群
		- 通过稳健地对齐同一组中的多个结构，使用广义 Procrustes 分析（GPA）的 RANSAC 扩展来构建更大的重建。
		- 分组-对齐过程迭代进行，直到实现完全的重建
		
- 关键贡献

	1. 提出 RANSAC 广义 Procrustes 分析方法，用于多个结构的对齐
		- 快速
		- 鲁棒性
	2. 设计一个用于组织无序图像和分组本地结构的浅层重建树
		- 实现快速可靠的3D重建

- 本文的方法具有分层解决方案的优点

	- 高计算效率
	- 对初始化和漂移的不敏感性。

- 我们的方法在以下几个方面优于最先进的 SfM 方法：

	1. 与分层方法相比，我们的方法具有更浅的树深度，速度更快
	2. 与全局优化方法相比，我们的方法对大相机旋转更具宽容性，避免了注册失败的问题
	3. 与增量方法相比，我们的分而治之策略更快，对漂移不太敏感，并且更容易并行化


## 2 基于 RANSAC 广义 Procrustes 分析的树形 SfM

- 一种基于 RANSAC 广义 Procrustes 分析（RGPA）的树形 SfM 算法

	- 能在存在适度噪声的情况下快速而鲁棒地恢复相机和3D场景
 
		1. 构建点 track 并进行像对重建
		2. 构建一个生成树，其包含跨越整个图像集最大连通成分的双视图重建。
		3. 生成树按自底向上的方式进行分组和合并
		
			- RGPA用于对齐和合并一组结构
			- 检测并删除异常值匹配
		
		4. 执行最终的捆绑调整，以细化重建的 3D 点和相机




### 2.1 Track 生成和像对重建

- 预处理步骤

	- 从 SIFT 特征建立 track
	- 使用 5 点算法估计相机姿势并三角测量点的坐标
	- 对于标定矩阵，使用标准相机模型

		- 主点 $(x_0,y_0)$ 位于图像中心
		- 焦距 $f$ 从相机的 EXIF 数据中提取

- 一个重要操作是检测并删除具有不正确的极线几何的不良图像对

	1. 删除具有少于 15个匹配的图像对
	2. 删除场景太相似的图像对（平面单应性测试的离群值比率 $r < 0.01$）
	3. 删除相机中心之间的距离 $d_1$ 与相机和重建点之间的中值距离 $d_2$ 之间的距离太小的图像对（$d_2 > 10d_1$）
	4. 检查所有两视图重建的三元组的旋转一致性，删除引起三元组不一致占所属三元组的 50 % 以上的影像对。



### 2.2 生成树构建

- 目标是从最可靠的两视图重建中构建生成树重建
 
	- 构建一个图，其中每个图像是一个顶点，每个有效的成对重建是一条边
	- 从此初始图的最大连接成分 $H$ 中提取生成树集 $\psi$ 。 $\psi$ 包含 $H$ 中的所有图像， $\psi$ 中的所有边对应于最可靠的两视图重建

- 生成树的优势

	1. 生成树通常涵盖场景的主体部分，并具有足够数量的特征，这在计算上比冗余的完全重建更有效
	2. 生成树中的原子重建是最可靠的，从而进一步减少了由于不良极线几何引起的失败风险。

- 先前的工作使用最大叶子生成树来定义生成树

	- 对于增量 SfM 效果很好，但对于分层 SfM 来说，可能导致初始结构之间的关系太弱。

- 提出一种用于构建强壮结构合并的生成树的方法

	- 启发式地收集具有更多有效匹配的两视图重建来计算图的生成树
	- 假设 $v_{i,j}$ 是图像对 $(i, j)$ 的有效匹配数
	- 在预处理步骤中删除了大多数不良图像对后，可合理地假设较大的 $v_{i,j}$ 对应于更好的重建
	- 共享更多匹配的本地结构通常具有更强的连接，会有更好的合并结果
	- 使用 $v_{i,j}$ 来定义边权重，并寻找总权重最大的生成树。
	- 在计算了最佳生成树 $\psi$ 之后，与 $\psi$ 的边关联的两视图重建成为后续合并步骤的原子结构，并充当重建树的叶子

- 考虑匹配的分布或其他语义信息可能会导致更高级技术的更好模型，但代价更高，是未来有趣的工作方向

### 2.3 通过 RGPA 进行自底向上的结构合并

- 每个重建树的叶子对应于一个两视图重建，并提供一组重建的 3D  点

	- 在全局范围内，这些 3D 点集中的任意两个都通过具有 7 个自由度的相似性变换相关联
	- 可使用一种精心设计的 3D 注册算法，通过它们的共同点将这些初始结构对齐

 - 本文使用广义 Procrustes 分析（GPA）的修改版本来进行注册任务。

#### 2.3.1 广义 Procrustes 分析

- 广义 Procrustes 分析

	- 用于形状对齐
	- 输入形状由 $n$ 个矩阵 $D_1,\dots,D_n$ 表示。每个形状 $D_i \in \mathbb{R}^{d\times m}$ 由 $m$ 个 $d$ 维点组成
		
		$$D_i =(D_{i,1},\dots,D_{i,m}),\quad D_{i,j} \in \mathbb{R}^d$$

	- 形状对齐问题是找到一组 $n$ 个相似性变换 $T_i:\mathbb{R}^d\rightarrow \mathbb{R}^d$ 和参考形状 $F =(F_1,\dots,F_m)\in \mathbb{R}^{d\times m}$ ，使得如下成本函数最小

		$$ \varepsilon(T, F) = \sum_{i=1}^{n} \sum_{j=1}^{m} \mu_{i, j} \|F_j - T_i(D_{i, j})\|_2^2 $$

	- 其中 $\mu_{i,j} \in \{0, 1\}$ 允许模型在不能在所有形状中观察到所有点时忽略缺失的数据
	- GPA 问题可以通过在 $T$ 和 $F$ 的估计之间交替进行求解

		- Procrustes 分析（PA）用于将所有形状逐个与当前参考 $F$ 对齐以解决 $T$，并且所有对齐的形状都叠加以更新 $F$
		- 当参考的变化足够小时，算法终止 

- 标准的 GPA 算法如下：
	
	1. 选择一个参考形状（通常是在可用实例中选择之一）
	2. 通过使用它们的共享点对进行 PA，将所有实例叠加到当前参考形状
	3. 计算当前叠加形状集的平均形状
	4. 如果平均形状和参考足够接近，则返回参考并终止算法，否则返回第 2 步

- 本文中，$n$ 是组中部分重建的数量，$m$ 是这些 $n$ 重建的所有重建 3D 点的总数

	- 考虑在3D空间中的非反射相似性（d = 3）
	- GPA 模型与标准 GPA 中的模型不同

		- 因子变量 $μ_{i, j}$ 不是固定的，而是根据实时更新的匹配的质量动态变化
		- 在迭代对齐和参考更新模块中，可靠点逐渐更新直到收敛
		- 通过跨多个形状进行交叉检查检测并删除虚假点。

#### 2.3.2 结构集合分组

- 给定一组部分重建的结构 $S_i,i = 1,\dots,s$，由重建的 3D 点集 $P_i$ 和相机集 $C_i$ 组成
	- 将这些结构分成 $g$ 组 $G_i,i = 1,\dots,g$（不一定是互斥的），每个组包含彼此相似的一簇结构。
	
	- 分组要求

		1. 为了可靠地合并局部结构，组中的所有结构应具有足够的到某个参考形状的匹配
		2. 为了完成重建，任何两个组都应通过具有在路径中相邻组之间足够强连接的路径连接
		3. 为了计算效率，组的总数应尽可能少

- 将结构集建模为一个图，其中每个结构充当一个顶点，共同点的数量确定连接两个顶点的权重。通过这个图，上述要求可以很好地描述为最小连接支配集（MCDS）问题
- 本文使用 MCDS 来帮助进行结构分组

	- 图 $\Gamma$ 的最小连接支配集是具有以下三个属性的顶点集合 $\Delta$
	
		1.  $\Gamma$ 中的每个顶点要么属于 $\Delta$，要么与 $\Delta$ 中的顶点相邻
		2.  $\Delta$ 中的任何顶点可以通过完全在 $\Delta$ 中的路径到达 $\Delta$ 中的任何其他顶点
		3.  $\Delta$ 在满足以上条件的所有 $\Gamma$ 的集合中具有最小的可能基数

	- MCDS 的 3 个属性与结构分组的 3 个要求完美相关。

- 采用了贪婪算法的变体来计算 MCDS 的近似值

	- 在贪婪搜索过程中，更高优先级地考虑了那些与邻居共享更多共同点的顶点
	- 算法如下
	
		1. 初始化：选择具有最大度的顶点，并将其着为灰色。将所有其他顶点着为白色。
		2. 选择具有 1) 最大度和 2) 与白色顶点的最强连接的灰色顶点 $v$，将其着为黑色。
		3. 将 $v$ 的白色邻居着为灰色。
		4. 返回第 2 步，直到所有顶点都是黑色或灰色。

	- 算法结束时，黑色顶点组成了最小连接支配集。每个 MCDS 顶点充当组的参考，该组由此顶点和与其相连的所有顶点组成

#### 2.3.3 RANSAC 广义 Procrustes 分析

- RANSAC GPA（RGPA）算法，用于合并组中的所有结构。

- 选择 MCDS 顶点 $P_r$ 作为初始参考形状。
- 在对齐步骤和参考更新步骤之间进行迭代

	- 对齐步骤，使用 RANSAC 扩展 Procrustes 分析（PA）确定的有效匹配，将组 $G$ 中的每个结构与参考 $P_r$ 对齐。

		- 通过使用少量点（5个点）进行多次 PA 试验，选择具有最多内点的结果来对齐两个结构
		- 选择 LMedS 的 RANSAC 策略
		- 每次 PA 操作因子变量 $μ_(i, j)$ 仅在 RANSAC 检测到的内点上取值为 1，故每次 PA使用的 3D 点不相同，而是根据 RANSAC 检测到的内点匹配动态变化

			- 公式（2）的注册和优化是在针对内点进行的，这些内点在迭代过程中进行实时更新
 
	- 参考更新步骤，将所有对齐的形状叠加，对至少在两个结构中出现的所有点进行平均并接受为新的参考点来计算新的参考形状

		- 此步骤可以添加一些最初不在 $P_r$ 中但在至少两个其他结构中的点到参考中
		- 只有至少在两个结构中的点才用于形状对齐和参考更新
		- 迭代 3 次已足够，可使参考形状在不同的噪声水平下收敛
		- 在嘈杂的环境下，RGPA 比早期的鲁棒 GPA 尝试更高效

			- RGPA 能够同时合并多个形状，并带来了较浅的重建树的好处
			- 对分布在原始形状中的离群值具有抵抗力，因此可以在相对嘈杂的环境中稳健地工作
			- 参考形状的动态变化是 RGPA  的另一个重要优势，为传统 GPA 框架提供了对抗噪声的理想特性

	- 组中的结构之间可能存在一些重复的相机（例如，结构 1 包含相机 1,2，结构 2 包含相机 2,3，相机 2 是重复的）

		- 在对齐后，在保留与更多 3D 点相关联的相机的情况下，丢弃其他相机，以确保一个结构中的相机的唯一性。

#### 2.3.4 更高级别的分组和合并

- 在继续到重建树的下一个较高级别之前，可以选择在每个新生成的结构上执行稀疏捆绑调整（BA）
- RGPA 内置的离群值检测机制，结构通常具有良好的准确性，因此不需要进行这个可选的 BA 操作
- 结构分组和合并操作是自下而上执行的
- 在达到顶层后，我们执行最终的BA以完善结构
- 非常小的树深度得以实现快速重建
- RGPA 可以并行化，以进一步减少运行时间

## 3 实验


- 采用了 SIFT 特征提取、5 点算法、BA、LMedS RANSAC
- 在 EPFL Fountain-P11 数据集进行测试，人为添加了各种比例（e）的离群噪声

	- 在不同数量的离群噪声下，RGPA 在迭代中估计的内点比率迅速增加
	- 离群值被有效地检测和丢弃
	- 在所有噪声水平下，估计的内点比率在第 3次迭代后收敛并非常接近 1.0，3 次迭代对 RGPA 方法已经足够了

- 比较在 RGPA 中是否使用 RANSAC（ LMedS ） 的性能差距

	- 没有 RANSAC，无法实现可靠的重建，重投影误差很大
	 
- 将 RGPA 与分层（HIER）、全局和增量（VSFM）SfM  的 SOTA 方法进行比较

	- 全局方法的相机姿态精度不如其他方法，这一问题在结构更不规则的数据集上被放大，无法在一些具有挑战性的数据集上使用
	- 分层方法（HIER 和 RGPA）在这个测试中表现最好，并且两者结果相近

- 测试了更大的数据集，这些数据集结构更不规则

	- 对于具有较大相机旋转的场景，全局方法的相机配准变得不稳定，并且无法提供可靠的初始化
	- **Building** 在这5个数据集中是最容易的

		- RGPA 和 HIER 只执行生成树重建，忽略了许多多余的点，因此它们重建的点较少
		- RGPA 和 HIER 比 VSFM 快得多

			1. 在 RGPA 和 HIER 中，中间 BA 被删除
			2. 分层 SfM 的树结构比增量 SfM 的线结构更有效、

	- **Dante** 是一个非常具有挑战性的数据集

		- 它是 5 个数据集中离群比例最高的，最大的局部离群比例达到 30%
		- 大相机旋转和相机之间的稀疏关联对全局方法造成问题
		- VSFM 恢复了 38 台相机，但由于内点投影较少，无法恢复最后一台相机
		- 而 RGPA 和 HIER 成功地恢复了所有相机
		- 由于漂移，HIER 的恢复点相对嘈杂，质量不如 RGPA。
		- 在 RGPA 中，中间BA是不必要的

	- **Piazzabra** 包含许多窄基线相机和重复结构

		- 导致了许多不正确的极线几何和大量的不匹配，有几个局部结构的离群比率相对较高，约为20%
		- VSFM受到了负面影响，并恢复了两个分开的部分，且未能将它们组合在一起
		- RGPA 和 HIER 成功地恢复了整个场景。HIER 的结果不如 RGPA 好。 
	
	- **Trevi** 和 **Colosseum** 是 Rome16k 数据集（BigSFM站点）中最大的两个子集

		- RGPA 和 VSFM 提供了令人满意的结果；HIER 得出了可接受的结果，但存在一些可见的错误

			- 更小的分层 SfM 的树深度和多结构之间的相互检查使得 RGPA 对漂移更不敏感

		- RGPA 和 HIER 恢复的相机较少于 VSFM

			- 确保场景的最大连通分量的良好状态，一些“孤立”图像会自动删除

		- RGPA 比 HIER 的速度优势明显

- 对于所有实验，通过最终 BA 计算的焦距的平均相对偏差都在 2% 以下
- RGPA 通过并行化，运行速度快了4-7倍；而 HIER 不容易并行化
- 总之，RGPA 比 HIER 和 VSFM 运行得更快，因为其树形结构组织可以一次合并多个结构，并且不需要中间 BA。而内置的离群值检测机制使 RGPA 对可能妨碍其他 SfM 方法的错误不敏感，从而实现更稳健的重建。

- RGPA 的局限性

	- 在一定程度上依赖于结构之间的紧密连接来实现完全重建

		- 粗略地说，对于松散连接的结构，场景将在弱边缘处分裂成几个连接组件，而在没有足够共同点的情况下，每个连接组件只能完成部分重建

## 4 结论

- 在本文中，我们提出了一种可以快速执行的树状 SfM 算法，同时可以可靠处理离群值
- 方法优点

	1. 通过使用 RGPA，可以同时合并多个结构。极大地增加了效率。
	2. 内置多结构交叉检查，可以有效地检测并去除离群值，使算法对噪声具有抗性。
	3. 通过使用生成树结构和最小连接支配集，高效地组织无序图像并分组结构。因而可以快速而可靠地完成自下而上的重建。
	
- 实验证明，我们的方法在高度非结构化场景的效率和稳健性方面优于最先进的方法

## 参考文献

??? info "References"

	Agarwal, S., Snavely, N., Simon, I., Seitz, S.M, Szeliski, R., 2009. Building Rome in a day. In: ICCV, pp. 72–79.

	Arie-Nachimson, M., Kovalsky, S.Z., Kemelmacher-Shlizerman, I., Singer, A., Basri, R., 2012. Global motion estimation from point matches. In: Proc. 3DPVT.

	Bhowmick, B., Patra, S., Chatterjee, A., Govindu, V., Banerjee, S., 2014. Divide and conquer: efficient large-scale structure from motion using graph partitioning. In: ACCV14.

	BigSFM site, http://www.cs.cornell.edu/projects/bigsfm/#data.

	Brown, M., Lowe, D., 2005. Unsupervised 3d object recognition and reconstruction in unordered datasets. In: 3DIM, pp. 56–63.

	Chatterjee, A., Govindu, V.M., 2013. Efficient and robust large-scale rotation averaging. In: ICCV.

	Choi, S., Kim, T., Yu, W., 2009. Performance evaluation of RANSAC family. In: BMVC.

	Corsini, M., Dellepiane, M., Ganovelli, F., Gherardi, R, Fusiello, A., Scopigno, R., 2013. Fully automatic registration of image sets on approximate geometry. IJCV 102, 91 – 111.

	Crandall, D., Owens, A., Snavely, N., Huttenlocher, D., 2011. Discrete-continuous optimization for large-scale structure from motion. In: CVPR, pp. 3001–3008.

	Crosilla, F., Beinat, A., 2002. Use of generalised Procrustes analysis for the photogrammetric block adjustment by independent models. ISPRS J. Photogramm. Remote Sens.

	Crosilla, F., Beinat, A., 2006. A forward search method for robust generalised Procrustes analysis. Data Anal. Classif. Knowl. Organ.

	Eggert, D., Lorusso, A., Fisher, R., 1997. Estimating 3d rigid body transformations: a comparison of four major algorithms. Mach. Vision Appl. 9 (5), 272–290.

	Enqvist, O., Kahl, F., Olsson, C., 2011. Nonsequential structure from motion. In: OMNIVIS.

	Fischler, M., Bolles, R., 1981. Random sample consensus: a paradigm for model fitting with applications to image analysis and automated cartography. Commun. ACM 24 (6), 381–395.

	Frahm, J., et al., 2010. Building Rome on a cloudless day. In: ECCV.

	Furukawa, Y., Curless, B., Seitz, S.M., Szeliski, R., 2010. Towards internet-scale multi-view stereo. In: Proc. CVPR.

	Gherardi, R., Farenzena, M., Fusiello, A., 2010. Improving the efficiency of hierarchical structure-and-motion. In: CVPR.

	Govindu, V., 2006. Robustness in motion averaging. In: ACCV, pp. 457–466.

	Guha, S., Khuller, S., 1998. Approximation algorithms for connected dominating sets. Algorithmica 20 (4), 374–387.

	Hane, C., Zach, C., Cohen, A., Angst, R., Pollefeys, M., 2013. Joint 3D scene reconstruction and class segmentation. In: CVPR, pp. 97–104.

	Hartley, R., Trumpf, J., Dai, Y., Li, H., 2013. Rotation averaging. IJCV 103 (3).

	Havlena, M., Torii, A., Knopp, J., Pajdla, T., Randomized structure from motion based on atomic 3D models from camera triplets, CVPR 2009.

	Havlena, M., Torii, A., Pajdla, T., 2010. Efficient structure from motion by graph optimization. In: Proc. ECCV.

	Heinly, J., Dunn, E., Frahm, J.M., 2014. Correcting for duplicate scene structure in sparse 3D reconstruction. In: ECCV.

	Jiang, N., Cui, Z., Tan, P., 2013. A global linear method for camera pose registration. In: ICCV.

	Li, X., Wu, C., Zach, C., Lazebnik, S., Frahm, J., 2008. Modeling and recognition of landmark image collections using iconic scene graphs. In: ECCV.

	Lou, Y., Snavely, N., Gehrke, J., 2012. Matchminer: efficiently mining spanning structures in large image collections. In: ECCV.

	Lourakis, M., Argyros, A., 2009. SBA: a software package for generic sparse bundle adjustment. ACM Trans. Math. Softw. 36 (1).

	Lowe, D., 2004. Distinctive image features from scale-invariant keypoints. IJCV 60 (2), 91–110.

	Martinec, D., Padjla, T., Robust rotation and translation estimation in multiview reconstruction. CVPR 2007, pp. 1–8.

	Moulon, P., Monasse, P., Marlet, R., 2012. Adaptive structure from motion with a contrario model estimation. In: ACCV.

	Moulon, P., Monasse, P., Marlet, R., 2013. Global fusion or relative motions for robust, accurate and scalable structure from motion. In: ICCV.

	Ni, K., Dallaert, F., HyperSfM, 2012. International Conference on 3D Imaging, Modeling, Processing, Visualization and Transmission.

	Nister, D., Reconstruction from uncalibrated sequences with a hierarchy of trifocal tensors. ECCV 2000, 649–663.

	Nister, D., 2004. An efficient solution to the five-point relative pose problem. IEEE Trans. PAMI 26 (6), 756–777.

	Nister, D., Stewenius, H., Scalable recognition with a vocabulary tree. CVPR 2006, 2161–2168.

	Olsson, C., Enqvist, O., 2011. Stable structure from motion for unordered image collections. SCIA, 6688, pp. 524–535.

	Olsson, C., Erisson, A., Hartley, H., 2011. Outlier removal using duality. In: CVPR.

	Pizarro, D., Bartoli, A., 2011. Global optimization for optimal generalized procrustes analysis. In: CVPR, pp. 2409–2415.

	Pollefeys, M., Gool, L., Vergauwen, M., Verbiest, F., Cornelis, K., Tops, J., Koch, R., 2004. Visual modeling with a hand-held camera. IJCV 59 (3), 207–232.

	Prim, R.C., 1957. Shortest connection networks and some generalizations. Bell Syst. Tech. J. 36, 1389–1401.

	Raguram, R., Frahm, J.-M., RECON, 2011. Scale-adaptive robust estimation via residual consensus. In: ICCV, pp. 1299–1306.

	Samantha site, http://www.diegm.uniud.it/fusiello/demo/samantha/.

	Shah, R., Deshpande, A., Narayanan, P., Multistage SFM: revisiting incremental structure from motion, 2014 3DV14.

	Sinha, S., Steedly, D., Szeliski, R., 2012. A multi-stage linear approach to structure from motion. Trends and Topics in Computer Vision. Springer, pp. 267–281.

	Snavely, N., Seitz, S., Szeliski, R., 2006. Photo tourism: exploring photo collections in 3d. ACM TOG 25, 835–846.

	Snavely, Seitz, S., Szeliski, R., 2008. Skeletal graphs for efficient structure from motion. In: CVPR.

	Strecha, C., von Hansen, W., Gool, L.V., Fua, P., Thoennessen, U., 2008. On benchmarking camera calibration and multi-view stereo for high resolution imagery. In: CVPR, pp. 2838–2845.

	Triggs, B., McLauchlan, P., Hartley, R., Fitzgibbon, A., 2000. Bundle adjustment—a modern synthesis. Vision Algorithms: Theory and Practice 298–372.

	Wilson, K., Snavely, N., 2014. Robust global translations with 1DSfM. In: ECCV.

	Wu, C., Agarwal, S., Curless, B., Seitz, S., 2011. Multicore bundle adjustment. In: Proc. CVPR, pp. 3057–3064.

	Wu, C., 2013. Towards linear-time incremental structure from motion. In: 3DV.

	Zach, C., Klopschitz, M., Pollefeys, M., 2010. Disambiguating visual relations using loop constraints. In: CVPR.
