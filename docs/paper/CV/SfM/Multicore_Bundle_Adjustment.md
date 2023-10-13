---
title: Multicore Bundle Adjustment
date: 2023-10-07 16:11:56
status: new
---


!!! info "论文链接"
	原文：Changchang Wu, Sameer Agarwal, Brian Curless, Steven M. Seitz. [Multicore bundle adjustment](https://doi.org/10.1109/CVPR.2011.5995552)


##  1 介绍


CPU 与 GPU 加速能达到  SOTA 的 10 和 30 倍

## 2 理论背景

- 光束法平差的任务
	- 根据同名影像上对应特征的位置（坐标），计算三维点的坐标、相机参数，使重投影误差最小。
	- 可转化为非线性最小二乘问题

		$$x^*=\arg\min_x\sum_{i=1}^k\|f_i(x)\|^2$$

		$x$ 为参数向量，$f(x)=[f_1(x),\ldots,f_k(x)]$ 为三维重建的重投影误差向量

- Levenberg-Marquardt (LM) 算法
	
	- 是一种流行的非线性最小二乘解算方法
	- 基础公式

		$J(x)$  为 $f(x)$ 的 Jacobian 矩阵，每次 LM 算法的迭代都在求解以下线性最小二乘问题

		$$ \delta^*=\arg \min_\delta\|J(x)\delta+f(x)\|^2+\lambda\|D(x)\delta\|^2 $$

		如果 $\|f(x+\delta^*)\|<\|f(x)\|$ ，则进行更新$x\leftarrow x+\delta^*$ 
		$D(x)$ 是非负对角矩阵，通常是 $J(x)^{T}J(x)$ 对角元素的平方根$\lambda$ 是控制正则化强度的非负参数。正则化是为了保证算法收敛$\lambda$ 的值由 $J(x)$ 对 $f(x)$ 的估计近似程度决定。
		求解上式等效于求解以下法方程：

		$$(J^{T}J+\lambda D^{T}D)\delta=-J^{T}f$$
		
		$H_{\lambda}=J{T}J+\lambda D^{T}D$ 是熟知的增广 Hessian 矩阵	

	- 未知参数分离形式

		将未知参数向量分解为相机参数 $x_c$ 和点参数 $x_p$ 向量。并令 $U=J_{c}^{T}J_{c}$，  $V=J_{p}^{T}J_{p}$，  $U_{\lambda}=U+\lambda D_{c}^{T}D_{c}$，  $V_{\lambda}^{T}=V+\lambda D_{p}^{T}D_{p}$， $W=J_{c}^{T}J_{p}$  则有
		
		$$\begin{bmatrix}U_\lambda&W\\W^T&V_\lambda\end{bmatrix}\begin{bmatrix}\delta_c\\\delta_p\end{bmatrix}=-\begin{bmatrix}J_c^Tf\\J_p^Tf\end{bmatrix}$$

	- Schur 补减少未知数

		$U_{\lambda}$ 和 $V_{\lambda}$ 是分块对角矩阵，用高斯消元法求解
		$$(U_\lambda-WV_\lambda^{-1}W^T)\delta_c=-J_c^Tf+WV_\lambda^{-1}J_p^Tf$$
		$S=U_{\lambda}-WV_{\lambda}^{-1}W^{T}$ 就是 Schur 补
		相应的，求解点参数向量

		$$\delta_{p}=-V_{\lambda}^{-1}(J_{p}^{T}f+W^{T}\delta_{c})$$

		$S$  是对称正定矩阵，可以选择 Cholesky 分解法。但分解方法即使如 CHOLMOD 这类利用了 $S$  的稀疏结构的解法，也很耗时间和空间，对于大型问题而言非常昂贵。

### 2.1 预条件共轭梯度法

- 共轭梯度法（CG）是一种对称正定线性方程组 $Ax=b$ 的迭代解法。
	
	- 只涉及 $Ap$ 的结果，不需要 $A$，可以在不显式在内存中构造矩阵 $A$ 的前提下解算线性方程组。

- 针对方程 $(J^{T}J+\lambda D^{T}D)\delta=-J^{T}f$ 有两种解法
	
	- 针对最小二乘的共轭梯度法（CG）
		已知 $J$ ，则 $H_\lambda p$ 很容易实现
		
		$$ H_{\lambda}p=J^{T}(Jp)+\lambda(D^{T}D)p $$

		使用分块 Jacobi 预条件子加速共轭梯度法的收敛
		
		$$M_\lambda=\begin{bmatrix}U_\lambda&0\\0&V_\lambda\end{bmatrix}$$

		$U_\lambda$ 和  $V_\lambda$ 以及它们的逆都很容易在线性时间内计算。 
		这种使用矩阵方向乘积的预条件共轭梯度法称为隐式 Hessian 算法

	- 隐式 Schur 算法

		在 Schur 补  $S$ 上使用共轭梯度法。$S$  比 $H$ 更良态，因此共轭梯度法会更快地收敛。
		利用 Schur 补计算 $Sp_c$ 

		$$Sp_{c}=U_{\lambda}p_{c}-W(V_{\lambda}^{-1}(W^{T}p_{c}))$$

		使用 $M_{\lambda}=U_{\lambda}$ 作为预条件子。
		仿造前一种方法中不构造 $H_\lambda$ 进行共轭梯度法的思路，可以不构造 $H_\lambda$ 的昂贵的子矩阵 $W$ 和 $W^{T}$

		$$\left.Sp_{c}=J_{c}^{T}\left(J_{c}p_{c}-J_{p}\left(V_{\lambda}^{-1}(J_{p}^{T}\left(J_{c}p_{c}\right)\right)\right)\right)+\lambda D_{c}^{T}D_{c}p_{c}$$

- 这两种方法本质上使用同样多的内存，相同一套 Jacobian 矩阵和向量乘积，以及类似迭代的计算复杂度。不仅不需要显式构造增广 Hessian 矩阵/ Schur 补，而且将复杂的分块稀疏矩阵和向量的乘法分解成了多个更简单，更易并行化的矩阵向量乘法。甚至在计算过程中不用存储矩阵 $J$，而是在运算过程中就计算了它的元素。

## 3 并行实现
- 在 LM 算法中使用线性迭代解法，计算开销的主要来源

	1. 计算重投影误差 $f$
	2. 计算 Jacobian 矩阵 $J=[J_{c},J_{p}]$
	3. 构造预条件矩阵 $M_{\lambda}^{-1}$
	4. 矩阵向量乘法 $Jx=J_{c}x_{c}+J_{p}x_{p}$
	5. 矩阵向量乘法 $J^{T}y=[J_{c}^{T}y,J_{p}^{T}y]$
	6. 矩阵向量乘法 $M_{\lambda}^{-1}v$

- 计算开销评估

	- 计算 $Jx$ 和 $J^{T}y$ 花费时间最多，因为每次 CG 的迭代都需要计算。
	- $J$ 和 $M_{\lambda}^{-1}$ 虽然很昂贵，但每次 LM 迭代只计算一次，所以相对并不重要
	-  $Jx$ 和 $J^{T}y$ 应当是优化的主要对象
	- 需要注意的是，上述函数（将计算 $f$ 或 $Jx$ 等过程称为函数）可以通过将计算任务分为按照摄像机、按照点和按照测量进行线程并行化

		- 计算 $f$、$J$ 和 $Jx$ 包括执行每个测量任务
		- 而 $J^Ty$、$M^{−1}_\lambda$ 和 $M_{\lambda}^{-1}v$ 包括执行每个摄像机和每个点的任务
		
	与构造 $H_\lambda$ 或 $S$ 相比，这些函数可以并行化。
	纯向量的并行化很容易，不作讨论。

### 3.1 相关工作

GPU 并行化的工作已经有了广泛的研究关注。包括一些通用的库。如 nVidia 的稀疏矩阵向量乘法库、各种基于 GPU 的预条件共轭梯度法和预条件技术的工作。但是基于问题的结构来构建系统可以获得更好的性能。

在计算机视觉领域，许多算法已经迁移到 GPU 上，速度显著提高，包括特征检测、特征匹配和立体重建等。【6】中描述了基于多 GPU 的快速 SfM 系统，但是在该系统中，捆绑调整步骤仍然是在单线程 CPU 上执行的。

【11】采用了混合方法，在 GPU 和 CPU 上运行重叠的计算，其中 Hessian 矩阵和 Schur 补是在 GPU 上构建的。但对于大型问题（数千张或更多张图像），尤其是对于具有密集 Schur 补的社区照片集来说，这并不实际。即使有足够的内存，构建 Schur 补充仍然对于大规模问题来说太过昂贵。

### 3.2 并行化准则

- 多核系统上开发高性能系统的问题
	- 处理器速度与从内存获取数据的速度不匹配
	- 多线程的对内存的访问竞争
- 为获得最佳性能的解决思路
	- 最大化处理器占用率
	- 优化内存访问减少竞争

#### 3.2.1 矩阵存储
采用块压缩稀疏行（BCSR）格式存储 $J_C$，$J_p$ 和 $J_c^T$（需要时）。根据 3D 点的 ID 对观测值进行分类，这就意味着 $J_p$ 是分块对角矩阵， $J_p$ 与 $J_p^T$ 有相同的 BCSR 表达

### 3.3 CPU 并行化

多核 CPU 通常可以同时运行数十个线程，并且可以利用多级 cache 访问大量 RAM。

#### 3.3.1 最大化处理器占用率

现代 CPU 上的 SSE 功能通过使用单个指令处理 4 个浮点数或 2 个双精度数来提高处理器吞吐量

- 加速简单的向量操作（如加法、乘法、范数和点积）变得容易
- 将相机参数的数量按 4 对齐

	- 虽然相机模型只有7个参数，但也使用 8-向量存储

- 相机导数的乘法和加法可以通 SSE 指令进行
- 分配对齐数据可利用快速对齐内存加载和存储
#### 3.3.2 优化内存访问模式

- 减少多个线程需要访问相同索引信息时的竞争，

	- 存储必要的索引结构，以使每个线程能够独立运行

		- $J^Ty$ 按相机分线程计算，访问每个相机的所有投影
		- 按测量分线程运行，则需要对同一相机的不同测量进行独占访问

	- 存储数据的混排副本（如果可能的话），使频繁调用的函数具有连续的内存访问模式

		- $J^Ty$ 需要访问由相机拍摄的所有测量的 Jacobian 分块

			- 最好的方式是按照行分块顺序存储额外的 $J^T_c$ 副本，而不是按索引访问 $J_c$

### 3.4 GPU 并行化

GPU 的核心很多，但内存较少。必须更谨慎地处理不同核心间的内存带宽竞争。

#### 3.4.1 最大化处理器占用率

- GPU 在定义上是 SIMD

	- nVidia 的 CUDA 将线程组织成线程块，进一步分为 32 个线程单位，称为 warp，它们本质上是 SIMD。

- 实现 GPU 峰值内存带宽需要进行合并内存访问

	- 将相机参数和点参数按 4 对齐，以使相机参数和点参数的处理与 warp 大小对齐
	- 在无法进行合并内存获取时，采用了使用 GPU 纹理内存作为缓存

- 简单的程序可以在 GPU 上实现更高的占用率

	- 将问题变为每个参数一个线程或每个相机半个 warp（16个线程）等方式。<del>而不是每个相机或每个点创建一个线程</del>
	- 使得同一相机/点的不同线程可以共用快速共享内存上的数据。

		- 使用 8 个线程反转 $8\times8$ 相机对角块矩阵。

#### 3.4.2 优化内存访问模式

- 将所有矩阵 $J^T c$、$J_c$ 和 $J_p$ 以 BCSR 格式存储
- 允许函数 $J_x$、$J^T_y$ 和 $M^{−1} _\lambda$ 连续获取 Jacobian 分块。
- 利用纹理获取和共享内存的块结构
- 计算并显式存储在各个线程之间进行聚合操作所需的索引，避免不同线程块之间的依赖关系
- 减小 GPU 算法的内存使用，使用 $Jx$、$J^Ty$ 和 $M^{−1} _\lambda$ 的无矩阵版本（$J$ 和 $M_\lambda$ 中需要的元素实时计算）

	- 不仅节省空间，也更节省了时间
	- GPU 上的大量线程并行性竞争会存在空闲
	- 最好读取可很好缓存的少量内存，并重新计算结果。
	- 按以下优先级：$J_p > J^T c > J_c$ 指导内存分配

- LM 步骤的计算已经卸载到 GPU 上后，GPU 和 CPU 之间的内存传输会减少
- 对于小型和中型问题，内存争用并不是一个问题
- 预先计算并存储在 GPU 内存中的 $J_p$ 和 $J_c$，计算 $J^T_y$ 更快

## 4 准确度
- 经过一些小心处理，可以使用单精度计算而不会在数值上有明显损失

	- 预处理数据归一化步骤，改善 Jacobian 值的分布

		- 尺度为 $\dfrac{F}{z^2}$ ，$F$ 是相机的焦距，$z$ 是相机中的 3D 点的深度
		- 进行缩放，控制 $F$ 和 $\dfrac{1}{z}$ 的值在有效范围中，使 Jacobian 矩阵更良态
		- 处理步骤 （$C_n$ 为正则化参数，取 0.5）
		- 找到中值 $F_m = \mathrm{median}\{F\}$ ， $z_m = \mathrm{median}\{z\}$ 
		- 把所有的焦距和测量值缩放 $\dfrac{C_n}{F_m}$ 
		- 把所有的平移值和 3D 点缩放 $\dfrac{1}{z_mC_n}$ 

	- 把 Jacobian 的按 $H_0$ 对角元素的平方根缩放（$H_0$ 为初始增广 Hessian）

## 5 实验

### 5.1 结果

- 算法的 GPU 实现速度提高了 10 到 30 倍，多线程 CPU 实现速度提高了约 5 到 10 倍

	- 速度提升在各种问题大小和问题稀疏性上都是一致的
	- 尽管系统设计和优化工作重点在大规模问题上，但系统在小型和中型问题上表现同样出色

- GPU 版本和 CPU 版本之间的速度比在中等大小的问题上最高

	- 似乎随着问题规模的增大，访问 GPU 内存的成本在增加
	- 由于 GPU 上全局纹理缓存的尺寸有限

- 与隐式 hessian 相比，隐式 schur 的 LM 步骤成本更高，但由于其更好的预条件行为，隐式 schur 在每个 LM 迭代中的收敛性更好。
- 即使结果是在高端的 Tesla GPU 上得出的，我们也可以期望在消费级 GPU 上获得类似的性能

## 6 结论

- 提出了在当前可用的 CPU 和 GPU 上运行的捆绑调整问题的多核解决方案。

	- 这些系统可以比现有系统提供 10 倍到 30 倍的速度提升
	- 同时减少了内存使用量。

- 实现方法

	- 通过将 PCG 迭代中使用的矩阵向量乘法小心地重组为易于并行化的操作。
	- 这种重组还为无矩阵实现打开了大门，从而大大降低了内存消耗和执行时间。
	- 我当与适当的规范化相结合时，单精度算术可以提供与基于双精度的求解器可比的数值性能，同时进一步降低内存和时间成本。

- 展望

	- 提出的策略可以应用于计算机视觉和其他领域的其他大规模优化问题。
	- 希望进一步提高我们的单精度求解器的数值稳定性
	- 在 GPU 上尝试双精度算术
	- 将这项工作移植到非 nVidia 平台
	- 未来工作的一个有趣方向是创建一个类似 FFTW【7】或 ATLAS【14】的框架，用于在编译时或运行时自动调整系统的参数。 

## 参考文献

??? info "References"

	【1】 S. Agarwal, N. Snavely, S. Seitz, and R. Szeliski. Bundle adjustment in the large. In ECCV10, pages II: 29–42, 2010. 3057, 3058, 3059, 3062, 3063

	【2】 S. Agarwal, N. Snavely, I. Simon, S. M. Seitz, and R. Szeliski. Building Rome in a day. In ICCV, 2009. 3057, 3064

	【3】 N. Bell and M. Garland. Implementing sparse matrix-vector multiplication on throughput-oriented processors. In SC ’09, pages 1–11, 2009. 3059, 3061

	【4】 M. Byrod and K. Astrom. Conjugate gradient bundle adjustment. In ECCV10, pages II: 114–127, 2010. 3058

	【5】 Y. Chen, T. Davis, W. Hager, and S. Rajamanickam. Algorithm 887: CHOLMOD, Supernodal Sparse Cholesky Factorization and Update/Downdate. TOMS, 35(3), 2008. 3058

	【6】 J. Frahm, P. Fite Georgel, D. Gallup, T. Johnson, R. Raguram, C. Wu, Y. Jen, E. Dunn, B. Clipp, S. Lazebnik, and M. Pollefeys. Building rome on a cloudless day. In ECCV10, pages IV: 368–381, 2010. 3057, 3059

	【7】 M. Frigo and S. G. Johnson. The design and implementation of FFTW3. Proc. of the IEEE, 93(2):216–231, 2005. 3064

	【8】 Y. Jeong, D. Nister, D. Steedly, R. Szeliski, and I. Kweon. Pushing the envelope of modern methods for bundle adjustment. In CVPR10, pages 1474–1481, 2010. 3058

	【9】 R. Li and Y. Saad. GPU-accelerated preconditioned iterative linear solvers,. Technical Report UMSI-2010-xx3, Minnesota Supercomputer Institute, University of Minnesota, 2010. 3059

	【10】 J. Nocedal and S. Wright. Numerical optimization. Springer, 2000. 3058

	【11】 S. C. Shubham Gupta and P.J.Narayanan. Practical time bundle adjustment for 3d reconstruction on gpu. In ECCV Workshop on Computer Vision on GPUs, 2010. 3059

	【12】 L. Trefethen and D. Bau. Numerical linear algebra. SIAM, 1997. 3058

	【13】 B. Triggs, P. McLauchlan, H. R.I, and A. Fitzgibbon. Bundle Adjustment - A modern synthesis. In Vision Algorithms’99, pages 298–372, 1999. 3057

	【14】 R. C. Whaley and A. Petitet. Minimizing development and maintenance costs in supporting persistently optimized BLAS. Software: Practice and Experience, 35(2):101–121, February 2005. 3064
