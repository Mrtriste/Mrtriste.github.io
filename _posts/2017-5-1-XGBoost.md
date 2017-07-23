---
layout:     post
title:      "XGBoost"
subtitle:   "XGBoost"
date:       2017-05-01
author:     "MrTriste"
header-img: "img/post-bg-js-module.jpg"
tags:
    - 机器学习
    - XGBoost
---


参考[xgboost原理](http://blog.csdn.net/a819825294/article/details/51206410)和[xgboost入门与实战](http://blog.csdn.net/sb19931201/article/details/52557382)



## 目标函数

XGBoost目标函数的定义就是：损失函数+正则项+常量。

正则项就是为了控制模型的复杂度，损失函数本来是预测的分类$$\hat{y_i}$$与目标$$y_i$$的差异，但是为了降低计算的复杂度，基于boosting的思想，每次根据上一次的残差更新这次预测值，那么损失函数就是$$(\hat{y_i}+f_t(x_i))$$与$$y_i$$的差，然后用泰勒展开来近似这个损失函数，最终损失函数只依赖于每个数据点在误差函数上的一阶导数和二阶导数。



## 简化目标函数

对于$$f_t(x)$$，我们可以将它细分，用$$f_t(x)=w_{q(x)}$$来表示，其中$$q(x)$$表示样本x所在的叶子索引，$$w_i$$表示叶子i的值是多少，最后可以简化成

$$Obj^{(t)} = \sum_{j=1}^T[G_jw_j+\frac{1}{2}(H_j+\lambda)w_i^2]+\gamma T$$

其中$$G_j$$和$$H_j$$分别是误差函数的一阶导和二阶导，$$T$$是叶子节点数。



##  分裂节点算法

###    1、贪心法

这种算法针对数据量不是那么大，因此遍历到了每个节点，也更精确。

假设有m个叶子节点，现在就其中某个叶子节点分裂的情况而言：

$$Gain = \frac{1}{2}[\frac{G_L^2}{H_L+\lambda}+\frac{G_R^2}{H_R+\lambda}-\frac{(G_L+G_R)^2}{H_L+H_R+\lambda}]-\lambda$$

在节点的某个位置分裂时，会形成左子树和右子树，[]中前两项分别是左右子树的分数，减去原来的分数，又由于分裂后多了一个节点，所以要减去$$1*\lambda$$

这样就可以得到由于在该位置分裂，整个树的分数增加了多少，取最大的即可。

###    2、近似算法

在数据量很大时，数据不能一次性的读入内存，那么就需要使用近似算法来计算，具体思想是根据特征分布的百分比提出候选的点，然后统计数据来找到最好的方案。

这个算法有两个变种，一种是全局的，一种是本地的，区别就是什么时候提出候选点。全局方案是在一开始初始化时就提出所有的候选点，本地的是在每次分裂之后提出候选点。与本地方案比，全局方案需要更少的提出步骤。但是，通常来说，全局方案也需要更多的候选点，因为一旦proposed后，这些候选点就不变了，但是本地方案每次分裂后都会重新提出，所以在有更深深度的树中，local proposal会提出更适合的点。



## XGBoost总结

不考虑工程上的原理，xgboost总结起来就是，我们打算使用CART树进行预测，我们首先得到学习目标函数，包含损失函数和树的复杂度，在每一轮的迭代中，我们要为预测值加上一个$$f_m(x)$$，代入目标函数使其最小（也就是树最优），那么怎么确定这个$$f(x)$$呢？在这之前和GBDT都很像，从这里开始就不一样了，我们这里的处理是用泰勒近似展开，保留到二次项，我们重新定义每棵树和树的复杂度项，之后可以推导出目标函数的值与分裂点左右子树的一二阶导有关，我们就可以根据贪心算法或者近似算法等来找到最优的分割点。



## XGBoost 与GBDT

参考[机器学习算法中GBDT和XGBOOST的区别有哪些？——wepon的回答](https://www.zhihu.com/question/41354392/answer/98658997)

- 传统GBDT以CART作为基分类器，xgboost还支持线性分类器，这个时候xgboost相当于带L1和L2正则化项的逻辑斯蒂回归（分类问题）或者线性回归（回归问题）。
- 传统GBDT在优化时只用到一阶导数信息，xgboost则对代价函数进行了二阶泰勒展开，同时用到了一阶和二阶导数。顺便提一下，xgboost工具支持自定义代价函数，只要函数可一阶和二阶求导。
- xgboost在代价函数里加入了正则项，用于控制模型的复杂度。正则项里包含了树的叶子节点个数、每个叶子节点上输出的score的L2模的平方和。从Bias-variance tradeoff角度来讲，正则项降低了模型variance，使学习出来的模型更加简单，防止过拟合，这也是xgboost优于传统GBDT的一个特性 
- Shrinkage（缩减），相当于学习速率（xgboost中的eta）。xgboost在进行完一次迭代后，会将叶子节点的权重乘上该系数，主要是为了削弱每棵树的影响，让后面有更大的学习空间。实际应用中，一般把eta设置得小一点，然后迭代次数设置得大一点。（补充：传统GBDT的实现也有学习速率）
- 列抽样（column subsampling）。xgboost借鉴了随机森林的做法，支持列抽样，不仅能降低过拟合，还能减少计算，这也是xgboost异于传统gbdt的一个特性。
- 对缺失值的处理。对于特征的值有缺失的样本，xgboost可以自动学习出它的分裂方向。
- xgboost工具支持并行。boosting不是一种串行的结构吗?怎么并行的？注意xgboost的并行不是tree粒度的并行，xgboost也是一次迭代完才能进行下一次迭代的（第t次迭代的代价函数里包含了前面t-1次迭代的预测值）。xgboost的并行是在特征粒度上的。我们知道，决策树的学习最耗时的一个步骤就是对特征的值进行排序（因为要确定最佳分割点），xgboost在训练之前，预先对数据进行了排序，然后保存为block结构，后面的迭代中重复地使用这个结构，大大减小计算量。这个block结构也使得并行成为了可能，在进行节点的分裂时，需要计算每个特征的增益，最终选增益最大的那个特征去做分裂，那么各个特征的增益计算就可以开多线程进行。
- 可并行的近似直方图算法。树节点在进行分裂时，我们需要计算每个特征的每个分割点对应的增益，即用贪心法枚举所有可能的分割点。当数据无法一次载入内存或者在分布式情况下，贪心算法效率就会变得很低，所以xgboost还提出了一种可并行的近似直方图算法，用于高效地生成候选的分割点。