---
title: SfM on-the-fly: A robust near real-time SfM for spatiotemporally disordered high-resolution imagery from multiple agents
date: 2025-09-28 21:27:41
---

# SfM on-the-fly: A robust near real-time SfM for spatiotemporally disordered high-resolution imagery from multiple agents

!!! info "论文链接"
    原文：Zongqian Zhan, Yifei Yu, Rui Xia, Wentian Gan, Hong Xie, Giulio Perda, Luca Morelli, Fabio Remondino, Xin Wang. [SfM on-the-fly: A robust near real-time SfM for spatiotemporally disordered high-resolution imagery from multiple agents - ScienceDirect](https://www.sciencedirect.com/science/article/pii/S0924271625001388?via%3Dihub)

!!! warning "注意"
    由 deepseek 和 chatgpt 翻译整理

## 摘要

过去二十年间，运动结构重建（SfM）一直是摄影测量、计算机视觉和机器人领域的研究热点，而实时性能直至近年才受到广泛关注。本研究基于原有实时 SfM 方法（Zhan et al., 2024），提出升级版本（v2），通过三项创新提升图像采集过程中的重建效果：（1）采用分层导航小世界（HNSW）图加速近实时图像匹配，更快识别更多真阳性重叠图像候选；（2）提出自适应加权策略，用于鲁棒的分层局部光束法平差（BA）以优化 SfM 结果；（3）支持多智能体协同 SfM，在存在共注册图像时无缝合并多个三维重建结果。综合实验表明，所提方法（命名为实时 SfMv2）能够以高效时序生成更完整、鲁棒的三维重建。代码详见：http://yifeiyu225.github.io/on-the-flySfMv2.github.io/  

**关键词**：运动结构重建、近实时、协同 SfM、图像检索、重叠图像对、光束法平差、多智能体  

## 1. 引言

近几十年来，运动恢复结构（SfM）越来越受到来自不同领域的研究人员的关注，该主题已发展到相当成熟的阶段，特别是在三个研究方向上：**增量式 SfM**（Agarwal et al., 2009; Wu, 2013; Schönberger and Frahm, 2016; Wang et al., 2018），它依次连续地进行前方交会和后方交会；**分层式 SfM**（Farenzena et al, 2009; Mayer, 2014; Toldo et al., 2015），将图像聚类成重叠的子集，然后分层进行定向；**全局式 SfM**（Jiang et al., 2013; Wilson and Snavely, 2014; Cui and Tan, 2015; Wang et al., 2019, Wang et al., 2021），同步估计所有图像的位姿。然而，由于某些 SfM 模块（特征提取与匹配、双视图几何验证、使用 PnP 的图像位姿估计、三角测量、光束法平差等）需要密集的计算，绝大多数 SfM 方法以**离线模式**工作，即首先收集所有图像，然后使用特定的 SfM 流程来估计图像位姿和相应的稀疏点云。虽然这些方法在精度和鲁棒性方面已显示出其优点，但由于整体离线处理时间，实时应用仅限于简单的在线测量，例如灾害场景中的应急响应制图、快速质量评估、检查和决策等（Zhu et al., 2005; Hinzmann et al., 2018; Kern et al., 2020; Menna et al., 2020）。此外，采集和处理过程中的**实时反馈**（Torresani et al., 2021）将使用户有可能改进最终重建，防止部分缺失和相机网络薄弱。另外，图像采集时间本身并未用于处理。

为了满足估计位姿和 3D 地图点的实时需求，另一个热门研究课题——**VSLAM（视觉同步定位与地图构建）**——应运而生，它以连续帧作为输入，实时输出相机轨迹和 3D 地图。通常，SLAM 系统可以根据以下方面进行区分：不同的嵌入式传感器，例如单目 VSLAM、立体 VSLAM 和视觉-惯性 SLAM（Mur-Artal et al., 2015; Mur-Artal and Tardós, 2017; Campos et al., 2021）；不同的跟踪方法，例如基于特征的 VSLAM（Mur-Artal and Tardós, 2017）和直接法 VSLAM（Engel et al., 2014）；不同的优化方法，例如基于卡尔曼滤波的方法（Davison, 2003）和基于图的 VSLAM（Grisetti et al., 2010）。所有这些 VSLAM 解决方案都由几个常见的并行处理线程组成，处理跟踪、地图生成、优化和闭环检测模块。VSLAM 的一个隐含假设要求输入数据必须是视频或连续帧，即帧必须在时空上是连续的，并且两个相邻帧在时间和空间上应该是连续的（如图 1 所示）。此外，为了减轻误差累积的限制，大多数 VSLAM 方法经常采用**闭环检测**来在长期跟踪后校正轨迹并改进地图，这需要在数据收集期间重新访问先前绘制过的区域。因此，为 VSLAM 收集的数据受到时空连续性（最好有闭环）约束的阻碍。与 VSLAM 不同，图 1 例示了所提出的实时 SfMv2 的一般工作流程，它无需时空连续性要求。实际上，图像可以由不同的智能体（例如人员、机器人等）使用移动采集平台以任意方式捕获。

通常，通用的 SfM 解决方案很难实现与常见 VSLAM 系统（例如 30 Hz）相同的在线、实时或实时性能。这与以下原因有关：SfM 中使用的图像（尤其是专业摄影测量图像）通常比 VSLAM 输入的视频帧具有更高的分辨率；SfM 采用更复杂的特征（例如 SIFT），而 VSLAM 采用更简单的特征（例如 ORB）以提高时间效率；与 SfM 复杂的穷举特征匹配不同，VSLAM 使用运动模型或光流在相邻帧之间跟踪同源特征；SfM 迭代调用全局光束法平差来优化大型 3D 特征地图以提高精度，而 VSLAM 使用滑动窗口应用局部光束法平差或对保持有限数量观测值的地图进行全局图优化，这对 3D 精度产生负面影响。在我们的实时 SfMv2 中，挑战在于以近实时的方式（通常需要 2-3 秒让新捕获的图像到达，因为需要图像捕获、存储和传输时间）提供每个新飞入图像与现有图像块的**协同配准**，同时保持全尺寸图像分辨率和高精度。

这项工作建立在 Zhan 等人（2024）提出的实时 SfM 方法之上，这是一个开源的近实时 SfM 流程，通过以任意方式但从**单个智能体**捕获的图像，以低延迟估计图像位姿和 3D 稀疏点（如传统离线 SfM 通常输出的那样）。如图 2 所示，这项工作更进一步，探索了加速处理和处理参与 3D 重建的**多个智能体**的可能性。主要进展包括：

- **改进的近实时图像检索**：在我们之前的实时 SfM（Zhan et al., 2024）中使用的是预训练的词汇树，而现在采用所谓的**分层导航小世界（HNSW）图**（Malkov and Yashunin, 2020）来改进近实时图像检索。HNSW 图在检索精度和时间效率上更优，并且可以随着新飞入图像动态构建，这更适用于所提出的实时 SfMv2。
- **局部光束法平差（BA）的自适应加权**：为了进一步提升局部光束法平差的性能，采用了一种用于分层局部 BA 的**自适应加权策略**。与第一个版本（Zhan et al., 2024）研究权重的幂函数经验设置不同，这项工作利用图像检索阶段生成的相似度来自适应地估计权重。
- **支持多智能体和重建的能力**：与第一个版本只能处理单个智能体相比，SfM 已扩展为在数据由多个智能体采集时以近实时方式运行，创建并合并并行的 3D 子重建。多智能体协作可以提高数据采集效率，同时实时的 3D 重建反馈有助于加速操作并避免数据中的漏洞和不足。

## 2. 相关工作

在过去的几十年中，许多 SfM 解决方案和应用被提出。在本节中，我们回顾了四个相关主题，包括 SfM、VSLAM、图像检索和高效光束法平差，重点关注实时性能和精度。

### 2.1. SfM（运动恢复结构）与 VSLAM（视觉同步定位与地图构建）

SfM 和 VSLAM 都专注于从图像生成几何信息，更具体地说，它们估计图像位姿和稀疏或密集数量的 3D 物体点位置。通常，SfM 还估计相机内参，而 VSLAM 通常假设已知内参。

**SfM**：SfM 主要分为三类：增量式 SfM、分层式 SfM 和全局式 SfM。**增量式 SfM**按顺序求解图像，交替进行迭代光束法平差和全局光束法平差，已被广泛探索，许多流行的学术软件包被成功开发。Bundler（Snavely et al., 2006）是最早处理大规模无序图像集合并允许用户在增量处理过程中可视化中间重建结果的开源资源之一，VisualSFM 由（Wu, 2016）实现，具有用户友好的图形界面（GUI）。最近，三个软件包，即 Theia（Sweeney, 2016）、OpenMVG（Moulon et al., 2016）和 Colmap（Schönberger and Frahm, 2016），经常出现在相关工作中。Colmap 因其在稳定性、鲁棒性和高精度方面的优越性而受到广泛关注（Jiang et al., 2020; Jiang et al., 2021）。得益于这些框架，许多努力致力于改进增量式 SfM。Wu（2013）采用迭代重三角化策略来处理由噪声位姿引起的误差累积。Schönberger and Frahm（2016）在 Colmap 中做出了三个主要贡献：通过考虑本质矩阵、基础矩阵和单应矩阵来验证双视图几何；根据可见连接点的数量及其在每个候选图像上的分布来选择新添加的图像；应用基于 RANSAC 的 DLT 进行三角测量。Wang et al.（2018）分别求解外部旋转和平移，使用新图像与已注册图像之间的相对旋转来鲁棒地估计旋转，并通过线性方程组获得平移。
随着增量式 SfM 的出现，**分层式 SfM**开始被研究，其总体思想是将所有图像分割成重叠的子簇，并分层合并每个子簇的 SfM 结果。由于每个子簇易于并行化，时间效率得到了提高。Zhao et al.（2018）提出了 Linear SFM，它从小型局部子簇开始，使用非线性光束法平差，然后使用线性最小二乘和非线性变换分层合并已求解的子簇。Michelini and Mayer（2020）从三元组开始，分层合并连接的三元组，并通过随机忽略共享的连接点进一步提高了时间效率。Liang et al.（2023）提出了一种专为大规模倾斜图像设计的分层 SfM。他们应用 GPS/IMU 和地形信息有效地分组图像，显示出良好的精度和时间效率。
为了进一步加速 SfM 处理，上述迭代和分层解决方案逐渐被**全局式 SfM**更新，因为它可以同时处理所有图像以估计其相应的初始外部定向参数，并且只运行一次最终的光束法平差进行优化。为了求解全局旋转，许多研究以相对旋转（来自双视图极几何）作为输入，并估计每张图像的旋转，以最小化与相应相对旋转的误差距离。Chatterjee and Govindu（2013）首先应用基于李代数的 L1 范数旋转平均，然后使用类似 Huber 的损失函数通过迭代重加权最小二乘法细化结果，该方法声称比 Crandall et al.（2011）和 Hartley et al.（2011）的计算效率更高、更鲁棒。基于估计的全局旋转，可以求解每张图像的全局平移。Wilson and Snavely（2014）提出了一个非线性优化系统，研究相对平移和全局平移的方向差异。Zhuang et al.（2018）探索了不同的基线长度，发现全局平移的精度对较长的基线更敏感，他们提出了一个在各种基线上都表现良好的角度误差函数。Wang et al.（2019）提出了一个线性方程组，首先计算相对平移的全局一致比例因子，然后通过这些缩放后的相对平移同时估计全局平移。Wang et al.（2021）研究了多视图本质矩阵的几何属性，并在相应图像不共线的情况下同步求解估计的全局旋转和平移。

此外，也有工作专注于使用**多智能体的协同 SfM**。Untzelmann et al.（2013）提出了一个城市尺度协同重建的框架，他们通过指定的智能体分别重建单个建筑物，然后将其映射到全局坐标系中。Locher et al.（2016）使用多个手机作为图像采集的前端，集中式服务器作为处理的后端，平衡了在线 3D 重建的相应处理负载和带宽。基于这项工作，Nocerino et al.（2017）和 Poiesi et al.（2017）提出了一种嵌入智能手机和基于云服务器的协同方法，其中输入具有高时空约束的连续帧以使图像匹配轻量化。

**VSLAM**：VSLAM 方法能够在线跟踪和建图，并朝着两个主要方向发展：第一，**基于特征的方法**，例如 PTAM（Klein and Murray, 2007）、VINS-Fusion（Geneva et al., 2020）、PL-SLAM（Pumarola et al., 2017）和 ORB-SLAM 系列。这些方法专注于提取和跟踪环境中的显著特征来估计每帧的位姿并构建地图。通常，关键点（斑点或角点）或线被提取为特征，然后在连续帧之间进行匹配以跟踪运动状态并重建结构。此类方法通常假设相机已预先校准，并且已在各种环境（例如，室内基准测试（Sturm et al., 2012）和室外基准测试（Geiger et al., 2012））中得到了广泛成功的测试。然而，它们在纹理较差和重复模式的情况下可能会退化；第二，**直接法**。为了确保在复杂场景（尤其是在纹理贫乏的地方）的长期跟踪，提出了直接法，例如 LSD-SLAM（Engel et al., 2014）、SVO（Forster et al., 2014）和 DSO（Engel et al., 2018），它们根据相邻帧之间的强度信息估计位姿和稠密（或近稠密）结构。其内在假设是 3D 物体场景在连续图像上的外观在极短时间内保持恒定，这可以在基于特征的方法可能失败的弱纹理环境中有效操作。然而，这些直接方法对光照变化敏感并且需要良好的初始化。另一方面，**基于学习的方法**（Jin et al., 2021; Morelli et al., 2022）也被集成到 VSLAM 的各个模块中。例如，在前端，DXSLAM（Li et al., 2020）用 CNN 提取的深度特征替换了原始的手工特征，并训练了新的对应词袋结构；类似地，（Bruno and Colombini, 2021）应用学习到的局部特征 LIFT（Yi et al., 2016）作为输入观测。在后端，Tateno et al.（2017）提出了 CNN-SLAM，通过 CNN 预测稠密深度图，并结合单目 SLAM 的稀疏深度；Tang and Tan（2019）提出了一个可处理的网络用于光束法平差，首先预测几个基础深度图，然后通过线性组合进行优化以产生最终深度。在 COLMAP-SLAM（Morelli et al., 2023a; Morelli et al., 2023b）中，用几种基于深度学习的局部特征和匹配器替代了 Colmap 中 SIFT 和最近邻匹配的实现，以应对具有挑战性的光照条件。

与传统的 SfM 方法将所有收集的图像一起输入并进行后续几何处理不同，这项工作实现了一个**在线 SfM 框架**，同时处理基于多智能体的图像采集，且无需专业的摄影测量规范。并且，与 VSLAM 相比，不需要时空连续性的必要性，也不需要 GPS/IMU 的先验知识。三个最相关的工作是 Hoppe et al.（2012）、所谓的实时 SfM（Zhao et al., 2022）和我们之前的实时 SfM（Zhan et al., 2024）：第一个工作试图通过假设新获取的图像与已有重建重叠来减少双视图几何估计的时间成本；Zhao et al.（2022）提出了一种分层解决方案，使用 BoW 和多视图单应性改进图像匹配，但图像间的时空连续性仍然需要；最新的工作是我们实时 SfMv2 的初始版本，其改进在第 1 节中已说明。

请注意，虽然有许多优秀的 SfM 和 VSLAM 方法值得回顾，但我们只选择了一些流行且相关的工作进行评述。

### 2.2. 高效图像检索

图像检索在 SfM 和 VSLAM 中扮演着至关重要的角色，因为它被广泛用于快速识别重叠图像对和检测闭环。至于**在线 SfM**，第一步是找到新添加的图像与已注册图像之间的相关性，因此，在这种情况下，高效的图像检索方法更加关键。如今，相关研究正朝着两个方向发展：**特征精炼**和**高效索引结构**。

- **特征精炼 (Features refinements)**：图像特征通常用于计算相应的相似度，这可以决定检索性能的精确程度。
    - **局部特征的精炼**：局部特征通常很丰富，并由高维描述子表示，这导致寻找其最近邻需要密集的计算。为了减少特征数量，**抢先匹配 (preemptive matching)** (Wu, 2013) 利用了更高尺度的 SIFT 特征是相似度的充分指标这一先验知识，并将匹配仅限于这些特征。Hartmann et al. (2014) 训练了一个随机决策森林分类器，输入 SIFT 特征描述子以预测其可匹配概率，并通过减少每幅图像的特征总数来加速图像匹配。Michelini and Mayer (2020) 将原始的 128 维实数空间 SIFT 特征映射到{0,1}汉明空间，从而可以使用流式 SIMD 扩展(SSE)轻松估计相似度。
    - **全局特征的精炼**：全局特征现在在检索相似图像方面更受欢迎，主要是因为其高效的计算能力以及基于学习的全局特征的优越性。Razavian et al. (2016) 提出了一种简单有效的方法，从现成的 CNN 模型中提取全局特征，并在最后的激活图上逐通道应用最大池化操作。Arandjelović et al. (2016) 将传统 CNN 模型的最后一个池化层更改为通过 VLAD 的软分配的可训练池化层，这增强了图像位置识别的性能。Radenović et al. (2016) 首先提出了一种利用 SfM 结果生成相似和不相似图像对的自动注释方法，基于此对预训练的 CNN 进行微调以获得更好的全局图像特征。Shen et al. (2018) 通过探索 3D 网格模型生成真实的重叠图像对，并研究了从网格重投影到成对图像上的三角形，然后调整 CNN 以使提取的全局特征对可匹配的图像对更敏感。Yan et al. (2021) 采用**图神经网络(GNN)** 从基于 CNN 的全局特征中进一步筛选相似的候选图像。最近，Hou et al. (2023) 提出了一种集成多个 NetVLAD 的 CNN 微调方法，以聚合不同通道的特征图，并发布了一个包含众包和摄影测量图像的基准测试 LOIP。

- **高效索引结构 (Efficient indexing structure)**：从图像中提取特征后，通常会构建索引结构以高效地检索目标查询的最近邻。

    **词袋模型(Bag-of-word)** (Sivic and Zisserman, 2006) 是一个预训练的分层索引树，以聚类中心为节点，基于 tf-idf 方法，每幅图像表示为一个向量，以高效地找到相似的候选图像。**VocMatch** (Havlena and Schindler, 2014) 使用 SIFT 特征训练一个大的两层词汇树，第一层有 4096 个节点，第二层有 4096×4096 个节点，对应关系应位于同一节点内。Zhan et al. (2015) 通过提出**多个分层索引树**改进了词袋模型，并使用 GPU 进一步提高了检索速度。后来，Wan et al. (2018) 没有使用局部图像特征，而是使用来自预训练 VGG-16 网络(Simonyan and Zisserman, 2014)的 CNN 特征图，通过基于 CNN 的 BoW 形成图像的表示向量来构建词袋(BoW)模型。Wang et al. (2019) 建议使用**随机 KD 森林**来构建多个相互独立的 KD 树，这增加了找到真正最近候选图像的可能性，而无需耗时的回溯过程。Jiang and Jiang (2020) 研究了通过词汇树估计的相似度得分的统计信息，表明其能够通过扩展最小生成树(MST)来选择更多的图像对，并避免摄影测量块破裂。受 VocMatch 启发，Zhan et al. (2024) 和 Wang et al. (2024) 使用全局特征(Hou et al., 2023)无监督地训练了一个分层词汇树，期望相似图像的全局特征被分类到同一节点，该方法已成功嵌入到我们之前的实时 SfM 中。

### 2.3. 光束法平差的高效优化

得益于过去几十年相关理论的发展，光束法平差(BA)已得到广泛研究，并已成为优化图像位姿和 3D 点位置的成熟技术，然而它仍然是所有几何处理中最耗时的步骤之一，这对于近实时 SfM 尤为关键。随着注册图像数量的增加，开发了许多以时间高效且稳定的方式求解 BA 的方法。例如，Agarwal et al. (2010) 发现大规模 BA 通常会导致大型线性方程组，而经典的舒尔补(Schur complement)速度仍然不够快，他们随后探索了**预条件共轭梯度法(PCG)** 来迭代但高效地估计大型正定方阵的逆。基于 PCG，Wu et al. (2011) 提出了**多核光束法平差(Multicore bundle adjustment)**，并通过在多个 CPU（中央处理单元）和 GPU（图形处理单元）上实现 PCG 进一步提高了 BA 的速度。Zheng et al. (2017) 将多核光束法平差的应用扩展到极大规模的高分辨率无人机图像上，并证明了其优越性。Huang et al. (2021) 提出了**DeepLM**，基于反向雅可比网络，他们开发了一个通用且高效的**Levenberg-Marquardt (LM)** 求解器，可以自动计算稀疏雅可比矩阵。为了处理大规模 BA 问题，将大型 BA 问题划分为几个重叠小子集的**分布式方法**引起了研究人员的兴趣。Zhang et al. (2017) 并行求解每个子集 BA，在全局相机一致性约束下，使用**交替方向乘子法(ADMM)** 算法迭代地将所有优化后的子集合并为一个完整的块。Mayer (2019) 将共享的 3D 连接点作为全局一致性约束，并通过整合相应的协方差信息实现了更好的收敛性。**MegBA** (Ren et al., 2022) 通过多个 GPU 并行求解子集，提供了一个更省时的解决方案。最近，Zheng et al. (2023) 提出了**DBA（分布式光束法平差）**，采用基于块的稀疏矩阵压缩，并且 LM 方法可以精确地应用于划分子集的简化相机系统。

所有上述回顾的工作都试图通过**同时处理所有未知数**（包括位姿和 3D 点）来实现高效的 BA 优化，这对于增量式、顺序式和在线 SfM 模式来说本质上是不可行的，因为在每个新图像添加后运行全局 BA 极其耗时。为了解决这个问题，VSLAM 方法 (Mur-Artal and Tardós, 2017; Campos et al., 2021) 应用一个**滑动窗口**来限制参与 BA 的图像数量，这导致在几个相邻帧（在时间和空间上）上形成一个小的优化问题。这种简化有效地将全局 BA 简化为局部 BA，以牺牲精度来节省处理时间。Colmap 提出了一种在局部和全局光束法平差之间交替的策略，以在精度和处理时间之间取得折衷。然而，局部平差中涉及的固定数量的图像可能无法覆盖所有关联图像，这可能会影响平差的准确性。

## 3. 实时 SfMv2

在本节中，我们详细概述了实时 SfMv2 的整体工作流程，重点关注三个主要贡献：1）使用基于学习的全局特征和 HNSW 实现更快的图像检索；2）通过自适应加权的分层树改进高效的局部 BA 优化；3）子地图关联与合并。

### 3.1. 实时 SfMv2 概述

图 3 显示了我们新系统的通用工作流程和主要组件，这些基本类似于第一版实时 SfM，其中最重要的新特性用红色虚线框突出显示。它主要包括四个部分：多智能体图像采集、基于 HNSW 的在线图像匹配、在线子地图以及多子地图处理。具体来说，对于每个新捕获的图像，首先提取其全局特征和局部特征。然后，基于 HNSW 检索结果执行在线图像匹配，以生成图像对之间的连接点。接着，进行在线子地图重建，包括使用获得的对应点进行双视图几何验证、图像注册、三角测量和局部 BA，从而生成子地图。最后，在多子地图处理模块中，系统根据图像检索结果检查子地图是否满足合并标准。满足条件的子地图将被合并，从而可能得到一个唯一且完整的重建结果。

- **多智能体图像采集**：与之前处理单个智能体和用于图像传输的无线 WIFI 发射器的实时 SfM 不同，实时 SfMv2 简化并加速了图像采集：首先，实时 SfMv2 兼容多种平台，如手机和 iPad，这提高了工作的灵活性；其次，支持多智能体工作模式以处理来自各种传感器的图像，这有望提高时间效率；第三，实时 SfMv2 的近实时图像传输通过局域网、4G 和 5G 实现，增强了系统的实用性（更多细节见第 4.1 节 多智能体实时 SfM 平台）。
- **基于 HNSW 的在线图像匹配**：必须对采集的图像进行比较，以识别显示相同场景的图像对和将用于图像块定向的 2D 连接点。对于近实时 SfM，为每个新飞入图像确定重叠图像对的准确性和时间效率至关重要，以限制新图像到达与其在现有图像块中准确定向之间的时间延迟。此外，多智能体需要更快的图像检索，因为可能同时有更多图像到达。为此，在实时 SfMv2 中，保留了基于学习的全局特征（Hou et al., 2023），但使用 HNSW（Malkov et al., 2018）来确保在线图像检索的速度和准确性（更多细节见第 3.2 节）。
- **在线子地图**：图像位姿估计和连接点三角测量模块基本与实时 SfM（Zhan et al., 2024）相同。首先使用各种模型（本质矩阵、基础矩阵和单应矩阵）（Schönberger and Frahm, 2016）验证双视图几何，并选择一个初始立体重建，然后使用 EPnP（Lepetit et al., 2009）和基于 RANSAC 的多视图三角测量（Schönberger and Frahm., 2016）来解决图像注册和三角测量问题。在这项工作中，我们研究了新飞入图像应如何影响其连接的重叠图像，提出了一种基于图像相似性和 HNSW 的、具有自适应分层权重的局部 BA，以鲁棒且快速地解决在线子地图处理的优化部分，显著减少了处理时间（详见第 3.3 节）。
- **多子地图处理**：相对于实时 SfMv1 的一个重要改进是处理一组子地图的能力，这在实际中经常发生（尤其是在任意捕获图像的情况下）：例如，一个智能体在一个建筑立面上拍摄图像，然后直接访问另一个立面采集图像，直到出现重叠图像；或者两个智能体从不同立面拍摄建筑物的图像，并逐渐相互重叠。因此，在实时 SfMv2 中，一旦通过提出的基于 HNSW 的图像检索方法识别出子地图之间的关联，就会将它们合并成一个完整的重建（关于合并解决方案的更多细节见第 3.4 节）。

### 3.2. 基于学习型全局特征和 HNSW 的更快图像检索

为确保多智能体的在线图像匹配和子地图的快速关联识别，我们研究了一种通过分层导航小世界（HNSW）图实现的更快图像检索方法。图 4 展示了所提出的图像检索工作流程和关键模块：1. 选择一个预训练的 CNN 模型作为图像的全局特征提取器（Hou et al., 2023; Arandjelović et al., 2016; Radenović et al., 2019）；2. 然后将每个新的全局特征输入 HNSW，以增量地优化相应的索引结构，并动态快速地识别可匹配的候选图像。

- **基于学习的全局特征提取器**：CNN 在提取可靠且具有区分性的全局特征用于图像检索方面已显示出优越性（Sturm et al., 2012）。在这项工作中，我们继续采用 Hou 等人（2023）的微调 CNN 模型作为我们的全局特征提取器，因为我们之前的实时 SfM 已经证明了其在准确识别重叠图像对和加速离线 SfM 方面的有效性。
- **HNSW 的增量建立**：据我们所知，词汇树（或其相关变体）是在大规模图像定向问题中加速图像检索最常用和最有效的方法之一（Havlena and Schindler, 2014）。然而，基于词汇树检索的时间效率和准确性严重依赖于预训练过程，这通常是一项非常耗时的任务，并且可能无法很好地推广到其他未见过的场景。相比之下，HNSW 可以在新捕获图像传入时近实时地增量构建，而无需预训练。HNSW 的增量过程如图 5 所示。对于每个新飞入的图像，首先使用 Hou 等人（2023）的预训练模型提取全局特征。然后，将该图像作为插入节点添加到 HNSW 结构中，其层级 $l$ 由公式（1）随机选择计算得出，并添加到 $l$ 层及以下的所有层中：
    $$ l = -\ln(unif(0,1)) • m_L \tag{1}$$

    其中 $l$ 是新节点的层，$m_L$ 是层生成的归一化因子，$m_L$ 的一个简单选择是 $1/\ln(Max)$，其中 $Max$ 是一个预设参数，表示 HNSW 中一个节点到所有其他节点的最大连接数。要插入一个新节点及其连接，需要从上到下遍历网络。对于 $l$ 层以上的层，在每层中找到最接近插入节点的节点作为下一层的入口点；而对于 $l$ 层及以下层，搜索最接近插入节点的 $Max$ 个节点，并将它们与插入节点的连接添加到 HNSW 图中（更多细节请参考算法 1 或 Malkov and Yashunin (2020)）。在最底层，$Max$ 增加两倍以确保良好的检索召回率。随着捕获更多图像，HNSW 图的结构会不断更新。

- **基于 HNSW 的快速图像检索**：基于使用已注册图像构建的 HNSW 图，可以在 HNSW 更新过程中快速找到新飞入图像的重叠图像对。如图 6 所示，一旦提取了新的全局特征，就将其作为插入节点输入 HNSW，从上到下遍历 HNSW 图。检索每层中最接近插入节点的现有节点，然后可以从最后一层的这些现有节点中获得最终的 Top-N 结果。在这项工作中，节点之间的距离通过全局特征的欧几里得距离来估计，每层的搜索策略如算法 2（Search_layer）所述。

    对于整个检索流程，每张图像首先由一个全局特征表示（使用预训练的主干网络在 GPU 上推断非常快），并且图像检索过程非常快，时间复杂度为 $O(log(N))$。此外，构建 HNSW 的时间复杂度仅为 $O(N•log(N))$。第 4.2 节的实验结果表明，HNSW 具有优越的时间效率，并且其性能不会随着图像数量的增加而显著下降，使其适用于近实时应用。

### 3.3. 具有自适应分层权重的高效局部光束法平差

为了实现在线重建的近实时性能，采用一种时间高效且鲁棒的光束法平差至关重要。受图 7(a)中同心水波的启发（中心的涟漪总是表现出更大的振幅），类似地，如图 7(b)示例所示，将一个新图像投入一个已求解良好的摄影测量块中，与该新图像连接更紧密的图像应具有更高的影响力，换句话说，新飞入图像中的不确定性对密切相关的图像的影响要大于对距离较远的图像的影响。如图 7(c)所示，我们的工作中使用了一种新颖的、具有自适应分层权重的**高效局部光束法平差**：首先，基于图像检索结果构建一个**分层关联树**（见第 3.2 节），它揭示了新图像与先前已注册图像之间的关联关系；然后，我们为每个局部关联的图像提出一个**自适应分层权重**，并利用它们执行鲁棒的光束法平差。

1. **分层关联树构建与自适应加权**

    所提出的 HNSW 是一种基于图的高效图像检索方法，能为每个新飞入图像快速找到 Top-N 相似图像。此外，图 7(b)直观地显示，来自不同涟漪（或分层）的图像受到新飞入图像添加的影响应该不同，这意味着在光束法平差的优化中不应将它们同等对待。因此，在这项工作中，我们提出了一个**分层关联树**来表示已注册图像与新图像之间的相关性，作为局部 BA 中加权策略的基础。
    第一层涟漪中的图像是当前新飞入图像的 Top-N 相似图像，第二层涟漪由第一层图像的 Top-N 相似图像生成，此过程重复直到达到预设的深度 *L<sub>h</sub>*。在图 8 中，我们展示了一个简化的 4 层分层关联树的示意图，其中每个底层图像都是来自其上一层检索到的 Top-N 图像。分层关联树中包含的所有图像表示为 *I<sub>hat</sub>*。值得注意的是，第一层涟漪图像受新图像的影响最显著，因此在已求解的摄影测量块中可靠性最低，在 BA 中应具有最小的权重。类似地，涟漪层越高，相应图像受影响越小，权重应越大。此外，在同一层中，由于与新图像的相似度不同，不同图像的权重应略有不同。
    根据上述内容，提出了一种针对不同涟漪层中图像的自适应分层加权方法，如公式（2）所示：

    $$
    p_{\ddot{y}}=
    \begin{cases}
    1,ifi,j=* \\
    \\
    1/(\sum_{\epsilon \to ij}\frac{1}{s_k})^{i-1},ifi\neq L_k \\
    \\
    \infty,ifi=L_h &
    \end{cases}   
    \tag{2}
    $$

    其中 $i$ 是层号，$*$ 表示当前新飞入图像，$j$ 指示分类到第 $i$ 层的图像，$S$ 返回由代表新飞入图像 $i$ 和图像 $j$ 的全局特征的欧几里得距离计算的相似度，$\varepsilon \to ij$ 包含连接新飞入图像和第 $i$ 层第 $j$ 个图像的最短边集合（由图 8(b)暗示，由相似度加权）。

    值得注意的是，生成自适应权重的信息已经在检索过程中计算过，因此，它不会改变 BA 的时间效率。

2. **具有自适应分层权重的局部光束法平差**

    - **光束法平差回顾**：令 $x$ 为需要优化的参数向量，包括相机和三维点；$f(x) = [f_1(x), \cdots, f_k(x)]$ 表示各投影射线的残差（reprojection error）。捆绑调整（Bundle Adjustment，BA）的目标是找到最优的 $x$，使重投影误差 $E(x)$ 最小化。通常，这个优化问题被表述为非线性最小二乘问题，其总误差定义为观测特征点与对应三维点在图像上的重投影误差之和，如公式 (3)：

        $$
        x^* = \arg\arg E(x)
        \tag{3}
        $$

        其中 $E(x) = \sum_{i=1}^k | f_i(x) |^2$, $k$ 为投影射线的数量。

        设 $x_t$ 是第 $t$ 次迭代后的更新解，公式 (3) 可以通过泰勒展开近似为：

        $$
        E(x) \approx E(x_t) + g^T(x - x_t) + \tfrac{1}{2}(x - x_t)^T H (x - x_t)
        \tag{4}
        $$

        其中 $g = \frac{\mathrm{d} E}{\mathrm{d} x}(x_t), H = \frac{\mathrm{d}^2 E}{\mathrm{d} x^2}(x_t)$。为解公式 (4)，高斯–牛顿法（Gauss-Newton）是众所周知的：

        $$
        \frac{\mathrm{d} E}{\mathrm{d} x} = g + H(x - x_t) = 0\rightarrow x_{t+1}=x_t-H^{-1}g\tag{5}
        $$

        设 $J(x)$ 为 $f(x)$ 的雅可比矩阵，则 $g = \frac{\mathrm{d} E}{\mathrm{d}l x}(x) = 2J^T f$，而 Hessian 矩阵 $H$ 可近似表示为 $2J^T J$。此外，为了确保矩阵 $H$ 可逆，我们通常采用经典的 Levenberg–Marquardt (LM) 算法（Nocedal and Wright, 2000）修改公式 (5)：

        $$
        x_{t+1} - x_t = -(H + \lambda D^T D)^{-1} g
        \tag{6}
        $$

        其中 $D(x)$ 是非负对角矩阵，常由 $J(x)^T J(x)$ 的对角线元素导出，$\lambda$ 是阻尼因子（非负参数），用于控制梯度下降的强度。于是，更新步长 $\delta$ 的正规方程写为：

        $$
        (J^T J + \lambda D^T D)\delta = -J^T f
        \tag{7}
        $$

        更新步长可写作 $\delta = [\delta_c, \delta_p]^T$，其中 $\delta_c$ 为相机参数，$\delta_p$ 为三维点参数。令 $U = J_c^T J_c, \quad V = J_p^T J_p, \quad U_\lambda = U + \lambda D_c^T D_c, \quad V_\lambda = V + \lambda D_p^T D_p, \quad W = J_c^T J_p$
        。那么公式 (8) 可写为分块线性系统：

        $$
        \begin{bmatrix}
        U_\lambda & W \\
        W^T & V_\lambda
        \end{bmatrix}
        \begin{bmatrix}
        \delta_c \\
        \delta_p
        \end{bmatrix}=-
        \begin{bmatrix}
        J_c^Tf \\
        J_p^Tf
        \end{bmatrix}
        \tag{8}
        $$

        利用 Schur 补，可得到仅关于相机参数的约化正规方程：

        $$
        (U_\lambda - W V_\lambda^{-1} W^T)\delta_c = -J_c^T f + W V_\lambda^{-1} J_p^T f
        \tag{9}
        $$

        然后，通过公式 (9) 可以解得相机参数 $\delta_c$，三维点参数 $\delta_p$ 则通过 $V_{\lambda}^{-1}(J_p^T f + W^T \delta_c)$ 来估计。

    - **集成自适应分层权重的 BA**：为保证在线 SfM 的鲁棒性，基于生成的局部光度块 $\hat{I}$ 及其对应的权重 $\hat{p}_{ij}$，提出了自适应分层权重的局部捆绑调整（local BA）方法。在该方法中，公式 (8) 改写为：

        $$
        \begin{bmatrix} 
        \hat{U}_\lambda \hat{P} & \hat{W}_\lambda \\ \hat{W}_\lambda \hat{P} & \hat{V}_\lambda 
        \end{bmatrix} 
        \begin{bmatrix} 
        \hat{\delta}_c \\ \hat{\delta}_p 
        \end{bmatrix}=-
        \begin{bmatrix} 
        \hat{J}_c^T \hat{P} \hat{f} \\ \hat{J}_p^T \hat{f} 
        \end{bmatrix}
        \tag{10}
        $$

        其中，$\hat{P}$ 为由公式 (2) 得到的对应权重矩阵。
        新的约化相机正规方程为：

        $$
        \big(\hat{U}_\lambda - \hat{W}_\lambda \hat{V}_\lambda^{-1} \hat{W}_\lambda^T\big) \hat{P} \hat{\delta}_c
        = - \hat{J}_c^T \hat{P} \hat{f} + \hat{W} \hat{V}_\lambda^{-1} \hat{J}_p^T \hat{f}
        \tag{11}
        $$

        通过公式 (11)，我们可以快速且鲁棒地计算相机参数更新 $\hat{\delta}_c$，并且三维点更新 $\hat{\delta}_p$ 也可以通过 $-\hat{V}_\lambda^{-1}\big(\hat{J}_p^T \hat{p}_{ij} \hat{f} + \hat{W}_\lambda^T \hat{P} \hat{\delta}_c\big)$ 高效获得。

### 3.4. 多子地图处理

多个子地图的出现是在线近实时解决方案中非常常见的问题，例如 Campos et al. (2021)，它出现在多智能体或使用一个智能体但方式非常任意的情况下。在这部分，我们介绍了处理多个子地图以产生完整重建结果的过程（图 9），包括：子地图关联和子地图合并。

- **子地图关联**：给定两个独立的子地图，当一个新图像飞入时，应用图像检索方法从这两个子地图中已注册的图像中查找相应的重叠图像对。如果在这两个子地图上都成功执行了在线注册，则记录一个共享图像。一旦共享图像（SIs）的数量超过阈值（本文中 $N_{si} = 3$），则使用公共 3D 点进行子地图合并。
- **子地图合并**：在获得公共共享图像和 3D 点后，应用**3D 相似性变换**（更多细节见算法 3）来估计子地图之间的相对定向，并将一个子地图转换到另一个子地图中。我们的实时 SfMv2 首先尝试将较小的子地图（注册图像较少）合并到较大的子地图中（换句话说，将具有更多注册图像的较大子地图视为参考），以提高时间效率和准确性。如果此初始融合失败，则启动替代方案，即将较大的子地图合并到较小的子地图中。只要这两种融合尝试中有任何一种成功，两个子地图就可以合并成一个单一而完整的子地图。如果两种尝试都失败，则两个子地图保持断开连接，同时，下一个新图像飞入并将识别出额外的共享图像，之后将进行另一次合并尝试。值得注意的是，合并尝试由一个全新的线程执行，并且在大多数情况下，合并尝试在几秒钟内完成，因此不会影响其他模块的进程。

    通过采用这种方法，可以快速启动两个关联子地图的合并，这有助于近实时子地图处理。对于多个子地图，以递归方式应用成对融合，直到所有子地图处理完毕。

## 5 结论

在本研究中，我们基于实时运动恢复结构（on-the-fly SfM）（Zhan et al., 2024）提出了一种创新的近实时摄影测量处理方法，用于相机位姿和稀疏点云估算。具体而言，为实现在图像采集过程中获得更好 SfM 重建结果的目标，我们提出了三个主要贡献：（一）提出基于 HNSW 的更快速近实时图像检索方法，以获取更准确的重叠图像；（二）通过集成分层自适应加权策略改进局部光束法平差；（三）新增对多智能体采集图像的融合处理功能。

实验结果表明，所提出的改进型实时 SfM 方法能够取得优于其他 SfM 方法的结果，并以接近图像采集速率的速度实现多智能体近实时 SfM 运算。处理并融合多智能体采集图像的能力使得获取更完整、更精确的三维重建结果成为可能。在三维物体空间精度方面可能略有不足，但处理速度得到了显著提升。未来工作中，我们将从两个方向进一步探索和改进方法：首先，通过集成近实时稠密点云和表面网格生成功能来扩展实时 SfM 系统；其次，以本实时 SfM 为原型工作，探索基于云端的 SfM 处理模式可行性。


??? info "References"

    Agarwal, S., Snavely, N., Simon, I., Seitz, S., Szeliski, R., 2009. Building Rome in a day. In: Proceedings of the IEEE International Conference on Computer Vision (ICCV), pp. 72–79.

    Agarwal, S., Snavely, N., Seitz, S.M., Szeliski, R., 2010. Bundle Adjustment in the Large. In: Proceedings of European Conference on Computer Vision (ECCV), pp. 29–42.

    Arandjelovic, R., Gronat, P., Torii, A., Pajdla, T., Sivic, J., 2016. NetVLAD: CNN architecture for weakly supervised place recognition. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR), pp. 5297–5307.

    Bruno, H.M.S., Colombini, E.L., 2021. LIFT-SLAM: a deep-learning feature-based monocular visual SLAM method. Neurocomputing 455, 97–110.

    Campos, C., Elvira, R., Rodríguez, J.J.G., Montiel, J.M.M., Tard ́os, J.D., 2021. ORBSLAM3: an accurate open-source library for visual, visual–inertial, and multimap SLAM. IEEE Trans. Rob. 2021, 1874–1890.

    Chatterjee, A., Govindu, V.M., 2013. Efficient and robust large-scale rotation averaging. In: Proceedings of the IEEE International Conference on Computer Vision (ICCV), pp. 521–528.

    Crandall, D., Owens, A., Snavely, N., Huttenlocher, D., 2011. Discrete-continuous optimization for large-scale structure from motion. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR), pp. 3001–3008.

    Cui, Z., Tan, P., 2015. Global structure-from-motion by similarity averaging. In: Proceedings of the IEEE International Conference on Computer Vision (ICCV), pp. 864–872.

    Davison, A.J., 2003. Real-Time Simultaneous Localisation and Mapping with a Single Camera. In: Proceedings of the IEEE International Conference on Computer Vision (ICCV).

    Engel, J., Sch ̈ops, T. and Cremers, D., 2014. LSD-SLAM: Large-scale direct monocular SLAM. In: Proceedings of European Conference on Computer Vision, pp. 834–849.

    Engel, J., Koltun, V., Cremers, D., 2018. Direct sparse odometry. IEEE Trans. Pattern Anal. Mach. Intell., 40 (3), 611–625.

    Farenzena, M., Fusiello, A., Gherardi, R., 2009. Structure-and-motion pipeline on a hierarchical cluster tree. In: Proceedings of the IEEE International Conference on Computer Vision (ICCV) Workshop, pp. 1489–1496.

    Forster, C., Pizzoli, M., and Scaramuzza, D., 2014. SVO: Fast semi-direct monocular visual odometry. In: Proceedings of IEEE International Conference on Robotics and Automation (ICRA), pp. 15–22.

    Geiger, A., Lenz, P., and Urtasun, R., 2012. Are we ready for autonomous driving? The KITTI vision benchmark suite. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition, pp. 3354–3361.

    Geneva, P., Eckenhoff, K., Lee, W., Yang, Y., Huang, G., 2020. OpenVINS: a research platform for visual-inertial estimation. In: Proceedings of IEEE Int. Conf. Robot. Autom. (ICRA), pp. 4666–4672.

    Grisetti, G., Kümmerle, R., Stachniss, C., Burgard, W., 2010. A tutorial on graph-based SLAM. IEEE Intell. Transp. Syst. Mag. 2 (4), 31–43.

    Hartley, R., Aftab, K., Trumpf, J., 2011. L1 rotation averaging using the Weiszfeld algorithm. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR), vol. 1, pp. 3041–3048.

    Hartmann, W., Havlena, M., and Schindler, K., 2014. Predicting Matchability. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition, pp. 9–16.

    Havlena, M., Schindler, K., 2014. VocMatch: Efficient Multiview correspondence for structure from motion. In: Proceedings of the European Conference on Computer Vision. ECCV, pp. 46–60.

    Hinzmann, T., Schönberger, J.L., Pollefeys, M., Siegwart, R., 2018. In: Hutter, M., Siegwart, R. (Eds.), Field and Service Robotics. Springer Proceedings in Advanced Robotics. Springer, Cham, pp. 383–396.

    Hoppe, C., Klopschitz, M., Rumpler, M., Wendel, A., Kluckner, S., Bischof, H., Reitmayr, G., 2012. Online feedback for Structure-from-Motion Image Acquisition. In: Proceedings of the British Machine Vision Conference (BMVC).

    Hou, Q., Xia, R., Zhang, J., Feng, Y., Zhan, Z.Q., Wang, X., 2023. Learning visual overlapping image pairs for SfM via CNN fine-tuning with photogrammetric geometry information. Int. J. Appl. Earth Obs. Geoinformation 103162.

    Huang, J., Huang, S., Sun, M.W., 2021. DeepLM: Large-scale Nonlinear Least Squares on Deep Learning Frameworks using Stochastic Domain Decomposition. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition. CVPR, pp. 10303–10312.

    Jiang, N., Cui, Z., Tan, P., 2013. A global linear method for camera pose registration. In: Proceedings of the IEEE International Conference on Computer Vision (ICCV), pp. 481–488.

    Jiang, S., Jiang, W.S., 2020. Efficient match pair selection for oblique UAV images based on adaptive vocabulary tree. ISPRS J. Photogramm. Remote Sens. 161, 61–75.

    Jiang, S., Jiang, C., Jiang, W.S., 2020. Efficient structure from motion for large-scale UAV images: a review and a comparison of SfM tools. ISPRS J. Photogramm. Remote Sens. 167, 230–251.

    Jiang, S., Jiang, W.S., Wang, L.Z., 2021. Unmanned aerial vehicle-based photogrammetric 3D mapping: a survey of techniques, applications, and challenges. IEEE Trans. Geosci. Remote Sens. Magazine 42 (2), 135–171.

    Jin, Y., Mishkin, D., Mishchuk, A., Matas, J., Fua, P., Yi, K.M., Trulls, E., 2021. Image matching across wide baselines: from paper to practice. Int. J. Comput. Vis. 129 (2), 517–547.

    Kern, A., Bobbe, M., Khedar, Y., Bestmann, U., 2020. OpenREALM: Real-time Mapping for Unmanned Aerial Vehicles. In: 2020 International Conference on Unmanned Aircraft Systems. ICUAS, Greece Athens, pp. 902–911.

    Klein, G. and Murray, D., 2007. Parallel tracking and mapping for small AR workspaces. In: Proc. 6th IEEE ACM Int. Symp. Mixed Augmented Reality, pp. 225–234.

    Lepetit, V., Moreno-Noguer, F., Fua, P., 2009. EPnP: An accurate O(n) solution to the PnP problem. Int. J. Comput. Vis. 81, 155–166.

    Li, D.J., Shi, X.S., Long, Q.W., Liu, S. H., Yang, W., Wang, F. S., Wei, Q., Qiao, F., 2020. DXSLAM: A Robust and Efficient Visual SLAM System with Deep Features. In: Proceedings of IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS), pp. 4958-4965.

    Liang, Y.B., Yang, Y., Fan, X.C., Cui, T.J., 2023. Efficient and accurate hierarchical SfM based on adaptive track selection for large-scale oblique images. Remote Sens. (Basel) 15 (5), 1374.

    Locher, A., Perdoch, M., Riemenschneider, H., Gool, V.L., 2016. Mobile phone and cloud — A dream team for 3D reconstruction. In: Proceedings of the IEEE Winter Conference on Applications of Computer Vision (WACV), pp. 1-8.

    Mayer, H., 2014. Efficient hierarchical triplet merging for camera pose estimation. In: Proceedings of German Conf. on Pattern Recognition, pp.99–409.

    Mayer, H., 2019. RPBA-Robust parallel bundle adjustment based on covariance information. In arXiv Preprint arXiv 1910, 08138.

    Malkov, Y.A., Yashunin, D.A., 2020. Efficient and robust approximate nearest neighbor search using hierarchical navigable small world graphs. IEEE Trans. Pattern Anal. Mach. Intell. 42 (4), 824–836.

    Menna, F., Nocerino, E., Remondino, F., Saladino, L., Berri, L., 2020. Towards online UAS-based photogrammetric measurements for 3D metrology inspection. Photogram. Rec. 35 (172), 467–486.

    Michelini, M., Mayer, H., 2020. Structure from motion for complex image sets. ISPRS J. Photogramm. Remote Sens. 166, 140–152.

    Moulon, P., Monasse, P., Romuald, P., Renaud, M., 2016. Open{MVG}: Open multiple view geometry. [https://github.com/openMVG/openMVG](https://github.com/openMVG/openMVG). (accessed 17.11.2023).

    Morelli, L., Bellavia, F., Menna, F., Remondino, F., 2022. Photogrammetry now and then–from hand-crafted to deep-learning tie points–. Int. Arch. Photogramm. Remote. Sens. Spat. Inf. Sci. 48, 163–170.

    Morelli, L., Ioli, F., Beber, R., Menna, F., Remondino, F., Vitti, A., 2023. COLMAP-SLAM: A framework for visual odometry. Int. Arch. Photogramm. Remote. Sens. Spat. Inf. Sci. 48, 317–324.

    Morelli, L., Menna, F., Vitti, A., Remondino, F. and Toth, C., 2023b. Performance Evaluation of Image-Aided Navigation with Deep-Learning Features. In Proceedings of the 36th International Technical Meeting of the Satellite Division of The Institute of Navigation (ION GNSS+ 2023) (pp. 2048-2056).

    Mur-Artal, R., Montiel, J.M.M., Tard́os, J.D., 2015. ORB-SLAM: a versatile and accurate monocular SLAM system. IEEE Trans. Rob. 1147–1163.

    Mur-Artal, R., Tardós, J.D., 2017. Orb-slam2: an open-source slam system for monocular, stereo, and rgb-d cameras. IEEE Trans. Robot. 33 (5), 1255–1262.

    Nocedal, J., Wright, S., 2000. Numerical optimization. Springer.

    Nocerino, E., Poiesi, F., Locher, A., Tefera, Y.T., Remondino, F., Chippendale, P., Gool, V. L., 2017. 3D reconstruction with a collaborative approach based on smartphones and a cloud-based server. Int. Arch. Photogramm. Remote Sens. Spatial Inf. Sci., XLII-2/W8, 187–194. [https://doi.org/10.5194/isprs-archives-XLII-2-W8-187-2017](https://doi.org/10.5194/isprs-archives-XLII-2-W8-187-2017).

    Poiesi, P., Locher, A., Chippendale, P., Nocerino, E., Remondino, F., Gool, V. L., 2017. Cloud-based collaborative 3D reconstruction using smartphones. In: Proceedings of the 14th European Conference on Visual Media Production (CVMP), 1, pp. 1-9.

    Pumarola, A., Vakhitov, A., Agudo, A., Sanfeliu, A. and Moreno-Noguer, F., 2017. PLSLAM: Real-time monocular visual SLAM with points and lines. In: Proceedings of IEEE Int. Conf. Robot. Autom. (ICRA), pp. 4503–4508.

    Radenovíc, F., Tolias, G., Chum, O., 2016. Cnn image retrieval learns from bow: Unsupervised fine-tuning with hard examples. In: Proceedings of the European Conference on Computer Vision (ECCV), pp. 3-20.

    Razavian, A.S., Sullivan, J., Carlsson, S., Maki, A., 2016. Visual instance retrieval with deep convolutional networks. ITE Trans. Media Technol. Appl. 4 (3), 251–258.

    Radenovíc, F., Tolias, G., Chum, O., 2019. Fine-Tuning CNN Image Retrieval with No Human Annotation. In: IEEE Transactions on Pattern Analysis and Machine Intelligence, pp. 1655-1668.

    Ren, J., Liang, W.T., Yan, R., Mai, L., Liu, S. W., Liu, X., 2022. MegBA: A GPU-Based Distributed Library for Large-Scale Bundle Adjustment. In: Proceedings of European Conference on Computer Vision, 2022.

    Scḧonberger, J.L., Frahm, J.M., 2016. Structure-from-motion revisited. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR).

    Shen, T.W., Luo, Z.X., Zhou, L., Zhang, R.Z., Zhu, S.Y., Fang, T., Quan, L., 2018. Matchable Image Retrieval by Learning from Surface Reconstruction. In: Proceedings of the Asian Conference on Computer Vision (ACCV), pp. 415-431.

    Sivic, J., Zisserman, A., 2006. Video google: A text retrieval approach to object matching in videos. In: Proceedings of the IEEE International Conference on Computer Vision (ICCV), pp. 1470-1477.

    Simonyan, K., Zisserman, A., 2014. Very deep convolutional networks for large-scale image recognition. arXiv preprint arXiv:1409.1556.

    Snavely, N., Seitz, S.M., Szeliski, R., 2006. Photo tourism: exploring photo collections in 3d. Acm Trans. Graphics 25 (3), 835–846.

    Sturm, J., Engelhard, N., Endres, F., Burgard, W., Cremers, D., 2012. A benchmark for the evaluation of RGB-D SLAM systems. Proceedings of IEEE/RSJ Int. Conf. Intell. Robots Syst. 573–580.

    Sweeney, C., 2016. Theia. [http://theia-sfm.org/](http://theia-sfm.org/) (accessed 17.11.2023).

    Tang, C., Tan, P., 2019. BA-Net: Dense Bundle adjustment networks. In: Proceedings of International Conference on Learning Representations.

    Tateno, K., Tombari, F., Laina, I., Navab, N., 2017. CNN-SLAM: Real-time dense monocular SLAM with learned depth prediction. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR), pp. 6243-6252.

    Toldo, R., Gherardi, R., Farenzena, M., Fusiello, A., 2015. Hierarchical structure-andmotion recovery from uncalibrated images. Comput. Vis. Image Underst. 140, 127–143.

    Torresani, A., Menna, F., Battisti, R., Remondino, F., 2021. A V-SLAM guided and portable system for photogrammetric applications. Remote Sens. (Basel) 13 (12), 2351.

    Untzelmann, O., Sattler, T., Middelberg, S. and Kobbelt, L., 2013. A Scalable Collaborative Online System for City Reconstruction. In: Proceedings of the IEEE Conference on Computer Vision Workshops (CVPRW), pp. 644-651.

    Wan, J., Yilmaz, A., Yan, L., 2018. DCF-BoW: build match graph using bag of deep convolutional features for structure from motion. IEEE Geosci. Remote Sens. Lett. 15 (12), 1847–1851.

    Wang, X., Rottensteiner, F., Heipke, C., 2018. Robust image orientation based on relative rotations and tie points. ISPRS Ann. Photogram. Remote Sens. Spatial Inf. Sci. IV-2, 295–302.

    Wang, X., Rottensteiner, F., Heipke, C., 2019. Structure from Motion for ordered and unordered image sets based on random k-d forests and global pose estimation. ISPRS J. Photogramm. Remote Sens. 147, 19–41.

    Wang, X., Xiao, T., Kasten, Y., 2021. A hybrid global structure from motion method for synchronously estimating global rotations and global translations. ISPRS J. Photogramm. Remote Sens. 174, 35–55.

    Wang, X., Wang, Z.W., Xu, Y.W., Zhan, Z.Q., 2024. Learning-based baseline method for efficient determination of overlapping image pairs and its application on both offline and online SfM. PFG–J. Photogramm., Remote Sens. Geoinformation Sci. [https://doi.org/10.1007/s41064-024-00312-z](https://doi.org/10.1007/s41064-024-00312-z).

    Wilson, K., Snavely, N., 2014. Robust global translations with 1DSFM. In: Proceedings of the European Conference on Computer Vision (ECCV). Springer, pp. 61–75.

    Wu, C. 2013. Towards linear-time incremental structure from motion. In: Proceedings of the IEEE Conference on 3dtv, pp. 127–134.

    Wu, C., 2016. VisualSFM. [http://ccwu.me/vsfm/](http://ccwu.me/vsfm/) (accessed 17.11.2023).

    Wu, C., Agarwal, S., Curless, B., and Seitz, S. M., 2011. Multicore bundle adjustment. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition (CVPR), pp. 3057-3064.

    Yan, S., Zhang, M.J., Lai, S.M., Liu, Y., Peng, Y., 2021. Image retrieval for structure-frommotion via graph convolutional network. Inf. Sci. 573, 20–36.

    Yi, K.M., Trulls, E., Lepetit, V., Fua, P., 2016. LIFT: Learned Invariant Feature Transform. In: Proceedings of the European Conference on Computer Vision (ECCV). Springer, pp. 467–483.

    Zhan, Z.Q., Wang, X., Wei, M.L., 2015. Fast method of constructing image correlations to build a free network based on image multivocabulary trees. J. Electron. Imaging 24 (3), 033029.

    Zhang, R.Z., Zhu, S.Y., Fang, T., Quan, L., 2017. Distributed Very Large Scale Bundle Adjustment by Global Camera Consensus. In: Proceedings of IEEE International Conference on Computer Vision, pp. 29-38.

    Zhan, Z.Q., Xia, R., Yu, Y.F., Xu, Y.B., Wang, X., 2024. On-the-Fly SfM: What you capture is What you get. ISPRS Ann. Photogramm. Remote Sens. Spatial Inf. Sci., X-1-2024 297–304. [https://doi.org/10.5194/isprs-annals-X-1-2024-297-2024](https://doi.org/10.5194/isprs-annals-X-1-2024-297-2024).

    Zhao, L., Huang, S.D., Dissanayake, G., 2018. Linear SfM: A hierarchical approach to solving structure-from-motion problems by decoupling the linear and nonlinear components. ISPRS J. Photogramm. Remote Sens. 141, 275–289.

    Zhao, Y., Chen, L., Zhang, X.S., Bu, S.H., Jiang, H.K., Han, P.C., Li, K., Wan, G., 2022. RTSfM: real-time structure from motion for Mosaicing and DSM mapping of sequential aerial images with low overlap. IEEE Trans. Geosci. Remote Sens. 1–15.

    Zheng, M.T., Zhou, S.P., Xiong, X.D., Zhu, J.F., 2017. A new GPU bundle adjustment method for large-scale data. Photogramm. Eng. Remote Sens. 23–31.

    Zheng, M.T., Chen, N.C., Zhu, J.F., 2023. Distributed bundle adjustment with blockbased sparse matrix compression for super large scale datasets. In: Proceedings of the IEEE International Conference on Computer Vision, pp. 18152-18162.

    Zhuang, B.B., Cheong, L.F., Lee, G.H., 2018. Baseline desensitizing in translation averaging. In: Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR).

    Zhu, J.G., Ye, S.H., Yang, X.Y., Qu, X.H., Liu, C.J., Wu, B., 2005. On-line industrial 3D measurement techniques for large volume objects. Key Eng. Mater. 295–296, 423–430.