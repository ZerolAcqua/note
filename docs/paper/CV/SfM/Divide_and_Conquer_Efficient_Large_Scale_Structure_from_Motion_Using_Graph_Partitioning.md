---
title: Divide and Conquer Efficient Large-Scale Structure from Motion Using Graph Partitioning
date: 2023-10-19 16:53:31
tags:
  - CV/SfM
rating: ⭐
status: new
---

# Divide and Conquer Efficient Large-Scale Structure from Motion Using Graph PartitioningDivide and Conquer Efficient Large-Scale Structure from Motion Using Graph Partitioning

!!! info "论文链接"
    Brojeshwar Bhowmick, Suvam Patra, Avishek Chatterjee, Venu Madhav Govindu, Subhashis Banerjee [Divide and Conquer: Efficient Large-Scale Structure from Motion Using Graph Partitioning](https://link.springer.com/10.1007/978-3-319-16808-1_19)

!!! warning "注意"
    在 chatgpt 辅助下翻译整理



## 1 引言
- 捆绑调整是对所有相机和 3D 点的联合优化，通常不适用于大数据集
	- 累积误差
	- 相机与 3D 特征点弱连接
	- 捆绑调整也需要很高的计算需求和时间
- 本文采用一种分治策略，旨在减轻这些问题
	- 将完整的图像数据集分为较小的集合
	- 每个集合使用标准的捆绑调整方法进行独立重建
	- 利用各个分区之间的相机间的几何关系，解决全局注册问题，正确准确地将每个独立的 3D 重建成分置入单一的全局参考框架中
- 这种方法不仅在重建失败方面更加稳健，而且在计算速度方面也显著优于最先进的技术
- 本文的主要贡献
	1. 基于归一化割【7】的原则方法
		- 将大量图像的匹配图分成可以独立和可靠重建的不相交连接组件
	2. 使用成对极点几何关系来注册与独立连接组件对应的点云的方法
		- 本文提出的基于极点的注册技术比使用 3D-3D 或 3D-2D 对应关系来注册点云的标准技术更稳健
			- 基于 3D 点对应的注册方法不使用所有可用信息（图像对应关系）
			- 基于 3D-2D 的方法，例如顺序绑定器【1,2,8】，通常会导致在点的数量不足以进行重新部分分割或在离群值拒绝阶段【1】删除共同的 3D 点时导致分断的重建（见表4）。
- 其他论文尝试中最大数据集只有 380 张图像，使用共同的3D点的重投影误差来合并是不适用于非常大的数据集的。
- 我们的方法中
	- 将图像集分解成较小的组件，以便每个组件的匹配图是密集连接的
	- 每个组件的基于 SfM 的重建可以并行进行
## 2 使用标准化割进行数据集分解

- 有组织的方式获取的图像，容易分解为较小集合
- 非组织数据集需要建立一种自动分组成视觉相似集并找到连接图像的方法
	- 使用词汇树形成一个匹配图
		- 每个节点都是一个图像
		- 节点之间的边权重是从词汇树获得的相似值
- 使用标准化割（NC）公式的多向扩展【7】来自动将匹配图 $G =(V, E)$ 分成单独的集群
	- 由于边权重基于视觉相似性，标准化割可得到视觉上相似的连接成分。
	- 图权除了基于成对图像特征相似度分数，还可合并几何信息（如连接图像的成对极点几何的稳健性），来改善图的质量
- 提取连接图像
	- 候选连接图像的数量通常非常大。减少连接图像的数量将减少估算成对极点几何的时间
	- 连接图像提取过程
		1. 对于每个连接图像，使用极点计算的稳健性度量来拒绝异常值出边（在成分内和成分间都 计算？）。
		2. 如果保留的出边数量小于 $T$（通常 $T$ = 原始出度的 60%），则从连接图像集中删除图像。
		3. 计算当前图像的所有保留出边的相似性分数的平均值。
		4. 如果割边的相似性分数超过它们连接的图像的平均相似性值，然后将图像标记为连接图像。


## 3 独立成分重建的注册
- 注册一对 3D 重建，需要估算它们之间的相对变换
- 使用已重建的相机和连接相机之间的极线关系来估算一对重建之间的相对旋转、平移和尺度
- 考虑两个独立重建的相机组 $A$ 和 $B$，让 $\mathbb{C}_{AB}$ 是 $A$ 和 $B$ 之间的连接相机集
	- 大写字母表示相机组，小写字母表示独立的相机
	- 确定 $A$ 和 $B$ 之间的相对尺度
	- 再通过刚性或欧几里德变换关联

### 3.1 一对重建之间相对尺度估算
- 估算 $A$ 和 $B$ 之间的相对尺度
	- 分别估算两者之间所有的连接相机（$k \in \mathbb{C}_{AB}$）在 $A$ 和 $B$ 的本地参考框架中的位置
	- 估算两个重建的相对比例关系
		- 如果连接相机 $k \in \mathbb{C}_{AB}$ 与 $A$ 中的相机有相同特征，那么 $k$ 的旋转和平移可以在 $A$ 的本地参考框架中找到。
		- 设 $k$ 相对于 $A$ 的参考框架的未知旋转和平移分别表示为 $R_{Ak}$ 和 $T_{Ak}$ 。
		- 考虑到 $i$ 是属于组 $A$ 的相机。假设 $i$ 相对于 $A$ 的本地参考框架的旋转和平移分别为 $R_{Ai}$ 和 $T_{Ai}$（在 $A$ 内估算）
		- 如果 $i \in A$ 且 $k \in \mathbb{C}_{AB}$ 并有相同特征，那么使用极线关系，我们可以找到 $i$ 和 $k$ 之间的相对旋转（$R_{ik}$）和相对平移方向（$t_{ik}$）
		- 显然，以下关系应该成立：
			
			$$R_{ik}=R_{Ak}R_{Ai}^T\Rightarrow R_{Ak}=R_{ik}R_{Ai}$$
            
            $$t_{ik}\propto T_{Ak}-R_{ik}T_{Ai}\Rightarrow[t_{ik}]_{\times}(T_{Ak}-R_{ik}T_{Ai})=0$$

			- 这里，$[\cdot]_\times$ 是向量叉积的反对称矩阵表示【20】。这些关系对于所有 $i\in A$ 都成立，以便 $i$ 和 $k$ 共享共同特征，并且可以估算它们的极线几何关系
			- 可以用所有这些旋转旋转 $R_{Ak}$ 估算的均值【21】来表示
				
				$$\widehat{R}_{Ak}=
\operatorname*{mean}_{i\in A}\left(R_{ik}R_{Ai}\right)$$

			- 类似地，可以得到相对平移 $T_{Ak}$ 的平均估算，如下所示：
			
				$$\widehat{T}_{Ak}=\underset{T_{Ak}}{\operatorname*{\mathrm{argmin}}}\sum_{i\in A}\frac{\left\|\left[t_{ik}\right]_{\times}\left(T_{Ak}-R_{ik}T_{Ai}\right)\right\|^2}{\left\|T_{Ak}-R_{ik}T_{Ai}\right\|^2}$$

				其中，可以使用【22】中提出的迭代方法来解决。

		- 在 $A$ 的参考框架中，相机 $k$ 的投影中心由 $-\widehat{R}_{Ak}\widehat{T}_{Ak}$ 给出。因此，在 $A$ 的参考框架中计算所有 $k \in \mathbb{C}_{AB}$ 的相机中心 $-\widehat{R}_{Ak}\widehat{T}_{Ak}$ 。类似地，在 $B$ 的参考框架中计算所有 $k \in \mathbb{C}_{AB}$ 的相机中心 $-\widehat{R}_{Bk}\widehat{T}_{Bk}$ 。然后，可以通过比较两个重建中共同相机的成对距离来稳健地估算 $A$ 和 $B$ 之间的相对尺度，如下所示：
			
			$$\widehat s_{AB}=\underset{k_1,k_2\in\mathbb{C}_{AB}}{\operatorname*{median}}\frac{\left\|-\widehat R_{Bk_1}\widehat T_{Bk_1}+\widehat R_{Bk_2}\widehat T_{Bk_2}\right\|}{\left\|-\widehat R_{Ak_1}\widehat T_{Ak_1}+\widehat R_{Ak_2}\widehat T_{Ak_2}\right\|}$$
			
			

	- 估算出 $A$ 和 $B$ 之间的相对尺度后，将 $A$ 的重建按 $B$ 的尺度进行缩放
		- 相机 $k$ 在 $A$ 的本地参考框架中的旋转仍然 $\widehat{R}_{Ak}$
		- 相机 $k$ 的平移在 $A$ 的缩放在本地参考框架中变为 $\widehat{s}_{AB}\widehat{T}_{Ak}$
		- 为了去除极线几何估算的平移方向存在的异常值，我们检查基本矩阵的两个非零特征值是否具有相似的值[20]
			- 如果两个最大特征值（按排序顺序的 $sigma_2$ 和 $\sigma_1$）的比值小于阈值，则丢弃估算的基本矩阵以及相应的平移方向，即 $\dfrac{\sigma_2}{\sigma_1} <T$。通常情况下，在实验中采用 $T=0.95$。

### 3.2 一对重建之间相对旋转和平移的估算
- $A$ 被调整到与 $B$ 相同的尺度下后，两个重建之间可通过刚性或欧几里德变换联系

	- $k$ 相对于 $A$ 的运动为旋转和平移，分别 $\widehat{R}_{Ak}$ 和 $\widehat{s}_{AB}\widehat{T}_{Ak}$
	- $k$ 在 $B$ 的参考框架中的运动为 $\widehat{R}_{Bk}$ 和 $\widehat{T}_{Bk}$
- 将 3D 旋转和平移表示为紧凑的欧几里德运动模型
	
    $$\left.\mathbf M=\left[\begin{array}{c|c}R&T\\\hline\mathbf0&1\end{array}\right.\right]$$

	其中 $\mathbf0{0}$ 表示一个 $1\times 3$ 零向量。

- 假设将 $A$ 和 $B$ 对齐到全局参考框架所需的未知运动分别是 $M_A$ 和 $M_B$。应用这些变换到 $A$ 和 $B$ 后，$A$ 和 $B$ 之间的所有共同连接相机（ $k \in \mathbb{C}_{AB}$）应该具有相同的运动参数。因此，在 $A$ 和 $B$ 与全局参考框架对齐后，有
	
	$$\widehat{\mathrm{M}}_{Ak}\mathrm{M}_A=\widehat{\mathrm{M}}_{Bk}\mathrm{M}_B$$


	需要强调的是，$\widehat{\mathrm{M}}_{Ak}$ 的平移分量是缩放版本，如 $\widehat{s}_{AB}\widehat{T}_{Ak}$
	因此，$A$ 和 $B$ 之间的相对运动为

	$$\mathrm{M}_{AB}=\mathrm{M}_{B}\mathrm{M}_{A}^{-1}=\mathrm{\widehat{M}}_{Bk}^{-1}\mathrm{\widehat{M}}_{Ak}$$

	从上式中，我们可以看到
	
	$$R_{AB}=R_BR_A^T=\widehat{R}_{Bk}^T\widehat{R}_{Ak}$$
	
- 如果 $A$ 和 $B$ 之间有许多连接相机，那么就有许多 $R_{AB}$ 的估算，取平均

	$$\widehat{R}_{AB}=\operatorname*{\mathrm{mean}}_{k\in\mathbb{C}_{AB}}\left(\widehat{R}_{Bk}^T\widehat{R}_{Ak}\right)$$
 
	类似地，$A$ 和 $B$ 之间的相对平移可以表示为

	$$T_{AB}=T_B-R_BR_A^TT_A=\widehat s_{AB}\widehat R_{Bk}^T\widehat T_{Ak}-\widehat R_{Bk}^T\widehat T_{Bk}$$

	由多个估算得到 $A$ 和 $B$ 之间的平均相对平移

	$$\widehat{T}_{AB}=\underset{T}{\operatorname*{argmin}}\sum_{k\in\mathbb{C}_{AB}}\left\|T-\left(\widehat{s}_{AB}\widehat{R}_{Bk}^T\widehat{T}_{Ak}-\widehat{R}_{Bk}^T\widehat{T}_{Bk}\right)\right\|_1$$

- 使用具有最大图像数量的最大重建（种子）开始全局注册过程，注册所有与此种子相连的其他重建，并将它们合并成一个单一模型
- 注册与当前模型相连的独立重建所需的运动模型可以并行估算。

## 4 实验结果
- 为了重建每个单独的图像集中的图像，我们使用VSFM 【1】作为迭代捆绑器。
- 在有组织的图像数据集上 **Hampi**
	- 比 VSFM 快得多之外，结果在质量上与 VSFM 获得的结果也相似
- 对于无组织图像数据集的实验 **Hampi**
	- 使用 SIFT 【19,23】特征训练了一个词汇树【18】
	- 实验中连接组件的期望数量是凭直觉决定的
	- 在平面视图的左上角附近特别是边缘分割的图像的结果略微优越
- 标准无组织数据集上测试 **Central Rome**
	- VSFM 无法对这个数据集进行重建
- **St Peter's Basilica** 和 **Colosseum** 数据集
	 - 在大多数情况下，由于抢占式匹配导致重建中断，我们不得不在  VSFM 中使用所有同名点对匹配
	- 我们方法的重建和总注册所需的时间远远少于 VSFM 的重建时间，我们取得的整体加速至少优于一个数量级
	- 迭代束调整方案通常会导致在成分内部重建中断。
    
## 5 结论

我们提出了一种从大量图像进行自动 3D 重建的新流程。我们已经证明了将图像划分为可以独立并可靠重建，然后在全局参考框架中对齐集群的实用性。对许多大型数据集的结果表明，与现有技术相比，我们的方法能够在不损失显著准确性的情况下显著提高速度。


## 参考文献

??? info "References"

    【1】 Wu, C.: Towards linear-time incremental structure from motion. In: Proceedings of the International Conference on 3D Vision, 3DV 2013, pp. 127–134 (2013)

    【2】 Agarwal, S., Snavely, N., Seitz, S.M., Szeliski, R.: Bundle adjustment in the large. In: Daniilidis, K., Maragos, P., Paragios, N. (eds.) ECCV 2010, Part II. LNCS, vol. 6312, pp. 29–42. Springer, Heidelberg (2010)

    【3】 Snavely, N., Seitz, S., Szeliski, R.: Modeling the world from internet photo collections. Int. J. Comput. Vis. 80, 189–210 (2008)

    【4】 Snavely, N., Seitz, S., Szeliski, R.: Photo tourism: exploring photo collections in 3D. In: Proceedings of ACM SIGGRAPH, pp. 835–846 (2006)

    【5】 Crandall, D.J., Owens, A., Snavely, N., Huttenlocher, D.P.: Discrete-continuous optimization for large-scale structure from motion. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition, pp. 3001–3008 (2011)

    【6】 Triggs, B., McLauchlan, P.F., Hartley, R.I., Fitzgibbon, A.W.: Bundle adjustment – a modern synthesis. In: Triggs, B., Zisserman, A., Szeliski, R. (eds.) ICCV-WS 1999. LNCS, vol. 1883, pp. 298–372. Springer, Heidelberg (2000)

    【7】 Shi, J., Malik, J.: Normalized cuts and image segmentation. IEEE Trans. Pattern Anal. Mach. Intell. 22, 888–905 (2000)

    【8】 Wu, C., Agarwal, S., Curless, B., Seitz, S.: Multicore bundle adjustment. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition, pp. 3057–3064 (2011)

    【9】 Frahm, J.-M., Fite-Georgel, P., Gallup, D., Johnson, T., Raguram, R., Wu, C., Jen, Y.-H., Dunn, E., Clipp, B., Lazebnik, S., Pollefeys, M.: Building rome on a cloudless day. In: Daniilidis, K., Maragos, P., Paragios, N. (eds.) ECCV 2010, Part IV. LNCS, vol. 6314, pp. 368–381. Springer, Heidelberg (2010)

    【10】 Raghuram, R., Wu, C., Frahm, J., Lazebnik, S.: Modeling and recognition of landmark image collections using iconic scene graphs. Int. J. Comput. Vis. 95, 213–239 (2011)

    【11】 Agarwal, S., Snavely, N., Simon, I., Seitz, S., Szeliski, R.: Building rome in a day. In: Proceedings of the International Conference on Computer Vision, pp. 72–79 (2009)

    【12】 Snavely, N., Seitz, S., Szeliski, R.: Skeletal graphs for efficient structure from motion. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition, pp. 1–8 (2008)

    【13】 Havlena, M., Torii, A., Pajdla, T.: Efficient structure from motion by graph optimization. In: Daniilidis, K., Maragos, P., Paragios, N. (eds.) ECCV 2010, Part II. LNCS, vol. 6312, pp. 100–113. Springer, Heidelberg (2010)

    【14】 Crandall, D.J., Owens, A., Snavely, N., Huttenlocher, D.P.: SfM with MRFs: discrete-continuous optimization for large-scale reconstruction. IEEE Trans. Pattern Anal. Mach. Intell. 35, 2841–2853 (2013)

    【15】 Moulon, P., Monasse, P., Marlet, R.: Global fusion of relative motions for robust, accurate and scalable structure from motion. In: Proceedings of IEEE International Conference on Computer Vision, pp. 3248–3255 (2013

    【16】 Jiang, N., Cui, Z., Tan, P.: A global linear method for camera pose registration. In: Proceedings of IEEE International Conference on Computer Vision, pp. 481–488 (2013)

    【17】 Farenzena, M., Fusiello, A., Gherardi, R.: Structure-and-motion pipeline on a hierarchical cluster tree. In: Proceedings of IEEE International Conference on Computer Vision Workshop on 3-D Digital Imaging and Modeling, pp. 1489–1496 (2009)

    【18】 Nister, D., Stewenius, H.: Scalable recognition with a vocabulary tree. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition, vol. 2, pp. 2161–2168 (2006)

    【19】 Wu, C.: SiftGPU: a GPU implementation of scale invariant feature transform (SIFT) (2007). http://cs.unc.edu/ccwu/siftgpu

    【20】 Hartley, R., Zisserman, A.: Multiple View Geometry in Computer Vision, 2nd edn. Cambridge University Press, New York (2004)

    【21】 Govindu, V.M.: Lie-algebraic averaging for globally consistent motion estimation. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition (2004)

    【22】 Govindu, V.: Combining two-view constraints for motion estimation. In: Proceedings of IEEE Conference on Computer Vision and Pattern Recognition, pp. 218–225 (2001)

    【23】 Lowe, D.: Distinctive image features from scale-invariant keypoints. Int. J. Comput. Vis. 60, 91–110 (2004)
