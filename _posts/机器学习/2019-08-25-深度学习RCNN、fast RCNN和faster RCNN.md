---
layout:     post
title:      深度学习RCNN、fast RCNN和faster RCNN
subtitle:   深度学习RCNN、fast RCNN和faster RCNN
date:       2019-08-25
author:     正版慕言
header-img: img/blog_bg_1.jpg
catalog: true
mathjax: true
tags:
    - 机器学习
    - 深度学习

---

项目源码：[RCNN](https://github.com/rbgirshick/rcnn),[fast RCNN](https://github.com/rbgirshick/fast-rcnn),[faster RCNN](https://github.com/ShaoqingRen/faster_rcnn)

# RCNN

> RCNN(Region CNN) 利用深度学习进行目标检测的开山之作。

> 解决了目标检测中的两个关键问题：

> 1. 速度。经典的目标检测算法使用滑动窗口检测所有可能的区域，RCNN先提取可能是物体的候选区域，然后仅在候选区域提取特征和判断。
> 2. 训练集。经典的目标检测算法需要人工提取特征，RCNN训练深度网络提取。

#### RCNN步骤

RCNN分为4个步骤：

1. 生成候选区域：在一张图上生成1k至2k个候选区域
2. 特征提取：对每个候选区域用CNN提取特征
3. 类别判断：将特征送入每一类的SVM分类器判断是否属于该类
4. 位置精修：用回归器修正候选框位置

![RCNN流程](/img/机器学习/RCNN流程.png)

#### 候选区域生成

候选区域生成使用Selective Search的方法：

1. 使用一种过分割的手段，从一张图中生成约2k张图
2. 合并可能性最高的两个区域直到整张图像合并成一个区域位置
3. 输出所有存在过得区域，即为候选区域

#### 候选区域合并

合并规则：

1. 颜色相近
2. 纹理相近
3. 合并后总面积小的：避免一大块区域吃掉其他小区域
4. 合并后总面积在BBOX(bounding box)中占比大的：保证合并后形状的规则

![形状合并](/img/机器学习/形状合并.png)

物体检测要定位物体的bounding box，还要识别出bounding box里的物体就是车辆。

#### 重叠度IOU

IOU定义了两个bounding box的重叠度，$IOU = (A \cap B) / (A \cup B) $

![IOU](/img/机器学习/IOU.png)

#### 非极大值抑制

RCNN会从一张图中找出n个可能是物体的矩形框，然后做类别分类概率。在这些矩形中，要判断哪些框是没用的。

![RCNN找到n个矩形框](/img/机器学习/RCNN找到n个矩形框.png)

非极大值抑制的方法是：对于6个框，根据类别分类概率排序，假设概率从小到大排序：ABCDEF

1. 从F开始分别判断A~E与F的IOU是否大于设定的阈值
2. 假设B,D与F的IOU超过阈值，扔掉，标记F是保留下来的
3. 从剩下的ACE中选择概率最大的E， 判断其与AC的IOU，超过阈值的扔掉，标记E是保留下来的。
4. 一直重复，找到所有保留下来的框。

#### 候选框的搜索

Selective Search搜索出来的候选框大小是不同的，需要缩放到固定大小才能送进CNN。原文试验了两种方法：

1. 各向异性缩放：不管长宽比例和扭曲直接缩放
2. 各向同性缩放：
	1. 先扩充后裁剪
	2. 先裁剪后扩充

## RCNN的训练

#### CNN特征提取阶段

**CNN的架构选择** ：有两个方案：

1. AlexNet
2. VGG16

**监督训练** ：因为数据量少，不能采用随机初始化权重。使用已经训练好的模型作为预训练模型。

**fine-turning** ：用selective search出的候选框对上一步确定的模型进行fine-turning训练。

**正负样本问题** ：用IOU为bounding box打标签。如果候选框与标注的IOU大于0.5，认为是正样本，否则是负样本。

#### SVM训练和测试

CNN提取特征后，为每一类训练一个SVM分类器。
如果CNN提取了2000个候选框，那么会产生一个2000*4096的特征矩阵，吧这个矩阵与SVM权值矩阵4096*N 点乘，就能得到结果。

测试阶段：相同的操作

# fast RCNN

fast RCNN解决了RCNN的三个问题：

1. 测试速度慢
2. 训练速度慢
3. 所需空间大

## 特征提取阶段

提出了一个叫做ROI pooling的层，可以把不同大小的输入映射到一个尺度上。它将每个候选区域均匀分成MxN块，对每一块进行max pooling。这样对每个区域都提取一个固定大小维度的特征，就可以通过softmax进行识别。

![ROI pooling](/img/机器学习/ROIpooling.png)

## 分类回归阶段

fast RCNN将最后的bbox regression也放在神经网络内部，合并成了一个multi-task模型：

![fast-RCNN](/img/机器学习/fast-RCNN.png)

# faster RCNN

fast RCNN的瓶颈：selective search找到所有的候选框很耗时

faster RCNN加入了一个提取边缘的神经网络，将寻找候选框的工作也交给神经网络。

![faster-RCNN](/img/机器学习/faster-RCNN.png)

用区域生成网络(Region Proposal Network, RPN)代替了selective search。

![RPN](/img/机器学习/RPN.png)

RPN的工作步骤：

1. 在特征图上滑动窗口
2. 建立一个神经网络用来进行物体分类和框位置的回归
3. 滑动窗口的位置提供物体的大体位置信息
4. 框的回归提供更精确的位置