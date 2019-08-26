---
layout:     post
title:      GBDT和XGBoost的理解
subtitle:   GBDT和XGBoost的理解
date:       2019-08-24
author:     正版慕言
header-img: img/blog_bg_1.jpg
catalog: true
mathjax: true
tags:
    - GBDT
    - XGBoost

---

## GBDT

GBDT分为两个部分：GB(Gradient Boosting,梯度提升)和DT(Decision tree,决策树)。

### 决策树

GBDT中采用的弱学习器是决策树。决策树采用的是树形结构，通常包含特征选择、决策树生成和决策树剪枝三个部分。决策树中的非叶子节点表示一个特征或属性，叶子节点表示一个分类。其本质是归纳出一组分类规则

#### 特征选择：

**信息熵与信息增益**：

决策树学习的关键在于选择最优的划分属性。
**信息熵** 是度量样本集合纯度的一种指标，信息熵的值越小，样本集合的纯度越高。
**信息增益** 表示使用某一属性进行划分获得的纯度提升。信息增益越大，纯度提升越大。

假设数据集D的信息熵为$Ent(D)$，特征$\alpha$给定后的信息熵为$End(D\|\alpha)$，则特征$\alpha$对数据集D的信息增益为

$$
Gain(D,\alpha) = Ent(D) - Ent(D|\alpha)
$$

**信息增益比**：用来矫正使用信息增益作为划分训练数据集的特征时存在的偏向数据较多的特征的问题。
信息增益比的公式：

$$
Gain_R(D,\alpha) = \frac{Gain(D,\alpha)}{Ent_A(D)}
$$

**基尼指数** 衡量数据集D的纯度

$$Gini(D) = 1 - \sum_{k=1}^{|y|} p_k^2 $$

对于属性$\alpha$的基尼指数，可以定义为

$$Gini(D,\alpha) = \sum_{v=1}^V \frac{|D^v|}{|D|}Gini(D^v) $$

#### 剪枝

剪枝是处理决策树过拟合的主要手段，有预剪枝和后剪枝。

预剪枝是在生成过程中对节点先估计，如果不能提升泛化性能就停止划分并标记为叶子节点
后剪枝是先生成完整决策树，再自底向上进行考察，如果将当前非叶子节点替换成叶子节点可以提升泛化性能，就替换成叶子节点。

### 分类回归树CART

CART假设决策树是二叉树，特征取值只有0和1。

CART和决策树的差别在于CART的叶子节点只包含判断分数

CART的组成：

1. 决策树的生成：用训练数据集生成决策树，生成的决策树要尽量大
2. 决策树的剪枝：用验证数据集进行剪枝

### 提升树

用加法模型和前项分布算法，以决策树为基函数的提升方法

提升树的模型可以表示成M个决策树的加法模型：

$$f_M(x) = \sum_{m=1}^MT(x;\theta_m) $$

$\theta_m$是决策树的参数。

给定loss函数$L(y,f(y))$之后，$f_m(x)$变成一个损失函数极小化问题：

$$\min_{\theta_m} = \sum_{i=1}^NL(y_i, f_M(x)) $$

可以通过前项分步算法求解该问题

$$f_m(x) = f_{m-1}(x) + T(x;\theta_m) $$

每次通过经验风险极小化来确定下一棵决策树的参数$\theta_m$。

$$\hat \theta_m = arg \min_{\theta_m} \sum_{i=1}^NL(y_i, f_{m-1}(x_i) + T(x_i;\theta_m)) $$

这种线性组合的决策树能够很好地拟合较为复杂的训练数据。

### GBDT

梯度提升算法：利用损失函数的负梯度在当前模型的值

$$r_{mi}} = -[\frac{\partial L(y_i, f(x))}{\partial f(x)}]_{f(x) = f_{m-1}(x)} $$

作为回归提升树算法中残差的近似值来拟合一个回归树。

#### 梯度近似

通过把样本带入模型可以得到$f_{m-1}(x_i) $，获得损失$Loss(y_i, f_{m-1}(x_i)) $
从而能够求出梯度下降的步长$r_m$，然后更新$f_m(x)$

$$f_{m}(x) = f_{m-1}(x) - r_{m}\sum_{i=1}{N}\delta L(y_{i}, f_{m-1}(x)) $$


## XGBoost

XGBoost与GBDT模型一致，目标函数也是一致的，使用前向分步算法进行学习。不过在目标函数中添加了**正则项**

$$L(\phi) = \sum_il(\hat y_i, y_i) = \sum_k\Omega(f_k) $$

这样，在第t轮可以得到

$$Obj^{(t)} = \sum_{i=1}^nl(y_i, \hat y_i^{(t-1)} + f_t(x_i)) + \Omega(f_t) + constant$$

XGBoost对误差函数进行二阶泰勒展开（我们可以自定义损失函数，但要保证二阶可导），得到新的目标函数

$$\sum_{i=1}^n[g_{i}f_{t}(x_{i}) + \frac{1}{2}h_{i}f_{t}^2(x_{i})] + \Omega(f_{t})$$

![二阶泰勒展开](/img/Journal/CTR/xgboost-误差函数二阶泰勒展开.png)

重新定义树结构：

用叶子节点集合和叶子节点得分表示，每个样本都落在一个叶子节点上

$q(x)$表示样本落在叶子节点上，$w_q(x)$表示该节点的得分(即该样本的模型预测值)

数的复杂度项：$$\Omega(f_t) = \gamma T + \frac{1}{2}\sum_{j=1}T w_j^2 $$，T表示树的叶子节点的个数

$$Obj^{(t)} = \Sigma_{j=1}^T[(\Sigma_{i \in I_j}g_i)w_j + \frac{1}{2}(\Sigma_{i \in I_j} h_i + \lambda)w_j^2] + \gamma T $$

可以计算$w_j$：

$$w_j = - \frac{\Sigma_{i \in I_j}g_i}{\Sigma_{i \in I_j}h_i + \lambda} $$

原目标函数变成：

$$-\frac{1}{2} \sum_{j=1}^T\frac{(\Sigma_{i \in I_j}g_i)^2}{\Sigma_{i \in I_j}h_i + \lambda} + \gamma T$$

切分点的算法：

$$Gain = \frac{1}{2}[\frac{(\sum_{i\in I_{L}}g_{i})^2}{\sum_{i\in I_{L}}h_{i} + \lambda} + \frac{(\sum_{i\in I_{R}}g_{i})^2}{\sum_{i\in I_{R}}h_{i} + \lambda} - \frac{(\sum_{i\in I_{j}}g_{i})^2}{\sum_{i\in I_{j}}h_{i} + \lambda}] - \gamma $$

## GBDT和XGBoost的区别

1. GBDT用CART(分类回归树)做分类器，XGBoost支持线性分类器。
2. GBDT只用到一阶导数信息，XGBoost进行了泰勒展开，用到了一阶和二阶导数，并支持自定义损失函数。
3. XGBoost在损失函数里增加了正则项来控制模型复杂度。正则项里包括树的叶子节点个数、每个叶子节点的输出score的L2模的平方和。
4. XGBoost每次迭代完成后，会将叶子节点的权重乘一个系数来缩减每棵树的影响。
5. 借鉴随机森林，支持列抽样，降低过拟合。
6. 对特征值有缺失的样本，XGBoost可以自动学习它的分裂方向。
7. 在特征层面支持并行。XGBoost训练前先对数据进行排序，在后面的迭代中重复使用这个排序结构，减少计算量，同时这个结构也提供并行的基础。进行节点分列时，对特征的增益计算可以多线程并行。
8. 有一种可并行的近似直方图算法。