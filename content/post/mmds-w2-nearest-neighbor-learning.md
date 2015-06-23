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

Locality-Sensitive Hashing，LSH，局部敏感hash或叫位置敏感hash。它的想法是在对原始数据空间的数据做Hash后，让位置相邻的数据有很大概率被放到同一个或者相近的bucket中，而不相邻的点放在一起的概率要很小。这样就会减少后期数据处理的数据集，从而简化后续的工作。

<!--more-->

## 相似数据集

许多数据挖掘的问题都能简化为查找相似数据集的问题，如：

* 查找含有相似单词的页面，用以给页面分类，或者找出页面的镜像站，或者检查剽窃等。
* NetFlix，理解成豆瓣就行，哪些用户有相似的爱好。
* 以及，哪些电影有相似的粉丝。
* 网上找到的个人信息，怎么才能确定哪些属于同一个人。

先从相似文档找起，有三个关键技术：

* Shingling：把文档，像页面啊，邮件啊，什么的，拆成sets。
* Minhashing：在保留相似性的基础上，把大的集合转化成短的标记。
* Locality-sensitive hashing：找出相似的对。

文档相似性可以使用 *Jaccard Similarity* 来衡量。对于两个集合 `$S$` 和 `$T$` 来说，它们之间的 *Jaccard Similarity* 为 `$|S \cap T| / |S \cup T|$`，记为 *SIM(S,T)* 或 *J(S,T)* 。不难看出，当此值为`0`时表示两个集合没有交集，而为`1`时则表示两个集合相等。

### k-shingling或叫k-gram

> *k-shingling* 是指把文档按连续的 *k* 个字母拆成子集的方法。

例如，给定文档`$D$`的内容为`abcdabd`，在`$k=2$`的情况下，获得的 *2-shingling* 集合为 `{ab, bc, cd, da, bd}`。shingling有一个变种是生成一个bag而非set，此时重复的元素不会被归并，而是按照其本来出现的次数出现在最后结果中，如本例中的`ab`将会出现2次。

对于空格的处理有多种选项，较常见的是把所有空格类的东西都替换成一个空格，然后将其作为一个正常元素参与到shingling中去。即shingling后的元素可能会包含2个或多个单词。

为了避免虚假的相似度， `$k$` 的取值需要足够大。一般而言，对于短的如电子邮件之类的文件，取`$k=5$`，而对于长的文档，如研究报告这种，取`$k=9$`会比较好。

对于shingling中的元素，可以直接使用字符串，但是更好的办法是把它通过hash变化映射到某个bucket中，而将这个bucket的编号作为shingling元素进行比较。这样可以在shingling元素空间不变的情况下，降低运行时占用的内存。而且在比较上，整数比字符串要更有优势。这一步叫做 **Compressing Shinglings** 。


### Minhashing

这一段是从 [wikipedia](https://en.wikipedia.org/wiki/MinHash) 上看到的定义：

设有一个hash函数`$h$`，可以将集合中的元素映射为不重复的整数值。这样对于任何集合`$S$`，都能找到一个元素`$x$`让`$h(S)$`取到最小值`$h_{min}(S)$`。这样就把对字符串比较，存储转换成了对整数的计算和存储。由于`$h_{min}(S)$`只能得到一个值，所以需要使用 *Hash Function Family* 去处理集合，以得到一个最小值的向量。在向量长度足够的情况下，两个集合的相似度等于最小值相等的概率。计算向量有两种办法，一种是选取足够多的hash函数，另一种是对一个hash得出的值作多次变换。

在课程中，首先介绍了怎么抽取多个集合的 *Characteristic Matrix* ，这个矩阵的每一列都是一个需要计算相似度的集合，记为`$S_{1}$`到`$S_{N}$`，而行是所有元素的集合，记为`$e_{1}$`到`$e_{M}$`。如果某个元素`$e_{i}$`包含于集合`$S_{j}$`中，则在矩阵相应的位置`$(i,j)$`标上`1`，反之则为`0`。典型情况下这个矩阵是稀疏的。

此后直接介绍了一个 *Minhashing* 的函数簇。假设前述的 *Characteristic Matrix* 的行排列是随机的，我们定义一个 *Minhashing* 函数 `h(S)` ，它的值是在特定排列下，列 `S` 中第一次出现 `1` 的行数。使用多个独立的哈希函数（如100个），即可为每一个集合创建一个 *signatures* ，而多个集合的结果合并后可以生成一个新的矩阵， *signatures matrix* 。这个矩阵的列是各个集合，而行是某一次计算 *Minhashing* 时的结果。


下面来分析下 *Jaccard Similarity*。首先看 *Characteristic Matrix* 。设有两个需要比较的集合 `$S_{1}$` 和 `$S_{2}$` ，假设它们的 *Characteristic Matrix* 为 `$M$`，那么在矩阵 `$M$` 中，每一行的元素只有4种组合： `(0,0)` ，`(0,1)` ， `(1,0)` 和 `(1,1)`。我们把这4种关系在M中的数量分别记为ABCD，不难看出，两个集合的相似度可以表示为 `$J(S_{1}, S_{2}) = D/(A+B+C)$`。

然后再来看 *signatures matrix* 。在某个特定的排列下，如果两个集合的 *Minhashing* 值相同，那第它们一定是 `(1,1)` 形式的，而其它三种形式不会有此结果）注意，此处只能保证， *Minhashing* 值相同时，能保证这一行是 `(1,1)`，但是一行是`(1,1)`并不能说明这一行是 *Minhashing* 值）。所以可以得知，两个集合 *Minhashing* 值相等的概率，也就是两个集合的 *Jaccard* 相似度，都是 `$J(S_{1}, S_{2}) = D/(A+B+C)$` 。

在实际实现上，给大量的数据做随机排列是比较难以实现的，所以更加通用的办法就是如wiki上说的，挑选多个 hash 函数来处理，下面是一段伪代码，计算某集合的 *Minhashing* 向量：

```
FOREACH hash_func_family:
  CALCULATE hi(r)

FOREACH columes:
  IF val(c) == 1:
    # Init value for SIG(i, c) is inf
    SIG(i, c) = min( SIG(i, c), hi(r) )
```

### Locality-Sensitive Hash

*By Me: 此处的概念还有些模糊，需要再啃啃。*

经过了前面的 *Shingling* 和 *Minhashing* ，需要处理的数据已经减少许多了，但是对于大文档集来说还不够。如果是需要找到任意两个集合之间的相似度，那么除了计算它们每两对之间的相似度以外没有其它任何办法。但是如果只是需要找到超过某个相似度阈值的集合对，则可以使用LSH，又叫 *Nearest Neighbor search* 。

LSH的一般做法是对元素使用多次Hash，让相似的元素落入同一个bucket中（即Hash冲突），而不相似的不在。对于上面生成的 *signatures matrix* ，一个有效的办法是把矩阵按`r`行分成`b`个brand，对每一个brand中的每一小块长度为`r`的特征值做hash，下面是分析（这块还是有些地方没想清楚，先记录下来）：

* 设矩阵分成了`b`个 *brand* ，每个 *brand* 中有 `r` 行。某特定两个文档的 *Jaccard Similarity* 是 `s` 。即在 matrix 中某个Minhashing字符串与其它有`s`的概率相似。
* 某个brand中选定的特征列和其它所有列相似的概率是 `$s^{r}$`。
* 某个brand中选定的特征列和至少一个其它列不相似的概率是`$1-s^{r}$`。
* 一个特征列和每一个brand中都有至少一个不相似列的概率是`$(1-s^{r})^{b}$`。
* 一个特征列和至少一个brand中所有列都相似，从而成为 *candidate pair* 的概率为 `$1-(1-s^{r})^{b}$`。

这个曲线是一个S型的连续曲线，我们需要做的就是通过挑选`b`和`r`，让这条曲线在两端尽量的平缓，而在中间部分尽可能的陡峭。这样就不会有过多的 *False Positive* 或者 *False Negative* 。


## 具体使用流程

综上所述，在实际应用中会有下面几步工作（文档比较）：

1. 选择整数 `$k$` ，将输入文档转换为 *k-shingling* 集合。此处可以通过Hash将 *k-shingles* 转换为较短的 *bucket序号* 。
1. 以 *shingle* 排序 `<document, shingle>` 对。
1. 选择长度 `$n$` 用于 *Minhashing Signature* ，并为所有文档计算特征值。
1. 确定一个概率 `$t$` 作为文档相似度的阈值，选择 `$b$` 和 `$r$` 并保证 `$br=n$` ，而且阈值`$t$`接近`$(1/b)^{1/r}$`
    * 如果需要最大程度的避免 *False Negative* ，那么选择 `$b$` 和 `$r$` 时要注意计算出来的值要小于 `$t$`。
    * 如果需要保证速度而且避免 *False Positive* ，那么选择 `$b$` 和 `$r$` 时注意计算出一个高的阈值。
1. 使用 *LSH* 找到所有的 *candidate pairs*。
1. 检查选择出来的特征对，确定它们的相似度都大于 `$t$` 。


## 实例

*注：此处只是列出课程中出现的示例，后续会尝试使用程序完成，再补齐说明。*

### Entity Resolution

### Fingerprints

### Similar News Articles


