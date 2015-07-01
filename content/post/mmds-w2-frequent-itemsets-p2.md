---
author: hzmangel
date: "2015-07-01T18:20:21+08:00"

categories:
- Happy coding

tags:
- Programming
- mmds
- StudyLog

title: |
  MMDS Notes: W2 - Frequent Itemsets - Part II
---

这一部分介绍 *A-Priori* 算法。

<!--more-->

前一篇中所说的两种计算数据对次数的办法，都是 *one-pass* 的，程序在读入数据的同时，生成数据对，然后在数据对对应的计数上加1。这样数据读取完成后，直接查找计数的数即可得知结果。但是这个方法在在数据量过大时会超过主存的大小，从而在计算时引发换页，降低效率。而本篇要介绍的 *A-Priori* 算法，通过减少需要计算的数据对个数，从而减少对内存的需求。

这个算法的核心思想是 *monotonicity* ，即如果 `{I}` 是 *frequent itemset* ，那么它的所有子集都是 *frequent itemset* 。反过来说，如果一个元素不是 *frequent itemset* ，那么任何包含它的集合都不可能是。

## 基本算法

*A-Priori* 是一个 *two-pass* 算法，

### First Pass

在读取的过程中创建两张Hash表，一张表是将所有 `item` 映射为 `1` 到 `n` 的索引，减少后面引用时占用的内存。第二张表是将 `item` 的索引与 `item` 出现的次数对应起来。

### Between Passes

检查索引和出现次数映射的表，找出其中所有的 *frequent itemset* ，并将它们重新索引为 `1` 到 `m`

### Second Pass

计算所有使用 *frequent itemset* 表中元素组成的数据对的 *support* 值。具体流程如下：

1. 对每一个 *basket* ，找出其中属于 *frequent itemset* 表中的元素。
1. 为找到的元素构建数据对。
1. 找到数据对对应的计数值，加上它出现的次数。

最后，只需要检查第二次生成的数据就好。另外，如果需要求多于2个元素的 *itemset* ，可以按上述办法将算法级联计算。

-----

基本算法介绍完成，下面就是改进了。

> Mutation: it is the key to our evolution.


## PCY

在 *first pass* 的时候，内存中有很多空闲的地方，所以在给 *item* 计数的同时，生成出所有的 *pair* ，取 Hash 后，给对应的 *bucket* 加1 。在 *second pass* 的时候，只处理如下的数据对：

1. `i` 和 `j` 都是 *frequent items*
1. `{i, j}` 被 Hash 的 *bucket* 是 *frequent bucket*

这样 *second pass* 中需要处理的数据又能少一些了。

这个算法的原理是，如果一个 *pair* 是 *frequent pair* ，那么它对应的 *bucket* 一定超过 *support* 的阈值。而如果一个 *bucket* 没有超过阈值，那么里面的所有 *pair* 一定不是 *frequent pair* 。 注意：一个 *bucket* 是 *frequent bucket* 并不能保证其中的 *pair* 都是 *frequent pair* 。

### First Pass

```
for b in buckets
  for item in b:
    count[item]++
  end

  for pair in b:
    bucket(hash(pair))++
  end
end
```

### Between Passes

除了找出 *frequent item* 外，还会把上一步中的 *hash bucket* 替换为 *bitmap* ，`1` 表示 *frequent bucket* ， `0` 表示不是。这样可以进一步减少内存空间的占用（从 `32bit` 降到 `1bit`）。

### Second Pass

根据上面列出的条件，找到 *canidate pair* ，计算。


PCY算法还有两个变种：

## Multistage

这个算法的核心在于，在PCY算法的 *first pass* 后，并不开始对 *bucket* 做筛选，而是将其中属于 *frequent bucket* 的 *pair* 挑出来，再做一次 Hash ，从而进一步减少 *second pass* 中需要处理的数据量。从定义上来看，这个算法可能会有多个 *pass*


## Multihash

在 *first pass* 中，使用多个独立的Hash函数来计算 *bucket* 。相比物 Multistage ，它还是 *two pass* 的算法，但是它的风险在于在多个 Hash 的作用下，可能会有更多的 *frequent bucket* 。如果它能保证 *frequent bucket* 的数目足够小，那么我们就能在 *two pass* 获得 Multistage 的优点。

----

算法的另一个发展趋势是获取到大部分的 *frequent itemsets* ，使用降低精确性的办法来换取效率。

## 基本想法

对样本 *basket* 进行随机抽样，并在内存中对样本进行 *A-Priori* 或其改进版本的计算，此处需要计算所有的 *frequent itemsets* ，而不仅仅是 *pair* 。计算时使用的 *support threshold* 需要根据样本容量处理，例如抽样率为 `1/100` ，那么阈值也相应的变成 `s/100` 。在 *second pass* 的时候，可以使用整体数据来验证是否找到的真的是 *frequent pair* 。

这个算法有个问题有些 *set* 可能在样本中不是 *frequent* 但在全局中是，这里有一个稍微改进的办法是适当降低样本计算中的 *support* 阈值，例如从 `s/100` 降为 `s/125` ，不过这会需要更大的内存空间。（而且还是有可能会漏掉的吧......）


## SON 算法

这个算法是将大的数据分成小集合分别读入内存进行计算，并将在任意一次计算中得到的 *frequent pair* 置于候选的位置。而在 *second pass* ，计算这些候选 *pair* 并得出最后结果。这个算法的核心是，任何一个 *pair* 如果是 *frequent pair* ，那么它一定会在某个子集中是 *frequent pair* 。

这个算法可以让计算并行化，以提高处理效率。

## Toivonen 算法

在 *first pass* 中，使用子集基本算法一样的方法，抽样并计算，但是在取阈值的时候设置的稍低，例如 `s/125`，以尽可能的不要遗漏在全集中是 *frequent set* 的数据。此后设置一个 *negative border* 区间，以存放所有子集是 *frequent set* 但是本身不是的集合。在 *second pass* 的时候，计算所有候选 *frequent itemsets* 和 *negative border* 中的数据。

* 如果 *negative border* 中没有数据是 *frequent itemsets* ，那么所得就是所需的 *frequent itemsets* .
* 如果在 *negative border* 中找到了 *frequent itesmsets* ，那么就需要重新取样进行计算，直到满足前一条的条件为止。

证明：假设 `S` 是一个在全集中的 *frequent itemset* ，但是在抽样样本中，它既不属于 *frequent itemsets* 也不是 *negative border* 。那么可以找到一个 `T` ，它是在样本中非 *frequent* 的 `S` 最小子集。即比 `T` 小的子集在样本中都是 *frequent itemset* ，那根据 *negative border* 的定义，`T` 一定会在 *negative border* 中，否则它就不是是最小子集。由于空集总是 *frequent itemset* ，所以 `T` 一定存在。这样就有一个 `T` ，它在全集中是 *frequent itemset* ，在样本中是 *negative border* ，于是，重新抽样吧。
