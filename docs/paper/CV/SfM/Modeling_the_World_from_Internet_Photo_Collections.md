---
date: 2023-08-25 21:25:09
status: new
---

!!! info "论文链接"
	原文：Noah Snavely, Steven M. Seitz, Richard Szeliski. [Modeling the World from Internet Photo Collections](https://doi.org/10.1007/s11263-007-0107-3)


!!! info "参考链接"
	转载自：https://zhuanlan.zhihu.com/p/75685722

### 1. Introduction

计算机视觉领域，一个关键的问题就是配准，即找出两个图像间的对应关系它们在一个公共的 3D 坐标系下的相对位姿问题。

### 2. Previous Work

### 2.1 Feature Correspondence

过去几十年的发展为现代特征检测和匹配技术发展奠定了基础，L-K 光流跟踪、 Harris 角点等。然而这些技术都依赖与特征点周围的图像块，即只是局部特征，因此在相似场景中的应用。Shi 和 Tomasi 提出了一个经过仿射扩展的 L-K 光流跟踪器，然而对于真实情况中的大基线匹配并不适用。本文使用的是一种更鲁棒的方法，由 Lowe 提出的 SIFT。

### 2.2 Structure from Motion

SfM 旨在通过一些列的特征对应关系同步重建未知 3D 场景结构及相机位姿。近年来，摄影测量技术，比如 BA，开始进入计算视觉领域，BA 被看做是执行三维重优化的黄金准则。作者发现，更好的估计 BA 优化中相机焦距可以产生更好的效果。本文是第一个成功的将 SfM 技术应用于各种各样的现实世界中的图片集的方法，其典型图片集来自数百种不同的相机、缩放尺度、分辨率、不同时间、光照、天气等。

### 2.3 Image-Based Modeling(IBM)

Image-Based Modeling 是一个通过输入的图像集合产生三维模型的过程。作者的工并不是为了进行三维可视化交互用的，而仅仅是用于重建一个稀疏的 3D 世界模型。

### 2.4 Image-Based Rendering(IBR)

The field of image-based rendering (IBR) is devoted to the problem of synthesizing new views of a scene from a set of input photographs.

### 2.5 Image Browsing, Retrieval, and Annotation

如何组织图片的研究已经存在很多方法了，大多数方法使用元数据，比如关键字、摄影师、时间等作为一个基本的图像组织结构。另一种方法就是标注，标注的好处是特征的对应关可以更好的得到保证，并且可以在图像间具体的物体和区域上传递标注。与大多数 AR 方法不作者使用 2D 图像标注在图像间传递。

### 3. Overview

三维重建最大的挑战是从数以百计、数以千计的不同视角、光照、天气条件、分辨率等的图片集中匹配并重建三维信息。作者使用当前计算机视觉中的两个重要的突破，即特征匹配和 SfM。作者的工作的一个关键特性就是可以自动的在图像间传递标注，因此一个图像中的物体信息可以被传播到所有包含该物体的其他图像中。

### 4. Reconstructing Cameras and Sparse Geometry

本文的方法不依赖于相机或其他设配来提供位姿、方位或几个信息，相反，而是通过使用计算机视觉技术从图片中获得这些信息。首先是在所有的图像中提取特征点，然后在图像对中匹配特征点，最后使用迭代优化的方法获得相机参数。

### 4.1 Keypoint Detection and Matching

作者使用 SIFT 检测子，该检测子同时提供了描述子。为了加速匹配，使用 ANN 进行匹配。为了降低误匹配，使用 SIFT 提出者使用的方法，即找出某个特征点的两个最近邻匹配，然后比较这两个距离的比值，如果比值 $\frac{d_1}{d_2} < 0.6$ 则认为是好的匹配点，否则删除这两个特征。在匹配好特征之后，使用 RANSAC 方法估计出较好的 Fundamental 矩阵，这里使用的是 8 点法，同通过阈值来剔除 outlier，最后 RANSAC 出的 Fundamental 矩阵在使用 inliers 进行非线性优化，方法为 L-M 优化。最后再次通过阈值剔除一些异常点。

### 4.2 Structure from Motion

下一步就是恢复一些列的相机参数，即旋转、平移及焦距。通过建立非线性最小而成问题通过使用 BA 可以求解这些参数。而非线性优化容易陷入局部极小值，所以给这些参数提一个较好的初始值至关重要。这里不是一次性的估计所有相机的参数，而是使用一增量的方法，逐步的增加相机。首先是找到一对匹配数量多的且基线大的图像，这就可以通过这两帧给出一个鲁棒的初始估计。位姿估计使用的是 5 点法。下一步从数据中选择那些拥有最对应的匹配点的图像已有位姿及对应的 3D 点图像作为下一待加入的图像。首先是运行 BA 优化，这里只允许优化当前帧的位姿及其观测到的 3D 点，先前加入的帧设置为固定，不参与优化。然后再将这些点和已加入进来的图像进行一个全局 BA 优化。然后重复此过程，直到所有可用的图像都处理完毕。这里并不是所有的图像都会使用，那些不符合要求的图像，比如匹配点太少等，不会用来进行三维重建。

在上述优化过程中，同时还会进行异常点的剔除工作，而且在添加新的图片时也并不是单纯只加入一张，而是加入多张。首先找出这多个图片中最大的匹配数量 M，然后提取不小于 0.75M 的图片作为新的图片加入到优化中。作者发现，同时估计径向畸变会给三维重建精度上带来很大的改善。为了防止畸变系数过大，在最小化方程中加入正则项 $\lambda(k_1^2+k_2^2)$ 来防它们过大。

### 9. Research Challenges

### Scale

互联网上图片非常多，然而有很多是冗余的，这意味着，我们可以仅使用这些图片的一小部进行高质量的三维重建。

### 9.1 Variability

虽然 SIFT 和其他特征匹配技术对外观变化有非常好的鲁棒性，但仍然还有很多场景下是有问题的。

### 9.2 Accuracy

大多数的 SfM 方法都是通过最小化重投影误差实现的，不能保证尺度精确度。而卫星图像、地图、数字高程度 (DEMs) 等可以提供精准的尺度数据，这些可以用来获得更精准的 SfM 结果。



## 参考文献

??? info "References"

	Akbarzadeh, A., Frahm, J.-M., Mordohai, P., et al. (2006). Towards urban 3D reconstruction from video. Proceedings of the international symposium on 3D data processing, visualization, and transmission.

	Aliaga, D. G. et al. (2003). Sea of images. IEEE Computer Graphics and Applications, 23(6), 22–30.

	Aliaga, D., Yanovsky, D., Funkhouser, T., & Carlbom, I. (2003). Interactive image-based rendering using feature globalization. Proceedings of the SIGGRAPH symposium on interactive 3D graphics, 163–170.

	Aloimonos, Y. (Ed.). (1993). Active perception. Mahwah: Lawrence Erlbaum Associates.

	Arya, S., Mount, D. M., Netanyahu, N. S., et al. (1998). An optimal algorithm for approximate nearest neighbor searching fixed dimensions. Journal of the ACM, 45(6), 891–923.

	Baumberg, A. (2000). Reliable feature matching across widely separated views. Proceedings of the IEEE conference on computer vision and pattern recognition, 774–781.

	Blake, A., & Yuille, A. (Eds.). (1993). Active vision. Cambridge: MIT Press.

	Brown, M., & Lowe, D. G. (2005). Unsupervised 3D object recognition and reconstruction in unordered datasets. Proceedings of the international conference on 3D digital imaging and modelling, 56–63.

	Buehler, C., Bosse, M., McMillan, L., et al. (2001). Unstructured lumigraph rendering. SIGGRAPH conference proceedings, 425–432.

	Chen, S., & Williams, L. (1993). View interpolation for image synthesis. SIGGRAPH conference proceedings, 279–288.

	Chew, L. P. (1987). Constrained Delaunay triangulations. Proceedings of the symposium on computational geometry, 215–222.

	Cooper, M., Foote, J., Girgensohn, A., & Wilcox, L. (2003). Temporal event clustering for digital photo collections. Proceedings of the ACM international conference on multimedia, 364–373.

	Debevec, P. E., Taylor, C. J., & Malik, J. (1996). Modeling and rendering architecture from photographs: a hybrid geometry- and imagebased approach. SIGGRAPH conference proceedings, 1120.

	Dick, A. R., Torr, P. H. S., & Cipolla, R. (2004). Modelling and interpretation of architecture from several images. International Journal of Computer Vision, 60(2), 111–134.

	Feiner, S., MacIntyre, B., Hollerer, T., & Webster, A. (1997). A touring machine: Prototyping 3D mobile augmented reality systems for exploring the urban environment. Proceedings of the IEEE international symposium on wearable computers, 74–81.

	Fergus, R., Fei-Fei, L., Perona, P., & Zisserman, A. (2005). Learning object categories from Google’s image search. Proceedings of the international conference on computer vision, 816–823.

	Fischler, M. A., & Bolles, R. C. (1981). Random sample consensus: A paradigm for model fitting with applications to image analysis and automated cartography. Communications of the ACM, 24(6), 381–395.

	Fitzgibbon, A. W., & Zisserman, A. Automatic camera recovery for closed and open image sequences. Proceedings of the European conference on computer vision, 311–326.

	Förstner, W. (1986). A feature-based correspondence algorithm for image matching. International Archives Photogrammetry & Remote Sensing, 26(3), 150–166.

	Goesele, M., Snavely, N., Seitz, S. M., et al. (2007, to appear). Multi-view stereo for community photo collections. Proceedings of the international conference on computer vision.

	Gortler, S. J., Grzeszczuk, R., Szeliski, R., & Cohen, M. F. (1996). The lumigraph. SIGGRAPH conference proceedings, 43–54.

	Grauman, K., & Darrell, T. (2005). The pyramid match kernel: discriminative classification with sets of image features. Proceedings of the international conference on computer vision, 1458–1465.

	Grzeszczuk, R. (2002). Course 44: image-based modeling. SIGGRAPH 2002.

	Hannah, M. J. (1988). Test results from SRI’s stereo system. Image understanding workshop, 740–744.

	Harris, C., & Stephens, M. J. (1988). A combined corner and edge detector. Alvey vision conference, 147–152.

	Hartley, R. I. (1997). In defense of the eight-point algorithm. IEEE Transactions on Pattern Analysis and Machine Intelligence, 19(6), 580–593.

	Hartley, R. I., & Zisserman, A. (2004). Multiple view geometry. Cambridge: Cambridge University Press.

	Hays, J., & Efros, A. A. (2007). Scene completion using millions of photographs. SIGGRAPH conference proceedings.

	Irani, M., & Anandan, P. (1998). Video indexing based on mosaic representation. IEEE Transactions on Pattern Analysis and Machine Intelligence, 20(5), 905–921.

	Johansson, B., & Cipolla, R. (2002). A system for automatic poseestimation from a single image in a city scene. Proceedings of the IASTED international conference on signal processing, pattern recognition and applications.

	Kadir, T., & Brady, M. (2001). Saliency, scale and image description. International Journal of Computer Vision, 45(2), 83–105.

	Kadobayashi, R., & Tanaka, K. (2005). 3D viewpoint-based photo search and information browsing. Proceedings of the ACM international conference on research and development in information retrieval, 621–622.

	Lalonde, J.-F., Hoiem, D., Efros, A. A., et al. (2007). Photo clip art. SIGGRAPH conference proceedings.

	Levoy, M., & Hanrahan, P. (1996). Light field rendering. SIGGRAPH conference proceedings, 31–42.

	Lippman, A. (1980). Movie maps: an application of the optical videodisc to computer graphics. SIGGRAPH conference proceedings, 32–43.

	Longuet-Higgins, H. C. (1981). A computer algorithm for reconstructing a scene from two projections. Nature, 293, 133–135.

	Lourakis, M., & Argyros, A. (2004). The design and implementation of a generic sparse bundle adjustment software package based on the Levenberg–Marquardt algorithm (Technical Report 340). Inst. of Computer Science-FORTH, Heraklion, Crete, Greece.

	Lowe, D. (2004). Distinctive image features from scale-invariant keypoints. International Journal of Computer Vision, 60(2), 91–110.

	Lucas, B. D., & Kanade, T. (1981). An iterative image registration technique with an application in stereo vision. International joint conference on artificial Intelligence, 674–679.

	Matas, J. et al. (2004). Robust wide baseline stereo from maximally stable extremal regions. Image and Vision Computing, 22(10), 761–767.

	McCurdy, N., & Griswold, W. (2005). A systems architecture for ubiquitous video. Proceedings of the international conference on mobile systems, applications, and services, 1–14.

	McMillan, L., & Bishop, G. (1995) Plenoptic modeling: An imagebased rendering system. SIGGRAPH conference proceedings, 39–46.

	Mikolajczyk, K., & Schmid, C. (2004). Scale & affine invariant interest point detectors. International Journal of Computer Vision, 60(1), 63–86.

	Mikolajczyk, K., Tuytelaars, T., Schmid, C., et al. (2005). A comparison of affine region detectors. International Journal of Computer Vision, 65(1/2), 43–72.

	Moravec, H. (1983). The Stanford cart and the CMU rover. Proceedings of the IEEE, 71(7), 872–884.

	Naaman, M., Paepcke, A., & Garcia-Molina, H. (2003). From where to what: Metadata sharing for digital photographs with geographic coordinates. Proceedings of the international conference on cooperative information systems, 196–217.

	Naaman, M., Song, Y. J., Paepcke, A., & Garcia-Molina, H. (2004). Automatic organization for digital photographs with geographic coordinates. Proceedings of the ACM/IEEE-CS joint conference on digital libraries, 53–62.

	Nistér, D. (2000). Reconstruction from uncalibrated sequences with a hierarchy of trifocal tensors. Proceedings of the European conference on computer vision, 649–663.

	Nistér, D. (2004). An efficient solution to the five-point relative pose problem. IEEE Transactions on Pattern Analysis and Machine Intelligence, 26(6), 756–777.

	Nistér, D., & Stewénius, H. (2006). Scalable recognition with a vocabulary tree. Proceedings of the IEEE conference on computer vision and pattern recognition, 2118–2125.

	Nocedal, J., & Wright, S. J. (1999). Springer series in operations research. Numerical optimization. New York: Springer.

	Oliensis, J. (1999). A multi-frame structure-from-motion algorithm under perspective projection. International Journal of Computer Vision, 34(2–3), 163–192.

	Pollefeys, M., Koch, R., & Van Gool, L. (1999). Self-calibration and metric reconstruction in spite of varying and unknown internal camera parameters. International Journal of Computer Vision, 32(1), 7–25.

	Pollefeys, M., & Van Gool, L. (2002). From images to 3D models. Communications of the ACM, 45(7), 50–55.

	Pollefeys, M., van Gool, L., Vergauwen, M., et al. (2004). Visual modeling with a hand-held camera. International Journal of Computer Vision, 59(3), 207–232.

	Robertson, D. P., & Cipolla, R. (2002). Building architectural models from many views using map constraints. Proceedings of the European conference on computer vision, 155–169.

	Rodden, K., & Wood, K. R. (2003). How do people manage their digital photographs? Proceedings of the conference on human factors in computing systems, 409–416.

	Román, A., et al. (2004). Interactive design of multi-perspective images for visualizing urban landscapes. IEEE visualization, 537–544.

	Russell, B. C., Torralba, A., Murphy, K. P., & Freeman, W. T. (2005). Labelme: a database and web-based tool for image annotation (Technical Report MIT-CSAIL-TR-2005-056). Massachusetts Institute of Technology.

	Schaffalitzky, F., & Zisserman, A. (2002). Multi-view matching for unordered image sets, or “How do I organize my holiday snaps?” Proceedings of the European conference on computer vision, 414–431.

	Scharstein, D., & Szeliski, R. (2002). A taxonomy and evaluation of dense two-frame stereo correspondence algorithms. International Journal of Computer Vision, 47(1), 7–42.

	Schindler, G., Dellaert, F., & Kang, S. B. (2007). Inferring temporal order of images from 3D structure. Proceedings of the IEEE conference on computer vision and pattern recognition.

	Schmid, C., & Zisserman, A. (1997). Automatic line matching across views. Proceedings of the IEEE conference on computer vision and pattern recognition, 666–671.

	Seitz, S. M., & Dyer, C. M. (1996). View morphing. SIGGRAPH conference proceedings, 21–30.

	Seitz, S., Curless, B., Diebel, J., et al. (2006). A comparison and evaluation of multi-view stereo reconstruction algorithms. Proceedings of the IEEE conference on computer vision and pattern recognition, 519–526.

	Shi, J., & Tomasi, C. (1994). Good features to track. Proceedings of the IEEE conference on computer vision and pattern recognition, 593–600.

	Sivic, J., & Zisserman, A. (2003). Video Google: a text retrieval approach to object matching in videos. Proceedings of the international conference on computer vision, 1470–1477.

	Snavely, N., Seitz, S. M., & Szeliski, R. (2006). Photo tourism: exploring photo collections in 3D. ACM Transactions on Graphics, 25(3), 835–846.

	Spetsakis, M. E., & Aloimonos, J. Y. (1991). A multiframe approach to visual motion perception. International Journal of Computer Vision, 6(3), 245–255.

	Strecha, C., Tuytelaars, T., & Van Gool, L. (2003). Dense matching of multiple wide-baseline views. Proceedings of the international conference on computer vision, 1194–1201.

	Szeliski, R. (2006). Image alignment and stitching: a tutorial. Foundations and Trends in Computer Graphics and Computer Vision, 2(1).

	Szeliski, R., & Kang, S. B. (1994). Recovering 3D shape and motion from image streams using nonlinear least squares. Journal of Visual Communication and Image Representation, 5(1), 10–28.

	Szeliski, R., & Zabih, R., et al. (2006). A comparative study of energy minimization methods for Markov random fields. Proceedings of the European conference on computer vision, 16–29.

	Tanaka, H., Arikawa, M., & Shibasaki, R. (2002). A 3D photo collage system for spatial navigations. Revised papers from the second Kyoto workshop on digital cities II, computational and sociological approaches, 305–316.

	Teller, S., Antone, M., Bodnar, Z., et al. (2003). Calibrated, registered images of an extended urban area. International Journal of Computer Vision, 53(1), 93–107.

	Tomasi, C., & Kanade, T. (1992). Shape and motion from image streams under orthography: a factorization method. International Journal of Computer Vision, 9(2), 137–154.

	Toyama, K., Logan, R., & Roseway, A. (2003). Geographic location tags on digital images. Proceedings of the international conference on multimedia, 156–166.

	Triggs, B., et al. (1999). Bundle adjustment—a modern synthesis. International workshop on vision algorithms, 298–372.

	Tuytelaars, T., & Van Gool, L. (2004). Matching widely separated views based on affine invariant regions. International Journal of Computer Vision, 59(1), 61–85.

	Vergauwen, M., & Van Gool, L. (2006). Web-based 3D reconstruction service. Machine Vision and Applications, 17(2), 321–329.

	von Ahn, L., & Dabbish, L. (2004). Labeling images with a computer game. Proceedings of the conference on human factors in computing systems, 319–326.

	Zitnick, L., Kang, S. B., Uyttendaele, M., et al. (2004). High-quality video view interpolation using a layered representation. SIGGRAPH conference proceedings, 600–608.