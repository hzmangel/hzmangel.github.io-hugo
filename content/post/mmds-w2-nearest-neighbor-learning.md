---
author: hzmangel
date: "2015-06-24T02:20:33+08:00"

categories:
- Happy coding

tags:
- Programming
- mmds
- StudyLog

title: |
  MMDS Notes: W2 - Nearest Neighbor Learning
---

第二周的 *Nearest Neighbor Learning* 只是一个大概的介绍。这是一个通过在训练集中找到离待查询数据最近的点从而做出预测的方法。

<!--more-->

## Supervised Learning

*Supervised learning* 是机器学习中的一种方法（还有两种分别是 [Unsupervised learning](https://en.wikipedia.org/wiki/Unsupervised_learning) 和 [Reinforcement learning](https://en.wikipedia.org/wiki/Reinforcement_learning) , via [wikipedia](https://en.wikipedia.org/wiki/Machine_learning) 。这个如果将来看了相应的课再写。）,它通过训练数据集建立一种模式或叫函数，再通过此模式去匹配待查询数据集从而得出预测结果。

## Instance Based Learning

*Instance Based Learning* 是机器学习的一种方法，也叫 *Memory Based Learning* 。它不生成完整的匹配模式，而是在从训练数据中找到和待查询数据相近的数据（这些数据通常存放于内存中），并输出结果。将要说到的 *Nearest Neighbor Learning* 即是其中的一种。此方法也分为 *1-Nearest Neighbor* 和 *k-Nearest Neighbor* 。进行计算需要了解如下4个元素：

* 如何衡量距离？
* 找几个 Neighbor ？
* 各点是否有不同的权重？
* 如何确定输出？

### 1-Nearest Neighbor 和 k-Nearest Neighbor

在 *1-Nearest Neighbor* 的情况下，距离使用欧氏距离，寻找最近的`1`个 *Neighbor* ，各点权重相同，直接以最近 *Neighbor* 的输出为查询数据的输出。而在 *k-Nearest Neighbor* 中，查找的节点变成了 *k* 个，而输出出变成了 *k* 个节点的平均值。


### Kernel Regression

核回归，统计中的一种方法，具体的算法及使用场景没看太明白，此处暂时仅为记录而用。对于这个算法，它所涉及到的4个参量是：

* 欧氏距离
* 查找所有训练集中的点
* 权重函数： `$w_{i} = exp(-\frac{d(x_{i}, q)^{2}}{K_{w}})$`
* 输出：`$\frac{\sum_{i}w_{i}y_{i}}{\sum_{i}w_{i}}$`
