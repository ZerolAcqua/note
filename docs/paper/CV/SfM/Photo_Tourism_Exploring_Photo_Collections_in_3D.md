---
title: "Photo Tourism: Exploring Photo Collections in 3D"
date: 2023-08-27 16:55:47
---

# Photo Tourism: Exploring Photo Collections in 3D

!!! info "论文链接"
	原文：Noah Snavely, Steven M. Seitz, Richard Szeliski. [Photo Tourism: Exploring Photo Collections in 3D](http://phototour.cs.washington.edu/Photo_Tourism.pdf)

!!! warning "注意"
    由 claude 和 chatgpt 翻译整理



## 摘要

本文提出了一种新颖的端到端系统，可以获取无序图片集合，对其进行注册，并以全新的 3D 界面向用户呈现这些图片。我们的系统包含一个基于图像的建模前端，它可以自动计算每个照片的视点以及场景的稀疏 3D 模型和图像与模型之间的对应关系。我们的照片探索器使用基于图像的渲染技术在照片之间平滑过渡的同时，还实现了完整的 3D 导航和场景与世界几何的探索，以及辅助信息比如俯视图。我们的系统也使浏览特定场景或历史位置的照片旅游变得非常容易，并可以在相关图像上自动传输图像细节注释。我们在几个大规模个人照片收藏以及从互联网照片共享网站收集的图像上演示了我们的系统。

## 1 介绍

基于图像的渲染的一个核心目标是基于一组照片激发强烈的现场感。过去几年中，通过研究界的视图合成方法以及商业产品比如全景工具，在实现这个目标方面取得了显著进展。梦想之一是有朝一日这些方法将允许人们虚拟游览世界上有趣和重要的场所。

与此同时，数字摄影结合互联网，使得照片得以实现前所未有的大规模共享。例如，在 Google 图片搜索中使用“Notre Dame AND paris”可以返回超过 15,000 张照片，从无数的视点、细节层次、光照条件、季节、十年等方面捕捉了这个场景。不幸的是，共享照片的激增速度已经超过了浏览这类收藏的技术，因为[谷歌](https://www.google.com)和 [Flickr](https://www.flickr.com) 等工具只是返回了成页的缩略图，用户必须逐一查看。

本文提出了一种浏览和组织热门场所大量照片收藏的系统，该系统利用共享的基础 3D 几何。我们的方法基于从图像本身计算出摄影师的位置和方向，以及场景的稀疏 3D 几何表示，使用当前最先进的基于图像的建模系统。我们的系统可以处理不同相机、变焦级别、分辨率、不同时间、光照、天气和遮挡程度的大量无组织照片。我们展示了相机姿态和场景信息如何实现以下功能:

- 场景可视化。在 3D 空间中通过在照片之间形态学平滑过渡来飞越热门全球场所。

- 基于对象的照片浏览。显示更多包含这个对象或场景部分的图像。 

- 我当时在哪里? 告诉我当我拍下这张照片时我在哪里。

- 我看到了什么? 通过从相似图像传输注释，告诉我此图像中可见的对象。

我们的论文提出了实现这些目标的新颖的基于图像的建模、基于图像的渲染和用户界面的技术，以及将其组合成一个端到端的 3D 照片浏览系统。得到的系统在实践中非常强健；我们包括了从圣母院（图 1）到长城和优胜美地国家公园等众多场所的结果，证明了其广泛的适用性。


## 3 相关工作

与我们的工作相关的主要有三类：基于图像的建模；基于图像的渲染；以及图像浏览，检索和注释。

### 3.1 基于图像的建模

基于图像的建模(IBM)是从一组输入图像创建三维模型的过程。我们的基于图像的建模系统基于最近的结构从运动(SfM)工作，其目的是从图像序列中恢复相机参数、姿态估计和稀疏 3D 场景几何。具体而言，我们的 SfM 方法与 Brown 和 Lowe 的方法类似，但对提高各种数据集的稳健性做出了几个修改。这些修改包括使用姿态估计初始化新相机以帮助避免局部极小；用于选择 SfM 的初始两张图像的不同启发式方法；在将点添加到场景之前检查重建点是否调节良好；以及使用图像 EXIF 标签中的焦距信息。Schaffalitzky 和 Zisserman 提出了另一种相关技术，用于校准无序图像集，着重于有效地在图像之间匹配兴趣点。虽然这两种方法都解决了我们所做的相同的 SfM 问题，但它们是在更简单的数据集上进行测试的，图像变化条件更为有限。我们的论文标志着 SfM 技术首次成功应用于在谷歌和 Flickr 上可以找到的那种真实世界图像集。例如，我们典型的图像集包含数百台不同相机拍摄的照片，有不同的变焦级别、分辨率、不同拍摄时间、光照条件、天气状况，以及不同程度的遮挡。

一个 IBM 的特定应用是创建大规模建筑模型。典型例子包括半自动 Facade 系统，它被用来重建非常令人信服的 UC Berkeley 校园飞越效果；Dick 等人的自动建筑重建系统；以及 MIT 城市扫描项目，它使用一个带有仪器的移动装置拍摄了成千上万的标定图像来计算 MIT 校园的 3D 模型。还有几个正在进行的学术和商业项目专注于大规模城市场景重建。这些工作包括 4D 城市项目(www.cc.gatech.edu/4d-cities),其目标是从历史照片中创建亚特兰大的时空模型；斯坦福 CityBlock 项目，使用城市街区的视频来创建多视角条形图像；以及 Pollefeys 和 Nistér 的 UrbanScape 项目(www.cs.unc.edu/Research/urbanscape/)。

### 3.2 基于图像的渲染

基于图像的渲染(IBR)领域致力于从一组输入照片合成新视点的问题。这一领域的先驱是具有开创性的阿斯彭电影地图项目，其中阿斯彭的成千上万张图像从移动汽车中捕捉，并与该城市的街道地图相关联，存储在激光碟上。一个用户界面使得用户可以交互式地在图像之间移动，作为用户所需路径的函数。额外的功能包括覆盖在图像显示上的城市导航地图，以及点击当前视野中任何建筑的能力并跳转到该建筑的正面。该系统还允许将元数据（如餐馆菜单和历史图像）与单个建筑相关联。我们的工作可以看作是自动从无组织图像集合中创建电影地图的一种方式。(相比之下，阿斯彭电影地图涉及十几个人团队十几年的工作。)我们的许多可视化、导航和注释功能与原始电影地图中的功能相似，但以改进和概括的形式出现。

更近期的 IBR 工作侧重于新视图合成的技术，例如。我们的工作在应用方面，Aliaga 等人的“图像之海”工作可能与我们的最接近，因为它使用了整个建筑空间拍摄的大量图像；同一作者还解决了跨多个图像进行一致特征匹配的问题，以实现 IBR 的目的。然而，我们的图像是不同摄影师随意获取的，而不是由引导机器人在固定网格上拍摄的。

与大多数先前的 IBR 工作相反，我们的目标不是准确合成场景中的所有视点，而是在给出场景几何感的背景下以 3D 空间下浏览特定的照片集合。因此，我们使用了一种近似的基于平面视图插值方法和对背景场景结构的非照片逼真渲染。通过这种方法，我们规避了更具挑战性的问题，比如重建完整的表面模型、光场或像素精确的视图插值。这样做的好处是我们能够稳健地处理之前的 IBM 和 IBR 技术无法处理的输入图像。

### 3.3 图像浏览检索和注释

存在许多技术和商业产品用于浏览照片集合，以及大量关于人们倾向于如何组织照片的研究，例如。许多这些技术使用元数据，比如关键词、摄影师或时间，作为照片组织的基础。

使用地理位置信息帮助照片浏览的兴趣日益增加。特别是，世界范围内的媒体交换在交互式 2D 地图上排列图像。PhotoCompas 根据时间和位置对图像进行聚类。Realityflythrough 使用与我们的资源类似的界面理念来探索配备 GPS 和倾斜传感器的摄像机记录的视频，而 Kadobayashi 和 Tanaka 提出了一种使用接近虚拟相机的接口检索图像。在这些系统中，位置是从 GPS 获得的，或是手动指定的。因为我们的方法不需要 GPS 或其他仪器，所以它可以应用于现有的图像数据库和互联网上的照片。此外，我们方法的许多导航特性利用了图像特征对应和稀疏 3D 几何的计算，因此超越了这些先前的基于位置的系统所能实现的功能。

许多技术也存在于相关的从图像数据库中检索图像的任务。一个与我们工作相关的特定系统是视频谷歌，它允许用户在视频的一帧中选择查询对象，并有效地在其他帧中找到该对象。我们的基于对象导航模式使用了类似的想法，但将其扩展到了 3D 领域。

许多研究人员已经研究了图像注释的自动化和半自动化技术，尤其是注释传输。LOCALE 系统使用接近度在带有地理参考的照片之间传递标签。我们系统的注释功能的一个优点是我们的特征对应使得可以在更细粒度级别上传输注释；我们可以在图像之间传输特定对象和区域的注释，同时考虑这些对象在视点变化下的遮挡和运动。这个目标与增强现实(AR)方法类似，后者也寻求对图像进行注释。而大多数 AR 方法是将一个 3D 计算机生成的模型注册到一个图像中，我们是将 2D 图像注释传输到其他图像。因此，生成注释内容要容易得多。(我们实际上可以导入 Flickr 等流行服务中已经存在的注释。)注释传输在视频序列中也已被探索。最后，Johansson 和 Cipolla 开发了一个系统，用户可以拍一张照片，上传到服务器进行与图像数据库的比较，并接收位置信息。我们的系统还支持这一应用，以及许多其他功能（可视化、导航、注释等）。

## 4 重建相机和稀疏几何

我们的系统需要每个收藏中照片的相对位置、方向和内在参数如焦距等的准确信息，以及场景的稀疏 3D 几何。我们的系统的一些功能需要相机的绝对位置，在一个地理参考坐标框架中。一些信息可以通过 GPS 设备和电子罗盘提供，但大多数现有照片都缺乏此类信息。许多数字相机会在图像文件的 EXIF 标签中嵌入焦距和其他信息。这些值对于初始化很有用，但有时不够准确。

在我们的系统中，我们不依赖于相机或任何其他设备来提供位置、方向或几何信息。相反，我们使用计算机视觉技术从图像本身计算这些信息。我们首先在每张图像中检测特征点，然后在图像对之间匹配特征点，最后运行一个迭代的稳健 SfM 过程来恢复相机参数和这些特征的 3D 位置。由于 SfM 仅估计每个相机的相对位置，而我们也对绝对坐标（如纬度和经度）感兴趣，所以我们使用一种交互技术将恢复的相机注册到俯视地图上。下面详细描述每个步骤。

### 4.1 关键点检测和匹配

第一步是在每张图像中找到特征点。我们使用 SIFT 关键点检测器，因为它对图像变换具有不变性。一张典型图像包含几千个 SIFT 关键点。也可以潜在地使用其他特征检测器; Mikolajczyk 等对几个检测器进行了比较。除了关键点位置本身之外，SIFT 还为每个关键点提供一个局部描述符。接下来，对于每对图像，我们使用 Arya 等的近似最近邻包在图像对之间匹配关键点描述符，然后使用 RANSAC 稳健地估计图像对的基础矩阵。在每个 RANSAC 迭代中，我们使用八点法计算一个候选基础矩阵，然后进行非线性优化。最后，我们删除基础矩阵的离群值匹配。如果剩余匹配数量少于 20，我们将所有匹配从考虑中删除。

在每对图像之间找到一组几何一致的匹配后，我们将匹配组织成轨迹，其中轨迹是跨多个图像的匹配关键点的连接集。如果一个轨迹在同一图像中包含多个关键点，则认为它不一致。我们保留包含至少两个关键点的一致轨迹供重建程序的下一阶段使用。

### 4.2 SfM

接下来，我们恢复一组相机参数和每个轨迹的 3D 位置。恢复的参数应该是一致的，即所有轨迹的再投影误差之和最小化。将这个最小化问题形成一个非线性最小二乘问题（详见附录 A）,并通过 Levenberg-Marquardt 等算法求解。这样的算法只能找到局部极小值，大尺度的 SfM 问题特别容易陷入局部极小值，所以提供良好的参数初始估计非常重要。我们没有一次估计所有相机和轨迹的所有参数，而是增量地添加相机。

我们首先估计一对相机的参数。这对初始相机应该有大量匹配，但基线也应该足够大，以使观测点的 3D 位置估计良好。因此，我们选择匹配数量最多的图像对，同时它们的匹配不能很好地由单应变换建模（以避免退化情况）。

接下来，我们将另一个相机添加到优化中。我们选择观测到最多已估计 3D 位置点的相机，并使用直接线性变换(DLT)技术在 RANSAC 过程中初始化新相机的外参。 DLT 还给出内参矩阵 K 的一个通用上三角矩阵估计。我们使用 K 和图像 EXIF 标签估计的焦距来初始化新相机的焦距（详见附录 A）。

最后，我们将新相机观测到的轨迹添加到优化中。如果一个轨迹被至少一个已恢复的相机观测，并且三角化该轨迹给出良好调节的位置估计，则添加该轨迹。这个过程对每一个相机逐一重复，直到没有剩余的相机观测任何已重建的 3D 点为止。为了在每次迭代中最小化目标函数，我们使用 Lourakis 和 Argyros 的稀疏捆绑调整库。在重建一个场景之后，我们可以可选地运行后处理步骤，使用 Schmid 和 Zisserman 的线段重建技术检测场景中的 3D 线段。

为了增加稳健性和速度，我们对上述基本过程做了一些修改。首先，每次运行优化后，我们检测包含至少一个高再投影误差的关键点的异常轨迹，并将这些轨迹从优化中移除。然后我们反复运行优化并拒绝每次运行后的异常值，直到不再检测到异常值为止。其次，我们不仅一次添加一个相机，而是添加多个相机。我们首先找到与现有 3D 点匹配最多的相机 M，然后添加任何与现有 3D 点具有至少 0.75M 匹配的相机。

图 2 和 3 显示了使用这种方法重建的相机（以视锥体渲染）和几个著名地点的 3D 特征点。

### 4.3 地理注册

SfM 过程估计的是相对相机位置。位置估计过程的最后一步是将模型与地理参考图像或地图（如卫星图像、楼层平面图或数字高程模型）对齐，以确定每台相机的绝对球心坐标。为了实现一些我们的资源管理器功能，这一步是不必要的，但对其他功能（如显示俯视图）是必需的。

理论上，估计的相机位置与绝对位置之间的关系是一个相似变换（全局平移、旋转和统一尺度）。为了确定正确的变换，用户交互式旋转、平移和缩放模型，直到它与提供的图像或地图一致。为了帮助用户，我们使用 Szeliski 的方法估计“向上”或重力向量。然后使用正交投影从场景上方指向下方渲染 3D 点、线段和相机位置，以与对齐图像叠加。如果向上向量估计正确，用户只需要在 2D 而不是 3D 中旋转模型。我们的经验是，在城市场景中，通过匹配恢复的点与图像中可见的特征（如建筑外观来执行此对齐相当容易。图 4 显示了这样的对齐的屏幕截图。

在某些情况下，恢复的场景无法使用相似变换与地理参考坐标系统对齐。这可能是因为 SfM 过程未能获得场景的完全度量重建，或者因为恢复的点和相机位置中的低频漂移。这些误差源在我们的资源管理器界面中用于的许多导航控制中通常不太明显，但在需要精确模型时会造成问题。

“拉直”恢复场景的一种方法是固定一小部分地面控制点或相机到已知的 3D 位置（例如，从附加到少数图像的 GPS 标签获取）通过向 SfM 优化添加约束条件。或者，用户可以像 Robertson 和 Cipolla 的工作那样手动指定点或相机与图像或地图中的位置之间的对应关系。

#### 4.3.1 对准数字高程模型

对于景观和其他大范围场景，我们可以利用数字高程模型（DEM）,例如在谷歌地球（earth.google.com）中使用的，以及通过美国地质调查局（www.usgs.com）可以获得覆盖美国大部分区域的 DEM。为了将点云重建与 DEM 对齐，我们手动指定点云与 DEM 之间的几个对应关系，并估计一个 3D 相似变换以确定初始对齐。然后我们重新运行 SfM 优化，与指定的 DEM 点配适作为额外的目标项。未来，随着越来越多的基于地面图像数据的出现（例如通过类似 WWMX 的系统），这一手动步骤将不再必要。

### 4.4 场景表示

我们在照片探索工具中使用的重建场景模型表示如下:

- 一组点 $P = \{p_1, p_2,\cdots, p_n\}$。每个点由一个 3D 位置和在观测到该点的一个图像位置获得的颜色组成。

- 一组相机 $C = \{C_1, C_2, \cdots, C_k\}$。每个相机 $C_j$ 包括图像 $I_j$ ，旋转矩阵 $R_j$ ，平移 $t_j$，和焦距 $f_j$。

- $Points$ 映射，在相机和它们观测到的点之间。也就是说，$Points(C)$ 是包含 $C$ 观测到的点的 $P$ 的子集。 

- 一组 3D 线段 $L = \{l_1, l_2, \cdots, l_m\}$ 和 $Lines$ 映射，在相机和它们观测到的线段集之间。

## 5 照片探索器渲染

一旦一组照片被注册，用户就可以使用我们的照片探索器界面浏览照片。这个接口的两个重要方面是我们如何渲染探索器显示（本节描述）和导航控制（第 6 节描述）。

### 5.1 用户界面布局

图 5 显示了我们的照片探索界面主窗口的屏幕截图。这个窗口的组成部分是占满窗口的主视图，以及三个叠加窗格：左侧的信息和搜索窗格，底部的缩略图窗格，和右上角的地图窗格。

主视图显示从用户控制的虚拟相机看到的世界。这个视图的目的不是显示场景的照片逼真视图，而是以空间环境显示照片并给出真实场景几何的感觉。

当用户访问一张照片时，信息窗格就会出现。这个窗格显示有关该照片的信息，包括其名称、摄影师姓名以及拍摄的日期和时间。此外，该窗格还包含用于搜索与当前照片具有某些几何关系的其他照片的控件，如第 6.2 节所述。

缩略图窗格显示搜索操作的结果，以缩略图片带形式显示。当用户访问一个相机 $Ccurr$ 并将鼠标悬停在一个缩略图上时，对应的图像 $I_j$ 会在主视图中的一个平面上进行投影，以显示该图像的内容以及它在空间中的位置（我们为每个相机对 $C_j$, $C_k$ 预计算投影平面 $CommonPlane(C_j, C_k)$ ,通过稳健拟合一个平面到 $Points(C_j)$ 和 $Points(C_k)$。缩略图面板还有用于按日期和时间对当前缩略图进行排序以及将它们作为幻灯片查看的控件。

最后，地图窗格显示俯视图的场景，跟踪用户的位置和朝向。

### 5.2 场景渲染

主视图从当前视点渲染场景。相机以视锥体渲染。如果用户正在访问一个相机，则使用不透明的全分辨率版本的照片纹理映射该相机视锥体的背面，以便用户可以详细查看它。其他相机视锥体的背面要么使用低分辨率半透明的照片缩略图进行纹理映射（如果视锥体接近用户当前位置）,要么呈现白色半透明。

恢复的点和线段用于描绘场景本身。点以它们获得的颜色绘制，线段为黑色。用户可以控制是否绘制点和线段、点大小和线条粗细。

我们还提供了一种非照片逼真渲染模式，它提供更有吸引力的可视化。这种模式使用淡出的着色给出场景外观和几何的印象，但抽象程度足以宽容对详细几何的缺乏。对于每个相机 $C_j$,我们首先使用 RANSAC 稳健拟合一个平面到 $Points(C_j)$。如果平面内点的数量至少是 $Points(C_j)$ 的 20%,我们然后拟合一个稳健的边界框 $Rectangle(C_j)$ 到平面中的内点。为了渲染场景，我们将每个图像 $I_j$ 的模糊半透明版本投影到 $Rectangle(C_j)$ 上并使用 $\alpha$ 混合将结果组合起来。在用稀疏点表示的场景的部分，我们切换回点和线段渲染。使用投影图像叠加线段的非照片渲染示例如图 5 所示。

### 5.3 照片之间的过渡

当用户在资源管理器中在照片之间移动时，我们使用的在图像之间过渡的方法是用户界面的一个重要元素。大多数现有的照片浏览工具从一张照片切换到下一张，有时通过渐变平滑过渡。在我们的情况下，我们推断出的有关照片的几何信息使我们能够使用相机运动和视图插值来使过渡更具视觉吸引力，并强调照片之间的空间关系。

#### 5.3.1 相机运动

当虚拟相机从一张照片移动到另一张时，我们在初始和最终相机位置之间线性插值相机位置，在表示初始和最终方向的单位四元数之间插值相机方向。虚拟相机的视场也进行插值，以便当相机到达目标时，目标图像将尽可能填充屏幕。相机路径的定时是非均匀的，进入和退出过渡缓和。我们在开始运动前淡出所有其他相机视锥，以避免它们快速穿过视野时造成的闪烁。

如果相机移动是由于对象选择导致的，过渡略有不同。在相机开始移动之前，它朝向选择点的平均值。当它移动时，相机保持指向该平均值，以使所选对象在视图中保持固定。这有助于避免过渡期间所选对象出现大的分散运动。最终方向和焦距计算使得所选对象居中并填满屏幕。

#### 5.3.2 视图插值

在相机过渡期间，我们还显示过渡图像。我们实验了两种简单的在起始和目标照片之间进行形态学的技术：三角网格和平面 impostors。

**三角网格形态学**为了在两个相机 $C_j$ 和 $C_k$ 之间创建三角形形态，我们首先为图像 $I_j$ 计算 Points(C_j)在 $I_j$ 中的投影的 2D Delaunay 三角剖分。Lines(C_j)在 $I_j$ 中的投影被强制作为三角剖分上的边界约束。所得到的 Delaunay 三角剖分可能不覆盖整个图像，所以我们在图像上叠加一个网格，并将未包含在原始三角剖分中的每个网格点添加进来。每个添加的网格点与通常在 $C_j$ 和 $C_k$ 见到的点组成的近似平面上的一个 3D 点相关联。三角剖分的连接性然后被用来从 $Points(C_j)$ 和 $Lines(C_j)$ 的端点创建一个 3D 网格。我们通过将 $I_j$ 投影到每个三角形来纹理映射网格。我们为 $C_k$ 以相同方式计算一个网格并进行纹理映射。

为了渲染介于的视图，我们从新视点渲染每个网格，并按照介于相机与两个端点的距离的比例对两个渲染图像进行融合。虽然这种技术没有使用完全准确的几何，但是网格通常足以给出场景的 3D 几何感。然而，缺失的几何和异常点有时会造成分散注意的伪像，如随附视频所示。

**平面形态学**为了使用平面替身在相机 $C_j$ 和 $C_k$ 之间创建形态，我们简单地将两个图像 $I_j$ 和 $I_k$ 投影到  $CommonPlane(C_j, C_k)$ 上并在相机从 $C_j$ 移动到 $C_k$ 时在投影图像之间交叉渐变。产生的介于图像不如三角形态那样忠实于底层几何，倾向于只稳定场景中的主导平面，但结果伪像通常更能接受，也许是因为我们已经习惯了从不同角度看平面时引起的变形。由于这种方法的稳健性，我们更倾向于将其而不是三角化用作默认过渡。两种技术的形态示例如随附视频所示。

在从图像 $C_j$ 到 $C_k$ 的过渡中，当视图插值不使用的一些特殊情况。首先，如果相机没有观察到公共点，我们目前没有基础进行图像插值。相反，我们淡出开始图像，像往常一样移动相机到目的地，然后淡入目的地图像。其次，如果 $CommonPlane(C_j, C_k)$ 的法向量近乎垂直于 $C_j$ 和 $C_k$ 的平均观测方向，那么投影的图像在形态期间会产生明显的变形。在这种情况下，我们改为使用法向量为观测方向平均值的通过两视图共同点平均值的平面。最后，如果 的消失线在图像 $I_j$ 或 $I_k$ 中可见，那么不可能将整个 $I_j$ 或 $I_k$ 投影到该平面上。在这种情况下，我们尽可能多地将 $I_j$ 和 $I_k$ 投影到该平面上，并将其余部分投影到无穷远平面上。

## 6 照片探索器导航

我们的图像探索工具支持通过场景导航和找到有趣照片的几种模式。这些模式包括自由飞行导航、找到相关视图、基于对象的导航和查看幻灯片。

### 6.1 自由飞行导航

自由飞行导航控件包括许多游戏和 3D 浏览器中常见的标准 3D 运动控件。用户可以向前、向后、左、右、上、下移动虚拟相机，并可以控制平移、倾斜和变焦。这使得用户可以自由地在场景中移动，并提供了一种简单的方法来找到有趣的视点和附近的照片。

在任何时候，用户都可以在主视图中点击一个视锥，虚拟相机将平滑移动直到与所选相机重合。虚拟相机会平移和变焦，以便所选图像尽可能多地填充主视图。

### 6.2 在相关视图间移动

当访问一张照片 $Ccurr$ 时，用户可以获得世界在单个观点和时间点的快照。用户可以平移和缩放来探索照片，但也可能想看到单张图片中捕捉的场景之外的方面；例如，他/她可能想知道视野之外的内容，照片中的对象左边的内容，或者在不同时间段场景的外观。

为了更容易找到这些相关视图，我们为用户提供了一组“几何”浏览工具。这些工具的图标出现在访问照片时出现的信息窗格的两行中。这些工具可以找到与当前视图在空间上具有某些关系的部分场景的照片。实现这些搜索工具的机制是将当前相机观察到的点 $Points(Ccurr)$ 投影到其他照片中（或反过来），并根据投影点运动选择视图。例如，要回答“显示我这张照片左边的内容”的查询，我们会搜索 $Points(Ccurr)$ 看起来向右移动的照片。

对于几何搜索，新图像从当前相机的邻居中选择。我们定义 $C$ 的邻居为与 $C$ 至少共享一个点的相机集合，即 $Neighbors(C) = {C_j | Points(C_j) \cap Points(C) \ne  \varnothing}$。

第一行中的工具用于在不同尺度下找到相关图像。这些工具通过估计不同视图中一组点的可见性和“大小”来选择图像。因为我们没有场景几何的完整知识，为了检查一个点在一个相机中的可见性，我们简单地检查该点是否投影在相机的视场内。为了估计一组点 $P$ 在图像 $C$ 中的明显大小，我们将 $P$ 中的点投影到 $C$ 中，计算投影在图像内的点的轴对齐边界框，并计算边界框面积与图像面积之比（以像素为单位）。我们将此数量称为 $Size(P, C)$。

这个集合中有三个工具:

- 找到当前照片的细节，或更高分辨率的特写

- 找到类似的照片

- 找到俯视图，或显示更多周围环境的照片。

如果当前照片是 $Ccurr$,这些工具通过计算 $Size(Points(C_j), Ccurr)$ 或相反地 $Size(Points(Ccurr), C_j)$ 来搜索适当的相邻照片 $C_j$。我们认为照片 $C_j$:

- 是 $Ccurr$ 的一个细节，如果 $Size(Points(C_j), Ccurr) < 0.75$ 且 $Ccurr$ 中可见的大多数点在 $C_j$ 中也可见

- 与 $Ccurr$ 类似，如果 $0.75 < Size(Points(Ccurr), C_j)/Size(Points(Ccurr), Ccurr) < 1.3$ 且 $Ccurr$ 和 $C_j$ 的视角方向之间的角度小于一个阈值

- 是 $Ccurr$ 的一个俯视图，如果 $Ccurr$ 是 $C_j$ 的一个细节

这些搜索的结果在缩略图窗格中显示（对于细节和俯视图，以增加的表观大小排序）。这些工具对于以更多细节查看场景、比较同一对象在不同时间、季节和年份下的类似视图以及“向外看”以看到更多场景非常有用。

第二行中的工具为用户提供了简单的逐步向左或向右的方式，即以特定方向看到更多场景。为每个相机，我们预先计算一个“左”和“右”图像，并将它们显示为缩略图。为了找到相机 $C_j$ 的左图像和右图像，我们计算 $Points(C_j)$ 从图像 $I_j$ 到每个相邻图像 $I_k$ 的投影的平均 2D 运动 $m_{jk}$。如果 $m_{jk}$ 与所需方向的角度很小（且两图像中 Points(C_j)的表观大小相似）,则 $C_k$ 是一个候选的左图像或右图像。在所有候选中，我们选择运动幅度 $||m_{jk}||$ 最接近 $I_j$ 宽度的 10% 的图像 $I_k$ 。

与左右图像一起，我们还提供了一个“向后步进”工具，这是一个找到上面描述过程选择的第一个俯视图的快捷方式。

### 6.3 基于对象的导航

我们的系统还支持“显示我这对象的照片”的查询，其中问题中的对象可以直接在一张照片或点云中选择。与关键词搜索相比，这种搜索在浏览场景时是非常有用的补充—当用户发现一个有趣的对象时，直接选择是一种直观的方法来找到该对象的更好照片。

在我们的照片探索系统中，用户可以通过在当前照片或点云中拖动一个 2D 框来选择对象。框内的所有点都被认为是选中状态。我们的系统然后通过对数据库中的每个图像进行评分来搜索所选点集的“最佳”图片，评分基于图像对选区的表示优劣。得分最高的照片被选为代表视图，虚拟相机移动到该图像。分数高于阈值的剩余图像显示在缩略图面板中，按降序排列。一个交互示例如图 6 所示。 

任何评价图像对选择的“好坏”的函数都可以用作评分函数。我们选择了基于三个准则的函数

- 选点可见

- 从一个好的角度看对象

- 对象出现在足够的细节中。对于每个图像 $I_j$ ,我们将分数计算为三个术语 $E_{visible}$， $E_{angle}$ 和 $E_{detail}$ 的加权和。这些术语的计算详见附录 C。

一个点选择有时可能包含用户并不打算选择的点。特别是，它可能包括碰巧投影在选择框内的被遮挡点。因为完整的场景几何是未知的，很难测试可见性。为了避免因过大的选择导致的问题，我们首先修剪点集以删除可能被遮挡的点。特别是，如果选择是在访问图像 $I_j$ 时在图像边界内完成的（并包含在其中）,我们使用从该视点已知可见的点($Points(C_j)$)来细化选择。我们计算所选点中也在 $Points(C_j)$ 中的点的 3×3 协方差矩阵，并删除所有马氏距离大于 1.2 的所选点。如果选择是直接在投影的点云上完成的（即不在图像上）,我们则计算所选全部点的加权均值和协方差矩阵。每个点的权重为从虚拟相机中心到该点的距离的倒数。

### 6.4 创建稳定的幻灯片

每当缩略图窗格中包含多个图像时，其内容可以通过按“播放”按钮以幻灯片形式查看。默认情况下，虚拟相机将在空间中从一个相机移动到下一个，在每个图像处停留几秒钟然后继续下一个。用户也可以“锁定”相机，将其固定在当前位置、方向和视场。当缩略图窗格中的图像几乎都从相同的位置拍摄时，这种模式可以稳定图像，从而更容易比较一个图像到下一个。当研究场景外观随时间、季节、年份、天气模式等的变化时，这种模式非常有用。稳定幻灯片的示例在随附视频中示出。

除了将搜索结果作为幻灯片查看之外，用户还可以加载保存的图像序列。这个功能可以用于交互式查看其他用户创作的旅游路径。

## 7 增强场景

我们的系统允许用户通过几种方式向场景添加内容。首先，用户可以在初始照片集注册之后，在运行时注册自己的照片。其次，用户可以对图像区域进行注释，这些注释可以传播到其他图像。

### 7.1 注册新照片

可以按如下方式即时注册新照片：首先，用户切换到俯视地图填充视图的模式，打开一组图像，它们在缩略图面板中显示，并将每个图像拖放到地图上的近似位置。在每个图像放置后，系统将估计每个新照片的位置、方向和焦距，通过在局部运行第 4 节描述的 SfM 流水线的缩减版本。首先，从最接近初始位置的 20 台相机提取并匹配 SIFT 关键点；对每个其他相机进行修剪，只保留具有几何一致性的匹配；识别匹配对应的现有 3D 点；最后，使用这些匹配来优化新相机的姿态。在一组照片被拖放到地图上后，我们的测试机器上（配备 3.40GHz Intel Pentium 4 的 PC)优化每个新相机的参数通常需要大约 10 秒钟。

### 7.2 对象注释

其他照片组织工具也支持注释，但我们系统的独特功能是注释可以从一幅图像自动传输到所有包含相同场景区域的其他图像。

用户可以选择图像的一个区域并输入文本注释。然后将注释存储下来，与所选点一起显示为围绕所选点的半透明框。一旦添加了注释，对象就可以链接到其他信息源，如网站、导游书以及视频和音频剪辑。

创建注释时，它会自动传输到所有其他相关照片。为了确定注释是否适合给定的相机 $C_j$ ,我们检查可见性和尺度。为了确定可见性，我们简单地测试标注点中至少有一个在 $Points(C_j)$ 中。为了检查该注释在图像中是否在适当的比例，我们确定注释点集 $P_{ann}$ 在图像 $I_j$ 中的表观大小 $Size(P_{ann}, C_j)$。如果注释可见且 $0.05 < Size(Pann, C_j) < 0.8$ (以避免几乎不可见的注释和占据整个图像的注释）,我们将注释传输到 C_j。当用户访问 $C_j$ 时，注释作为点周围的框显示，如图 7 所示。

除了快速用语义信息增强场景之外，传输注释的能力还有几个应用。首先，它支持一种系统，其中游客可以拍一张照片（例如，从运行我们的软件的相机手机上）,并立即看到场景中对象的信息叠加在图像上。与头戴显示器结合使用，这样的功能可以提供一个高度便携的、基于计算机视觉的增强现实系统。其次，它使得为关键字搜索准备照片注释更有效。如果在一张照片中使用一组关键字对对象进行了注释，将注释传输到其他照片可以基于单个注释向关键字搜索数据库添加多个图像。

我们也可以利用已经存在的大量已注释图像。例如，在 Flickr 上，用户可以将注释附加到照片的矩形区域。ESP 游戏和 LabelMe 等工具鼓励用户对网上图像进行标注，并积累了注释数据库。通过使用我们的系统将这些标注图像与现有照片集合注册，我们可以将现有标签传输到系统中每个其他相关照片。网页上的其他图像也隐含注释：例如，Wikipedia 页面上的图像就带有该页面的 URL“注释”。通过注册这些图像，我们可以将其他照片链接到同一页面。

## 8 结果

我们使用几个数据集评估了我们的系统。前两个集合在更受控的设置中拍摄（即单人使用单个相机和镜头）:Prague,在两天内拍摄的 197 张布拉格老城广场的照片，以及 Great Wall,沿长城拍摄的 120 张照片（其中 82 张最终被注册）。

我们还用从 Flickr 下载的“非受控”图像集进行了实验。在每种情况下，我们的系统都检测和匹配整个图像集中的特征，并自动识别和注册与场景的一个连接组件对应的子集。我们注册的四个集合如下:Notre Dame 是一个包含 597 张图像的圣母院集合，它是从匹配“notredame AND paris”的 2,635 张照片中注册的。Yosemite 是一个包含 325 张半圆顶图像的集合，它是从匹配“halfdome AND yosemite”的 1,882 张照片中注册的。Trevi Fountain 是一个包含 360 张照片的许愿池集合，它是从匹配“trevi AND rome”的 466 张照片中注册的。Trafalgar Square 是一个包含 278 张照片的特拉法加广场集合，它是从匹配“trafalgarsquare”的 1893 张照片中注册的。

我们注册的每个数据集的平均重投影误差为 1.516 像素（我们注册的图像的平均长边和短边分别为 1,611 和 1,128 像素）。这些数据集的可视化如图 1-8 所示。请观看随附视频，其中展示了我们的照片资源管理器在几个数据集上的功能，包括对象选择、相关图像选择、形态学和注释传输。

对于半圆顶数据集，在最初构建模型后，我们使用第 4.3.1 节所述的方法将其与数字高程模型对齐。然后我们使用第 7.1 节中描述的方法将一张历史照片 Ansel Adams 的“Moon and Half Dome”注册到数据集中，方法是将其拖放到模型上。图 8 显示了从 Ansel Adams 拍摄照片的估计位置合成场景的渲染结果。

## 9 讨论和未来工作

我们提出了一种新颖的端到端系统，可以获取无序照片集合，对其进行注册，并以新颖的交互式 3D 浏览器向用户呈现。我们在从互联网和个人照片收藏中收集的几组大型照片集上评估了我们的重建算法和探索界面。

对于从互联网上收集的数据集，我们的重建算法成功地注册了很大一部分照片。未能注册的大多数照片属于与重建部分断开的场景的部分，但其他一些照片由于其他因素无法匹配，例如过度模糊、噪声、曝光不足或与其他照片重叠太少。图 9 显示了一些我们的系统无法注册的许愿池数据集的代表性图像。


我们的重建算法也存在几个限制，我们希望在未来解决。随着注册相机数量的增长，我们当前的 SfM 实现变得缓慢。我们希望加快过程，例如通过更好的选择注册照片的顺序。一个更有效的顺序可能先用少量视图重建大尺度场景结构，然后使用局部优化注册剩余视图。另外，我们可以使用分区方法改进效率。我们的 SfM 过程也不能保证在没有地面控制点的情况下获得场景的完全度量重建，这使得获得非常精确的模型更具挑战性。随着更多的地理参考数据的出现，这个问题将得到缓解。我们的相机模型不处理镜头畸变，导致更大的重建误差；我们计划在未来使用更复杂的模型。某些场景的自动重建非常具有挑战性，特别是那些具有重复结构或没有强特征的场景。最后，我们的算法目前只重建输入图像的一个连接组件。我们计划扩展它以重建输入中存在的所有结构。

我们的探索器接口也可以在几个方面改进。对于大量照片，场景可能会因视锥体太多而变得杂乱。为了缓解这种问题，我们希望探索针对具有某些属性的照片进行聚类并仅显示那些照片的方法。更好的形态学和视图插值技术将提供更具吸引力的临场感。

最终，我们希望将我们的重建算法扩展到处理数百万张照片并维护不同场景和注释的数据库目录。我们设想一个系统，它可以自动在网上搜索照片，并确定新照片如何适配数据库。这样一个系统还可以利用现有的地理参考数据对照片进行地理注册。

## 参考文献

??? info "References"

	ALIAGA, D., FUNKHOUSER, T., YANOVSKY, D., & CARL BOM, I. (2003). Sea of images. IEEE Computer Graphics and Applications, 23(6), 22–30.

	ALIAGA, D., YANOVSKY, D., FUNKHOUSER, T., & CARL BOM, I. (2003). Interactive image-based rendering using feature globalization. In Proc. SIGGRAPH Symposium on Interactive 3D Graphics, 163–170.

	ARYA, S., MOUNT, D. M., NETANYAHU, N. S., SILVERMAN, R., & WU, A. Y. (1998). An optimal algorithm for approximate nearest neighbor searching in fixed dimensions. Journal of the ACM, 45(6), 891–923.

	BROWN, M., & LOWE, D. G. (2005). Unsupervised 3D object recognition and reconstruction in unordered datasets. In Proc. Int. Conf. on 3D Digital Imaging and Modeling, 56–63.

	BUEHLER, C., BOSSE, M., MCMILLAN, L., GORTLER, S., & COHEN, M. (2001). Unstructured lumigraph rendering. In SIGGRAPH Conf. Proc., 425–432.

	CHEN, S., & WILLIAMS, L. (1993). View interpolation for image synthesis. In SIGGRAPH Conf. Proc., 279–288.

	CHEW, L. P. (1987). Constrained Delaunay triangulations. In Proc. Symposium on Computational Geometry, 215–222.

	COOPER, M., FOOTE, J., GIRGENSOHN, A., & WILCOX, L. (2003). Temporal event clustering for digital photo collections. In Proc. ACM Int. Conf. on Multimedia, 364–373.

	DEBEVEC, P. E., TAYLOR, C. J., & MALIK, J. (1996). Modeling and rendering architecture from photographs: a hybrid geometry- and image-based approach. In SIGGRAPH Conf. Proc., 11–20.

	DICK, A. R., TORR, P. H. S., & CIPOLLA, R. (2004). Modelling and interpretation of architecture from several images. International Journal of Computer Vision, 60(2), 111–134.

	FEINER, S., MACINTYRE, B., HOLLERER, T., & WEBSTER, A. (1997). A touring machine: Prototyping 3D mobile augmented reality systems for exploring the urban environment. In Proc. IEEE Int. Symposium on Wearable Computers, 74–81.

	FISCHLER, M., & BOLLES, R. (1987). Random sample consensus: a paradigm for model fitting with applications to image analysis and automated cartography. Readings in Computer Vision: Issues, Problems, Principles, and Paradigms, 726–740.

	GORTLER, S. J., GRZESZCZUK, R., SZELISKI, R., & COHEN, M. F. (1996). The Lumigraph. In SIGGRAPH Conf. Proc., 43–54.

	GRZESZCZUK, R. (2002). Course 44: Image-based modeling. In SIGGRAPH 2002.

	HARTLEY, R. I., & ZISSERMAN, A. (2004). Multiple View Geometry. Cambridge University Press, Cambridge, UK.

	IRANI, M., & ANANDAN, P. (1998). Video indexing based on mosaic representation. IEEE Trans. on Pattern Analysis and Machine Intelligence, 20(8), 905–921.

	JOHANSSON, B., & CIPOLLA, R. (2002). A system for automatic pose-estimation from a single image in a city scene. In Proc. IASTED Int. Conf. Signal Processing, Pattern Recognition and Applications.

	KADOBAYASHI, R., & TANAKA, K. (2005). 3D viewpoint-based photo search and information browsing. In Proc. ACM Int. Conf. on Research and Development in Information Retrieval, 621–622.

	LEVOY, M., & HANRAHAN, P. (1996). Light field rendering. In SIGGRAPH Conf. Proc., 31–42.

	LIPPMAN, A. (1980). Movie maps: An application of the optical videodisc to computer graphics. In SIGGRAPH Conf. Proc., 32–43.

	LOURAKIS, M., & ARGYROS, A. (2004). The design and implementation of a generic sparse bundle adjustment software package based on the Levenberg-Marquardt algorithm. Tech. Rep. 340, Inst. of Computer Science-FORTH, Heraklion, Crete, Greece. Available from www.ics.forth.gr/òlourakis/sba.

	LOWE, D. (2004). Distinctive image features from scale-invariant keypoints. International Journal of Computer Vision, 60(2), 91–110.

	MCCURDY, N., & GRISWOLD, W. (2005). A systems architecture for ubiquitous video. In Proc. Int. Conf. on Mobile Systems, Applications, and Services, 1–14.

	MCMILLAN, L., & BISHOP, G. (1995). Plenoptic modeling: An image-based rendering system. In SIGGRAPH Conf. Proc., 39–46.

	MIKOLAJCZYK, K., TUYTELAARS, T., SCHMID, C., ZISSERMAN, A., MATAS, J., SCHAFFALITZKY, F., KADIR, T., & VAN GOOL, L. (2005). A comparison of affine region detectors. International Journal of Computer Vision, 65(1/2), 43–72.

	NAAMAN, M., PAEPCKE, A., & GARCIA-MOLINA, H. (2003). From where to what: Metadata sharing for digital photographs with geographic coordinates. In Proc. Int. Conf. on Cooperative Information Systems, 196–217.

	NAAMAN, M., SONG, Y. J., PAEPCKE, A., & GARCIA-MOLINA, H. (2004). Automatic organization for digital photographs with geographic coordinates. In Proc. ACM/IEEE-CS Joint Conf. on Digital Libraries, 53–62.

	NOCEDAL, J., & WRIGHT, S. J. (1999). Numerical Optimization. Springer Series in Operations Research. Springer-Verlag, New York, NY.

	POLLEFEYS, M., VAN GOOL, L., VERGAUWEN, M., VERBIEST, F., CORNELIS, K., TOPS, J., & KOCH, R. (2004). Visual modeling with a handheld camera. International Journal of Computer Vision, 59(3), 207–232.

	ROBERTSON, D. P., & CIPOLLA, R. (2002). Building architectural models from many views using map constraints. In Proc. European Conf. on Computer Vision, vol. II, 155–169.

	RODDEN, K., & WOOD, K. R. (2003). How do people manage their digital photographs? In Proc. Conf. on Human Factors in Computing Systems, 409–416.

	ROMAN, A., GARG, G., & LEVOY, M. (2004). Interactive design of multi-perspective images for visualizing urban landscapes. In Proc. IEEE Visualization, 537–544.

	RUSSELL, B. C., TORRALBA, A., MURPHY, K. P., & FREEMAN, W. T. (2005). Labelme: A database and web-based tool for image annotation. Tech. Rep. MIT-CSAIL-TR-2005-056, Massachusetts Institute of Technology.

	SCHAFFALITZKY, F., & ZISSERMAN, A. (2002). Multi-view matching for unordered image sets, or "How do I organize my holiday snaps?". In Proc. European Conf. on Computer Vision, vol. 1, 414–431.

	SCHMID, C., & ZISSERMAN, A. (1997). Automatic line matching across views. In Proc. IEEE Conf. on Computer Vision and Pattern Recognition, 666–671.

	SEITZ, S. M., & DYER, C. M. (1996). View morphing. In SIGGRAPH Conf. Proc., 21–30.

	SIVIC, J., & ZISSERMAN, A. (2003). Video Google: A text retrieval approach to object matching in videos. In Proc. Int. Conf. on Computer Vision, 1470–1477.

	STEEDLY, D., ESSA, I., & DELLEART, F. (2003). Spectral partitioning for structure from motion. In Proc. Int. Conf. on Computer Vision, 996–103.

	SZELISKI, R. (2005). Image alignment and stitching: A tutorial. Tech. Rep. MSR-TR-2004-92, Microsoft Research.

	TELLER, S., et al. (2003). Calibrated, registered images of an extended urban area. Int. J. of Computer Vision, 53(1), 93–107.

	TOYAMA, K., LOGAN, R., & ROSEWAY, A. (2003). Geographic location tags on digital images. In Proc. Int. Conf. on Multimedia, 156–166.

	VON AHN, L., & DABBISH, L. (2004). Labeling images with a computer game. In Proc. Conf. on Human Factors in Computing Systems, 319–326.

	ZITNICK, L., KANG, S. B., UYTTENDAELE, M., WINDER, S., & SZELISKI, R. (2004). High-quality video view interpolation using a layered representation. In SIGGRAPH Conf. Proc., 600–608.