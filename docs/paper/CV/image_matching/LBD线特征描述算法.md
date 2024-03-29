---
title: LBD 线特征描述算法
date: 2023-03-02 17:59:27
excerpt: LBD 线特征描述算法
tags: 
- CV/特征描述
- CV/线特征
rating: ⭐⭐⭐
---

# LBD 线特征描述算法

!!! note "参考网页"
    [LBD 算法 - Line Band Discriptor 描述符分析\_Xue Feng BUPT 的博客-CSDN 博客](https://blog.csdn.net/chishuideyu/article/details/78132093)

	[OpenCV: Binary descriptors for lines extracted from an image](https://docs.opencv.org/4.0.0-beta/dc/ddd/group__line__descriptor.html)

	[【论文阅读】An efficient and robust line segment matching approach based on LBD descriptor 高斯金字塔在哪篇论文中被提出 林北不要忍了的博客 - CSDN 博客](https://blog.csdn.net/weixin_43849505/article/details/123313314)

!!! info "论文链接"
	原文：Lilian Zhang, Reinhard Koch. [An efficient and robust line segment matching approach based on LBD descriptor and pairwise geometric consistency](https://doi.org/10.1016/j.jvcir.2013.05.006)

## 介绍
### 相关工作
线匹配的分类：

- 单个线段的匹配
	- Mean-Standard deviation Line Descriptor (MSLD)
- 线段组的匹配
- 线段上相应点的匹配
	- coplanar Line Intersection Context Features (LICF)
### 本文方法
- 线特征匹配的困难
	- 端点定位不精确
	- 线段碎片化
	- 缺少强几何约束（如极线约束）
	- 低纹理场景中缺少突出现象
	- 大影像变换时不稳定
- **采用策略**
	- 在不同的尺度空间中提取线段，增强方法的尺度变化稳健性。
		- **EDLine**
	- 使用 LBD 描述线段的局部特征
	- 线段局部特征与线段对之间的几何约束结合，构建相关图（？）
## 线段检测与描述
### 尺度空间线段的检测与描述
- 用尺度缩放因子与高斯模糊建立尺度空间金字塔。
- 对每层使用 **EDLine** 检测出尺度间线段集
- 判断线段方向（让最多的像素梯度从左到右的方向）
- 重新组织线段，把尺度空间中对应的线段归为一个 LineVec 中

### 线段支持区域的带宽描述
- 选择线段方向 $d_L$ 和与线段垂直的方向 $d_{\perp}$ （从线段方向顺时针得到），以及线段中点，建立局部坐标系。
- 受 SIFT 启发，引入两个高斯函数
	- 一个全局加权 $f_g$，对每一行采用不同的 $\sigma$ 值
		- **减少远离线段的像素的梯度的影响，减小对线段垂直方向上微小变化的敏感度。**
	- 一个局部加权 $f_l$，对某带，以及相邻带进行加权
		- **减少边界效应，避免描述子在带与带之间产生突变**

### 线段带宽描述子的构建
- 对于线段支持域的某带，其带描述子由该带及相邻带的各行计算得到（边缘两带不存在超出部分的相邻带），带描述子拼接后作为线段的描述子。
- 带描述子的计算方法，是对该带（以及相邻带）每行分别统计梯度特征（在 $d_L$ 和 $d_{\perp}$ 上对正负分别加权累加），得到带描述矩阵。
- 对带描述矩阵计算均值标准差向量（$M_j$ 和 $S_j$ ，每个向量都有 4 个分量）。因此 LBD 最终有 $8m$ 个分量 （$m$ 为 LBD 的带数）
- 均值与标准差的强度不同，所以需要分开进行标准化
- 为了减少非线性照明的变化，LBD 的每一维都被限制在一个阈值内
- 最后，重新标准化

## 使用谱方法（？）进行图像匹配
### 生成候选匹配线对
- 参考影像与查询影像之间的 LineVecs 检测，如果它们不能通过一元几何属性与局部特征相似性的测试的话，那么是不匹配的（？）
	- 一元几何属性这里指 LineVecs 的方向（如果计算不出旋转角，则只进行下一测试）
		- 比较两幅影像的 LineVecs 方向直方图分布情况，选择出一个旋转角。为保证准确性，还要保持一个角度上的线的长度相似
	- 局部特征相似性
		- 用线特征描述子的距离表示，最小距离大于阈值则不考虑这一线对的匹配
### 构建相关图
- $\kappa\times \kappa$ 的邻接矩阵记录两组匹配线对之间相关度的评分，是非负实对称矩阵（涉及四个 LineVec，两组匹配）
- 评分通过线对几何属性与特征相似度计算
	- 线对几何属性是由线对尺度空间中描述子距离最短的一对线对衡量的
	- 在原影像中定位线对的端点后，计算交比、投影角与相关角
	- 评分计算通过比较两组线对的交比、投影角与相关角差异得到
### 生成最终匹配结果
构建了邻接矩阵 $A$ ，匹配问题转化为找到一个 $x^*$ 最大化二次型 $x^\mathrm{T}Ax$ 的问题，$x$ 为选择的匹配线对。由于一个 LineVec 可能参与了多组匹配线对（一对多情况），所以不能简单地取 $x$ 为全 1 的向量。

在放宽了整数限制后，我们采用谱方法，利用主特征向量计算 $x$ 。

计算步骤如下：

1. 从两张图中用 **EDLine** 筛选出线的向量
2. 利用**方向直方图**计算全局旋转角
3. 计算两幅图 LineVecs 的 **LBD** 描述子
4. 根据两步筛选得到候选线对 $\mathcal{CM}$
5. 计算任意两组匹配线对相似度并组合出邻接矩阵 $A$
6. 通过 ARPACK 计算 $A$ 特征向量
7. 初始化一个集合 $\mathcal{LM}$，为 $\emptyset$，以存储正确匹配的线对
8. 根据主特征向量，找出其中的最大值，如果这个最大值的 $x^*$ 对应位不为 0，就将对应位置的候选线对加入 $\mathcal{LM}$，并在候选线对中删除这一对；若最大值为 0，结束计算返回 $\mathcal{LM}$
9. 剩下的候选线对与选择过的线对存在矛盾时（一个线段描述子只能作为一个匹配线对，不能一对多），将其从 $\mathcal{CM}$ 中剔除，并令 $x^*$ 相应位为 0
10. $\mathcal{CM}$  为空时，结束计算返回 $\mathcal{LM}$；否则，回到第 8 步