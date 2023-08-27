---
title: Improving the efficiency of hierarchical structure-and-motion
date: 2023-08-22 20:47:31
status: new
---


!!! info "论文链接"
	原文：Riccardo Gherardi, Michela Farenzena, Andrea Fusiello. [Improving the efficiency of hierarchical structure-and-motion](https://doi.org/10.1109/CVPR.2010.5539782)

    关联：Riccardo Gherardi, Michela Farenzena, Andrea Fusiello. [Structure-and-motion pipeline on a hierarchical cluster tree](https://doi.org/10.1109/ICCVW.2009.5457435)

!!! warning "注意"
    由 claude 和 chatgpt 翻译整理

## 1 绪论

主要的挑战是计算效率（针对越来越多的影像）和通用性问题。

针对计算效率：主要 BA 和特征提取占主要时间

- 分块方法（Partitioning Methods）：将重建问题简化为更小更良态的子问题（更有效地优化）
- 选择影像集子集：
	- 适合视频序列
	- 最近的方法
		- 依然是增量处理
		- 还要计算每对影像之间的极线几何
- 投入更多算力
	- 负载均衡
	- 单管道转身并发
- 层次化
	- 降低了计算复杂度
	- 对于顺序方法中的典型问题更不敏感
		- 初始化和漂移问题

通用性问题

- 针对输入影像存在一些推测假设
	- 也即一些辅助信息
	- 内参数、EXIF、外部信息
- 自动标定提出已有几年，但没有针对 SaM 管道的无辅助信息标定参数的工作
## 关键点匹配

标准流程

## 视图聚类

组织为一个分层聚类结构（树），保持树的平衡很重要

- 定义亲和力矩阵，由匹配点的一致程度和匹配点分布面积的重合程度共同决定
- 构建二叉树，采用自底而上的策略进行聚类合并，聚类距离由最近的两个元素决定
- 保持树的平衡，采用以下策略
	- 合并 $\mathcal{l}$ 个最近聚类中，基数最小的两个聚类
	- 弱化“最近优先”的聚类标准，引入“最小优先”准则
	- 平衡程程度由 $\mathcal{l}$ 确定
		-  $\mathcal{l}=1$ 标准聚类
		-  $\mathcal{l} = n/2$ 完美平衡树，聚类效果差
		- 实验中使用  $\mathcal{l}=5$ 
	- 注意避免影像对退化情况（同质）
		- 考虑 GRIC 几何鲁棒信息准则
			- 基础矩阵应当比单应矩阵要好（GRIC 得分更低）
## 层次结构和运动重建

重建从未校准状态开始，一旦未校准的聚类达到给定维数 m，就会触发欧几里德升级过程

### 两视图重建

两个视图进行的重建始终是投影的，并且是基于基本矩阵进行的

#### 三角测量
迭代线性最小二乘方法，通过分析线性系统的条件数和投影误差，对点进行修剪

### 单视图添加

在要添加的视图中可见的重建 3D 点提供了一组 3D-2D 对应关系，这将被用于添加视图的单视图重建。

为了处理异常值，会使用 MSAC。被连接的视图可能引入了一些新的轨迹，这些轨迹将在后续进行三角测量

### 聚类合并
两个聚类合并时，他们共有的点是用于计算未知转换的关键点，通过使用 MSAC 来丢弃错误匹配。
当合并一个欧几里德聚类和一个投影聚类时，会寻找投影矩阵的单应性，将其转换为欧几里德基础，然后进行相机注册。


## 自动校准

利用双重绝对二次曲线（DIAQ）的自动校准。通过对 DIAQ 应用约束，实现相机参数的自动校准，重建从投影升级到欧几里德级别。

## 参考文献

??? info "References"

	[1] S. Agarwal, N. Snavely, I. Simon, S. M. Seitz and R. Szeliski, "Building rome in a day", International Conference on Computer Vision, 2009.

	[2] M. Brown and D. Lowe, "Recognising panoramas", Proceedings of the 9th International Conference on Computer Vision, vol. 2, pp. 1218-1225, October 2003.

	[3] M. Brown and D. G. Lowe, "Unsupervised 3D object recognition and reconstruction in unordered datasets", Proceedings of the International Conference on 3D Digital Imaging and Modeling, June 2005.

	[4] [online] Available: http://phototour.cs.washington.edu/bundler/.

	[5] N. Cornelis, B. Leibe, K. Cornelis and L. V. Gool, "3D urban scene modeling integrating recognition and reconstruction", International Journal of Computer Vision, vol. 78, no. 2–3, pp. 121-141, July 2008.

	[6] M. Farenzena, A. Fusiello and R. Gherardi, "Structure-and-motion pipeline on a hierarchical cluster tree", IEEE International Workshop on 3-D Digital Imaging and Modeling, October 3–4 2009.

	[7] P. D. Fiore, "Efficient linear solution of exterior orientation", IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 23, no. 2, pp. 140-148, 2001.

	[8] A. W. Fitzgibbon and A. Zisserman, "Automatic camera recovery for closed and open image sequences", Proceedings of the European Conference on Computer Vision, pp. 311-326, 1998.

	[9] S. Gibson, J. Cook, T. Howard, R. Hubbold and D. Oram, "Accurate camera calibration for off-line video-based augmented reality", Mixed and Augmented Reality IEEE/ACM International Symposium on, 2002.

	[10] F. Hampel, P. Rousseeuw, E. Ronchetti and W. Stahel, Robust Statistics: the Approach Based on Influence Functions. Wiley Series in probability and mathematical statistics, John Wiley & Sons, 1986.

	[11] R. Hartley and A. Zisserman, Multiple View Geometry in Computer Vision, Cambridge University Press, 2003.

	[12] R. I. Hartley and P. Sturm, "Triangulation", Computer Vision and Image Understanding, vol. 68, no. 2, pp. 146-157, November 1997.

	[13] N. J. Higham, "Computing a nearest symmetric positive semidefinite matrix", Linear Algebra and its Applications, vol. 103, pp. 103-118, 1988.

	[14] A. Irschara, C. Zach and H. Bischof, "Towards wiki-based dense city modeling", Proceedings of the 11th International Conference on Computer Vision, 1–8, 2007.

	[15] G. Kamberov, G. Kamberova, O. Chum, S. Obdrzalek, D. Martinec, J. Kostkova, et al., "3D geometry from uncalibrated images", Proceedings of the 2nd International Symposium on Visual Computing, November 6–8 2006.

	[16] M. Lourakis and A. Argyros, "The design and implementation of a generic sparse bundle adjustment software package based on the Levenberg-Marquardt algorithm", Technical Report 340 Institute of Computer Science - FORTH Heraklion Crete Greece, August 2004.

	[17] K. Ni, D. Steedly and F. Dellaert, "Out-of-core bundle adjustment for large-scale 3D reconstruction", Proceedings of the International Conference on Computer Vision, 1–8, 2007.

	[18] D. Nistér, "Reconstruction from uncalibrated sequences with a hierarchy of trifocal tensors", Proceedings of the European Conference on Computer Vision, pp. 649-663, 2000.

	[19] M. Pollefeys, R. Koch and L. Van Gool, "Self-calibration and metric reconstruction in spite of varying and unknown internal camera parameters", Proceedings of the International Conference on Computer Vision, pp. 90-95, 1998.

	[20] M. Pollefeys, F. Verbiest and L. V. Gool, "Surviving dominant planes in uncalibrated structure and motion recovery", Proceedings of the European Conference on Computer Vision, pp. 837-851, 2002.

	[21] F. Schaffalitzky and A. Zisserman, "Multi-view matching for unordered image sets or “how do I organize my holiday snaps?", Proceedings of the 7th European Conference on Computer Vision, pp. 414-431, 2002.

	[22] Y. Seo, A. Heyden and R. Cipolla, "A linear iterative method for auto-calibration using the DAC equation", Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, vol. 1, pp. 880, 2001.

	[23] H.-Y. Shum, Q. Ke and Z. Zhang, "Efficient bundle adjustment with virtual key frames: A hierarchical approach to multi-frame structure from motion", Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, June 1999.

	[24] N. Snavely, S. M. Seitz and R. Szeliski, "Photo tourism: exploring photo collections in 3D", SIGGRAPH: International Conference on Computer Graphics and Interactive Techniques, pp. 835-846, 2006.

	[25] N. Snavely, S. M. Seitz and R. Szeliski, "Skeletal graphs for efficient structure from motion", Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, 1–8, 2008.

	[26] D. Steedly, I. Essa and F. Dellaert, "Spectral partitioning for structure from motion", Proceedings of the International Conference on Computer Vision, pp. 649-663, 2003.

	[27] T. Thormählen, H. Broszio and A. Weissenfeld, "Keyframe selection for camera motion and structure estimation from multiple views", Proceedings of the European Conference on Computer Vision, pp. 523-535, 2004.

	[28] P. H. S. Torr, "An assessment of information criteria for motion model selection", Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pp. 47-53, 1997.

	[29] P. H. S. Torr and A. Zisserman, "MLESAC: A new robust estimator with application to estimating image geometry", Computer Vision and Image Understanding, vol. 78, no. 2000, 2000.

	[30] B. Triggs, "Autocalibration and the absolute quadric", Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pp. 609-614, 1997.

	[31] M. Vergauwen and L. V. Gool, "Web-based 3D reconstruction service", Machine Vision and Applications, vol. 17, no. 6, pp. 411-426, 2006.