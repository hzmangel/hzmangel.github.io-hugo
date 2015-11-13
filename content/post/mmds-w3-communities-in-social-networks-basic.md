---
author: hzmangel
date: "2015-11-11T23:53:33+08:00"

categories:
- Happy coding

tags:
- Programming
- mmds
- StudyLog

title: |
  MMDS Notes: W3 - Communities in Social Network (Basic)
---

第三周分两部分，第一部分是 *Communities in Social Network* 。是介绍如何在社交网络中给用户分组的。这一部分的课也分为基础和高级，这一篇是基础， 高级的课程另开一篇吧（主要是基础中还有些东西没完全弄明白...）。

<!--more-->

社交网络包含有用户和用户之间的联系。把用户看成顶点，把用户之间的联系看成边，就可以得到一个图 (*social graph*)。像 Facebook 中的图就是无向图，而 Twitter/G+ 中的就是有向图。在 *Network* 和 *Communities* 这块，主要的任务有两类：

* 给定一个模型，怎么去生成网络
* 给定一个网络，怎么找到最好的 *Communities* 模型

## 从模型生成网络

从模型生成网络是指定义一个模型，它接受一系列的参数，并最终生成一张网络（主要是边的生成）。

### AGM: Affiliation Graph Model

#### 参数

* `$V$`: 节点
* `$C$`: *Community*
* `$M$`: 节点与 *Community* 之间的关系
* `$P_{c}$`: 某个 *Community* 的联通率，即都属于某个 *Community* 的节点之间联通的概率。


#### 生成过程

遍历每个 *Community* `$A$` 中的节点对，以 `$P_{A}$` 的概率连接它们（边的生成）。

任意两点间的连接概率为`$P(u,v)=1-\prod_{c \in M_{u} \bigcap M_{v}}(1-p_{c}) $` 。其中，`$M_{u}$` 表示包含有节点 `$u$` 的 *Community* 集合。如果 `$u$` 和 `$v$` 没有共用的 *Community* ，则概率 `$P(u,v) = \epsilon $` 。


#### 适应性强

可以用于生成多种 *Community* ：

* 无交集 (Non-overlapping)
* 有交集 (Overlapping)
* 内嵌 (Nested)


## 从网络生成模型

### AGM: Affiliation Graph Model

这个是mmds书本[配套幻灯片](http://www.mmds.org/mmds/v2.1/ch10-graphs2.pdf)上的内容，占坑。


### BigCLAM

#### Membership Strength

引入一个新概念，某个节点 `$u$` 对某个 *Community* 的 *membership strength* ，记为 `$F_{u,A}$`，同时定义如果此值为0，则表明节点不在 *Community* 中。所以在某个 *Community* 中，两个节点的联通概率为 `$P_{A}(u,v)=1-\exp (-F_{uA} \cdot F_{vA})$` 。

> 此处不是很明白为什么同 *Community* 中节点的连通概率可以写成上面的方式，需要去书中相应章节找答案。

上面介绍的是在同一个 *Community* 中两个节点的联通概率，现在将要说的是在不同的 *Community* 中如何确定联通概率。首先需要定义一个 *Community membership strength matrix* ，它的列是 *Community* ，行是节点。将矩阵记为 `$F$`，某个位置 `$F_{vA}$` 的值表示的是节点在这个 *Community* 中的 *strength* ，所以任意一行 `$F_{u}$` 则表示的是节点在多个 *Community* 中的 *membership strength* 向量。

上面的公式计算的是两个节点在单 *Community* 中的联通概率，两个节点之间存在至少相同的 *Community* ，那么它们的联通概率即为 `$P(u,v)=1-\prod_{C}(1-P_{C}(u,v))$` ，经过和上面的公式整合，简化，可以得到 `$P(u,v)=1-\exp (-F_{v} \cdot F_{v}^{T} )$`

所以问题就从如何从一个网络生成 *Community* 变成了如何从给定的网络中找到可以最大化 *likelihood* 的矩阵 `$F$` ，即 `$argmax_{F} \prod_{u,v \in E} p(u,v) \prod_{(u,v \notin E)}(1-p(u,v))$` 。通常这个 *likelihood* 会使用对数表示，记为 `$l(F)=\log P(G|F)$` 。最后问题就演变为找到可以使 `$l(F)$` 最大化的 `$F$`。而 `$$l(F) = \sum_{(u,v) \in E} \log (1-\exp (-F_{u}F_{v}^{T})) - \sum_{(u,v) \notin E}(F_{u}F_{v}^{T})$$`

#### 计算

考虑到需要求 *likelihood* 的极大值，所以考虑使用导数（ *gradient* ）来计算。将上式中的`$F_{u}$`看作变量，即某节点 `$u$` 的最大 *likelihood* 值，则对上面公式的求导后可变为

`$$ \nabla l(F_{u})=\sum_{v \in \mathcal{N}(u)} F_{v} \frac{\exp (-F_{u}F_{v}^{T})}{1- \exp (-F_{u}F_{v}^{T})} - \sum_{v \notin \mathcal{N}(u)} F_{v} $$`

其中，`$\mathcal{N}(u)$`表示节点 `$u$` 的邻接节点。


这个公式的问题在于，后面的一个求和操作是线性时间的，而且是和所有节点数目相关，这样会拖慢运算速度。式子的后一项是计算所有不在节点 `$u$` 邻接节点中的节点的`$F$`值的和，它可以替换为 `$\sum_{v} F_{v} - F_{u} - \sum_{v \in \mathcal{N}(u) F_{v}}$` ，而这个式子的第一项 `$\sum_{v} F_{v}$` 是可以预先计算得到的，而 `$\sum_{v \in \mathcal{N}(u) F_{v}}$` 虽然也是线性时间，但是只和 `$u$` 的邻接节点数目想关，这个数值在真实网络中远小于整个网络的节点数的。所以在速度上有很大的改进。

> 有点不明白，`$\sum_{v} F_{v}$` 在开始的时候是怎么可以事先计算的，要求的不就是某个 `$F$` 吗。后期实现的时候再看如何处理吧。

------

## 其它阅读: Graph and Social Network

**注**：此处的内容并没有在课程讲义中，而对应书本的 *10.1.2 Social Networks as Graphs* 节。主要是觉得在课程上说的有些东西不是很清楚来龙去脉，所以去书本上找一找，就看到了这个东西。目前还有一些问题不是很明了，也一起列在最后。

一般认为，一个 *SN* 可以看成一张图，但这并不是表示任意一个 *graph* ，就能表示一个 *social network* ，这个是由这张图的 *locality of relationships* 确定的。

检查一个 *graph* 是不是 *SN* ，则需要计算这张图的 *locality of relationships* （找不到这个词怎么翻译，先原样放这吧）。给定一个有 `$N$` 节点， `$E$` 条边的图，它的联通率是指，有三个点`$X$`，`$Y$`和`$Z$`，在确定`$(X,Y)$`和`$(X,Z)$`联通的情况下，`$(Y,Z)$`联通的概率。

此概率的计算有以下几步：

* **理论联通率**：完全图会有`K=$\binom{N}{2}$`条边，所以理论联通率应为 `$E/K$`。
* **单条边的联通率**：在大规模的图中，一般认为理论联通率就是图的联通率，但是在小规模的图中，这个数值偏差有些大，所以需要重新计算。考虑给定条件，在已经确定有两条边的情况下，计算第三条边出现的条件概率，即为`$(E-2)/(K-2)$`
* **实际联通率**：实际概率的计算需要遍历图中的每个节点，假设为`$X$`，再找到邻接节点 `$Y$` 和 `$Z$` ，最后计算 `$(Y,Z)$` 出现的概率。

书中表示，最后算出来的实际概率大于单边理论概率，所以这张图可以看成是某 *SN* 网络（可以表示 *SN* 网络的 *locality* ）。

> 1. 是不是说大规格图的联通率直接认为是理论联通率，所以就肯定可以表示 *SN* 而不用再计算？
> 1. 如果计算出来的联通率比单边联通率还要低，那是不是就说明这个图不能表示 *SN* ？
> 1. 是不是说实际联通率只要超过单边联通率就可以看成是 *SN* ？还是说需要超过某个阈值才可以算？




