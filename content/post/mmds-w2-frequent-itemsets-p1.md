---
author: hzmangel
date: "2015-06-26T19:35:21+08:00"

categories:
- Happy coding

tags:
- Programming
- mmds
- StudyLog

title: |
  MMDS Notes: W2 - Frequent Itemsets - Part I
---

第二周的最后一块内容是 *Frequent Itemsets* 。主要介绍了 *Frequent Itemsets* ， *Association Rule* 以及算法。这一部分介绍前面的，后面一篇会介绍算法和优化。

<!--more-->

## *Market-Basket* 模型

这个模型一般用来描述两种数据之间的 *m2m* 关系。其中的一个数据是 *item* ，而另一个是 *baskets* 。每一个 *basket* 中包含有多个 *items* ，即为 *itemset* 。 通常认为：

* *basket* 中的 *items* 数量较少，远小于整体 *items* 的数量。
* *basket* 的数量较大，不能完全的放于内存中。

这是一个模型的例子：

* *Market* ：一个超级市场。
* *Item* ：市场中售卖的所有东西。
* *Basket* ：用户的订单。

### *Frequent Itemsets*

*Frequent Itemsets* 即指在 *baskets* 中经常出现的 *itemset* ，它的定义为：

> 假设 *I* 是一个 *itemset* ，定义 *I* 的 *support* 为包含 *I* 的 *basket* 个数。定义数字 *s* 为 *support threshold* ，所有 *support* 数超过 *s* 的 *I* 即为 *frequent itemset* 。

**注意** ：一个 *basket* 可能包含有多个 *itemset* ，并不是说一个 *basket* 中包含的就是一个 *itemset* 。

一个 *itemset* 可以包含有不定个元素。如果一个 *itemset* 是 *frequent itemset* ，那么它的的任何子集都是 *frequent itemset* 。


#### 应用场景

##### 超市商品

* *item* ：超市商品。
* *basket* ：用户订单，上面包含有一个或多个商品。

通过分析订单中的 *frequent itemsets* ，可以得出哪些商品会经常被同时购买。

##### 相关内容查找

* *item* ：单词
* *basket* ：文档，如网页，blog，tweets等。包含一系列的单词。

找出的 *frequent itemset* 中，肯定包含有一些 common word，除去这些词后，可以揭示出一些单词之间的关系。

##### 剽窃检测

* *itme* ：文档
* *basket* ：句子。如果一个文档包含这个句子，那么它在这个 *basket* 中。

找出的 *frequent itemset* 表示这些文档中有大量的句子相似，可以检测抄袭的方法。

##### 生物标记

* *item* ：生物标记，如基因，血蛋白，疾病等。
* *basket* ：病人数据，如基因检测，生化指标等。

找到的包含疾病和生物标记的 *frequent itemset* 可以用于疾病检测。


### *Association Rule*

定义 `$I \rightarrow j$` ，表示如果一个 *basket* 包含有 `{I}` 中所有的元素，那么它就有可能包含 `j` 。即 `{I}` 是一组元素，而 `j` 是一个元素。这个推断，就是一个 *association rule* 。 *association rule* 有一个属性为 *Confidence* ，它表示给了 `{I}` 后出现 `j` 的概率，即在所有包含`{I}`的 *basket* 中同时包含`j`的概率。

现在问题来了：找到所有 *support >= s* 且 *confidence >= c* 的 *association rule* 。此处的 *support* 是指 `{I}` 的 *support* ，而不是 `{I, j}` 的。

计算过程的描述如下：

1. 找到所有 *support >= sc* 的集合
1. 找到所有 *support >= s* 的集合
1. 如果 `{I, j}` 的 *support >= cs* ，那么找到它少一个元素，而且 *support >= s* 的子集。
1. 当且仅当下面条件都满足时， `$I \rightarrow j$` 才是一个被接受的 *rule* 。
  * `{I}` 的 *support s1 >= s*
  * `{I, j}` 的 *support s2 >= sc*
  * *s2/s1 >= c* （此比值即为 *confidence* 的定义）.

在实现上，数据通常是以普通文件方式存储于磁盘上的，存储的方式是按 *basket* 排列。在读入数据时，即将数据切分成不同长度的集合。不难看出，磁盘IO是数据读取过程中主要的开销。在实践中，数据是多次读取的。所以在计算时，读取次数也是一个需要考虑的部分。

在计算过程中，由于设置的阈值比较高，所以通常能找到的就是 *frequent pair* ，所以这也是需要面对的最大问题。

#### 一般算法

一般的算法就是读一次文件，在内存中计算出所有 *pair* 出现的次数。对于一个有 *n* 个元素的 *basket* ，最后会生成 *n(n-1)/2* 个 *pair* 。所以如果 *n**2* 的大小超过内存，这个算法就会失败。

在计算上，有两种方式：

* 计算所有数据对出现的次数，并存放于三角矩阵中。
* 使用类似稀疏矩阵的存储方法，使用坐标+次数的方式存储。

第一种方式每个数据对需要 `4` 个字节（假设一个整型数使用4个字节）。第二种方式每对需要12字节（坐标及计数），但是只有存在的数据对才会占用空间。
