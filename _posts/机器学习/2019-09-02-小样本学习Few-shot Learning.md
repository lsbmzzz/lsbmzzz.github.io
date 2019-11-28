---
layout:     post
title:      小样本学习Few-shot Learning
subtitle:   小样本学习Few-shot Learning
date:       2019-09-02
author:     正版慕言
header-img: img/blog_bg_1.jpg
catalog: true
mathjax: true
tags:
    - 机器学习

---

Few-shot Learning是Meta Learning在监督学习中的应用。

# Meta Learning

Meta Learning又叫做learning to learn，在meta training阶段将数据集分解成不同的meta task，学习类别变化情况下模型的泛化能力，在meta testing阶段面对新的类型，不用变动已有类别就可以完成分类。

元学习的思想是学习“学习的过程”。

人类可以快速学习的原因：

1. 先验知识
2. 快速适应

因此我们的问题就是如何给予模型总结先验知识和将其用于快速应用于一个新任务的能力。

在一般的学习任务中，我们关注数据的标签；而在元学习中，我们尝试用机器从少量数据中更快的学习。

训练元学习器需要有一个学习器和一个训练器（元学习器）。学习器的任务是快速利用少量的数据学习新任务。这个学习器可以用院学习器训练，从而能从大量任务中学习。学习的过程是元学习器不断向学习器展示不同的任务，最后学习器就会学到众多任务知识。

元学习的训练数据并非以batch_size的形式来组织，而是set的形式。

首先我们有一个支持集support set，它由一些属于样本集的图像构成。然后我们要有一个目标集target set，作为被分类的图像。支持集和目标集共同组成一个训练的episode。

元学习器学习各种训练集，将它们一个一个的展示给学习器，让学习器尝试对每一个训练集中的目标集图像正确分类。

![元学习器的训练集](/img/机器学习/Metalearning_dataset.jpg)

在meta learning中，我们每次不向快速学习器展示所有类别。我们希望在少数类别中只展示少量图像时，模型能够得到正确的预测结果。元学习器每次给快速学习器不同的子集，快速学习器可以给每个子集快速分类。另外，并非所有类别都会被用来训练。我们往往希望模型能够准确预测在训练中未出现过的类别。元学习器可以泛化到其他数据集上以实现这一点。

# Few-shot Learning

如果在训练阶段，从训练集中每次抽取C个类别，每个类别K个样本作为support set，从这C个类别中的剩余数据里抽取一批样本做target set，这样的任务称为 C-way K-shot 问题。

在图像领域 Few-shot Learning模型总体上可以分成三类：Mode Based, Metric Based, Optimization Based。 

Mode Based方法通过模型结构的设计，在少量样本上快速更新参数，直接建立输入x和预测p的函数；
Metric Based方法通过度量target和support中样本的距离借助最邻近思想进行分类；
Optimization Baased方法认为梯度下降方法难以在few-shot场景下拟合，因此通过调整优化方法来完成小样本分类任务。

#### Model Based方法

Santoro等人基于LSTM等RNN模型，将数据看成序列来训练，在测试时输入新的类的样本进行分类。

在具体的操作上，在t时刻输入$(\mathbf x_t, y_{t - 1})$，也就是说在当前时刻预测类别，在下一时刻给出真实label。同时添加了external memory外部存储，用于存放上一次的输入。这样反向传播时能够让x和y建立关系，让之后的x能通过外部存储获取相关的图像数据进行对比，实现更好的预测。

#### Metric Based方法

神经网络中有大量的参数，在Few-shot Learning任务中难免的会发生过拟合。

使用非参数的方法（如最近邻、K-近邻、K-means等）由于不需要优化参数，因此可以用它构造一种端到端训练的分类器。这种方法对样本间的距离进行建模，让同类样本相互靠近，异类样本互相远离。

![孪生网络](/img/机器学习/孪生网络.jpeg)

孪生网络(Siamese Network)通过监督训练孪生网络来学习，重用提取的特征进行few-shot学习。

在训练时，将每一个support set送入网络进行训练，在最后通过样本对的距离判断他们是否属于同一类并产生对应的概率分布；预测时，处理测试样本和supportset之间的每一个样本对，最终的结果为support set上概率最高的那一类。

#### Optimization Based方法

梯度优化算法基本都无法在几步内完成优化，其次不同任务的分别随机初始化也会影响收敛。

Optimization Based学习的是模型参数的更新函数或者说更新规则。

拿对梯度下降的参数更新算法来说，采用LSTM来表达元学习器，用其状态来表达目标分类器的参数的更新，最终学会如何在新的分类任务上进行初始化和参数更新。LSTM还能同时考虑一个任务的短期知识和跨越多个任务的长期知识。

![Optimization Based](/img/机器学习/Optimization_Based.jpeg)

