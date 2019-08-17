# 3D-Detection-localization
Papers for 3D Detection


把最近自己看的一些3D检测论文做一个总结吧，省得以后会忘……
目前论文中看到的主要是基于相机、雷达以及雷达相机融合三种方式，主要方法都是运用深度学习来做的。

## 一、Camera
### 1.《3D Bounding Box Estimation Using Deep Learning and Geometry》

论文地址：https://arxiv.org/abs/1612.00496 

目前这个方法github上面完全实现的不多，而且基本运算也和论文中的有一些出入。
该方法的一些检测思路主要着基于相机的几何思维，关键的车辆3D bounding box的设计并没有在论文里准确的说清楚，大致应该是如下设计的初始框，与相机
坐标一致。

![Image text](https://github.com/WAN96/3D-Detection-localization/tree/master/image/1.png)

剩下的几个点：

##### 1).回归物体检测角度的确定
根据论文里描述：虽然随着本车与其他物体逐渐靠近，由于相机的成像原理，在图像中物体的行驶方向好像一直在变化，但实际上是没有发生变化的，通过单一回归
文章中设置的  来得到最终的检测物体方向。具体就是  

![Image text](https://github.com/WAN96/3D-Detection-localization/tree/master/image/2.png) ![Image text](https://github.com/WAN96/3D-Detection-localization/tree/master/image/3.png)

##### 2).Multi-bin
基本也是依据RPN的类似思想，把anchor换成角度值，通过输出给定角度值的概率确定大体角度，网络再另回归出角度的offset以补偿角度的不准确性。另一个输出就是box的offset，因为这里论文中也事先对不同类别的物体尺寸做了一个近似值，通过不同类别先确定一个大概尺寸，之后再通过offset补足检测的不准确性。

![Image text](https://github.com/WAN96/3D-Detection-localization/tree/master/image/4.png)

##### 3).没有回归计算检测目标的平移量
论文里只是回归出了目标的三维尺度，没有对物体位置平移量做回归，文章说这样回归计算十分耗时且精度下降。
we carried out experiments on regressing alternative parameters related to translation and found that choice of parameters matters: we obtained less accurate 3D box reconstructions using that parametrization. 
目标的平移量则是通过相机投影的2D-3D约束利用SVD最终求解出的。具体不展开了，该论文有一篇扩展，我也从github上找了一个比较详细的解释，但是都没有具体说明论文中的这个约束应该实际如何使用。

### 2.《Orthographic Feature Transform for Monocular 3D Object Detection》
论文地址：https://arxiv.org/abs/1811.08188 

利用网络近似训练出类似birdview的场景，通过置信度来确定物体的三维位置，实际的启发并不大,主要是积分图的使用比较有用，topdown net对后续的融合方法实际作用不大，主要是也没太看懂

这篇文章是对图像采用了voxel 体素格的思想，通过正交的方式投影到类似鸟瞰图的方位来估计物体的3D 数据。
这篇文章的整体思路较好，对于相机的几何成像运用得较好。

网络结构：

![Image text](https://github.com/WAN96/3D-Detection-localization/tree/master/image/5.png)

几个关键点：
##### 1).体素格的建立
这里主要是建立一个W,H,D 大小的空间，然后将空间均匀划分为等尺寸的3D 格子，将该格子投影到相机成像平面，并通过多尺度的平面特征提取，以此来获取体素格的特征值，文章里的把3D 盒投影近似看作一个矩形。

![Image text](https://github.com/WAN96/3D-Detection-localization/tree/master/image/6.png)

在获得了3D 空间特征后，利用top down network 将其投影至鸟瞰平面，这里的用的网络结构是resnet-16.
The orthographic feature map is obtained by summing voxel features along the vertical

![Image text](https://github.com/WAN96/3D-Detection-localization/blob/master/image/7.png)

axis after multiplication with a set of learned weight matrices W(y) 2 Rn n:


##### 2).积分图的使用
考虑到利用这种方法，在平面上投影的实际ROI 过多，大约在150K 个，与faster-RCNN 的2K 个相比实在太多，为了减少运算量，文章里运用了类似Harr 特征里使用的积分图特性来加快特征运算。

##### 3).运用了confidence map 的方式来定位物体
用了这篇文章的方法《Densebox: Unifying landmark localization with end to end
object detection》利用概率的大小来定位，之后再用近似的边界框来回归修正

### 3.《GS3D: An Efficient 3D Object Detection Framework for Autonomous Driving》
论文地址：https://arxiv.org/abs/1903.10955 
这篇论文主要说了一个在图像中表达3D信息具有一定的模糊性，于是该文章根据预先由设计好的网络生成的2D框以及方向先产生一个近似正确的3D框，之后对这个框在图像上投影的三个面分别进行特征学习以此来提高检测精度。

![Image text](https://github.com/WAN96/3D-Detection-localization/tree/master/image/8.png)

其他还有
《Pseudo-LiDAR from Visual Depth Estimation: Bridging the Gap in 3D Object Detection for Autonomous Driving》 https://arxiv.org/abs/1812.07179 和《Accurate Monocular 3D Object Detection via Color-Embedded 3D Reconstruction for Autonomous Driving》 https://arxiv.org/abs/1903.11444 
在https://zhuanlan.zhihu.com/p/41460767 都有讲解，就不重复造轮子了。
