---
author: hzmangel
date: "2015-06-13T23:13:32+08:00"

categories:
- Happy coding

tags:
- Programming
- mmds
- StudyLog

title: |
  MMDS Notes: W1 - HDFS & MR
---

前段时间在Cousera上各种挤时间跟完了一门 [MMDS](https://class.coursera.org/mmds-002) ，手上留下了一堆笔记，整理下，顺便给新blog开光吧。

课程总共7周，这篇整理的第一周的 `HDFS` 和 `MR` 部分。

<!--more-->

## DFS

课程开始是从分布式存储DFS讲起，介绍性的东西居多。在大规模的集群中，硬件故障是个很容易发生的问题。解决方案要么就是堆大把的钱上NB的机器，要么就是多做备份。这里的DFS就是后一种解决方案。

现在较为通用的分布式架构就是使用不那么NB的Linux机器组建Cluster，然后用不那么NB的网络把它们连接起来。而在分布式存储的情况下，把数据在不同节点之间复制是需要时间的，所以让程序去找数据就是一个比较正确的解决方案了。

DFS中的节点有两类，`Chunk Server`用来存放数据，`Master Node`用来存放文件和`Chunk Server`的对应关系。一个文件会被分成多处连续的`chunks`，典型的大小是16~64MB。每一个`chunk`都会复制2到3份，分别存储在不同的机器上（最好是在不同rack上）。而`Master Node`就会存储这些机器和文件的映射关系。当需要查找这个文件时，先从`Master Node`处取到`Chunk Server`的信息，再从相应的server上获取文件内容。


## Map Reduce

MR是一种编程模型，典型的应用场景就是 **Word Cound** 。将MR应用到 WordCount 的先决条件为：

* 文件过大，不能完整的放到内存中去
* 但是所有的`<key, value>`对可以完全放到内存中

### 算法流程

1. 将数据分块读入
1. `Map` ：创建`<k, v>`数据对。在本例中，即创建出一系列的`<word, 1>`对。
1. `Group by Key` ：对`<k, v>`进行排序。本例中即将同样key的`<key, value>`对放在一起。
1. `Reduce` ：将同一个`key`的`<k, v>`对合并到一起，并对`v`做相应处理，在本例中，就是把所有的值相加。
1. 输出结果

在具体实现上，程序需要指定`Map`和`Reduce`两个方法，这两个方法的参数都是`<key, value>`对。例如`Map`函数的`key`可以是文件名，`value`是对应的某一行或者某一块文字。而在`Reduce`函数中，输入的`key`是某个单词，而`value`是1。`Reduce`负责归并结果，并输出。

### Map Reduce的环境

除了根据逻辑实现`Map`和`Reduce`两个函数外，还需要一个运行Map Reduce的环境，它需要提供下面几项功能：

* 把输入数据分块
* 在机器前调度程序运行
* 处理 **Group by key**
* 处理机器故障
* 管理机器间通信

### 出错处理

MR出错分几种，处理方式也不同：

* `Map` Worker出错：MasterNode会将此块数据标为未完成，并等待下一轮调度
* `Reduce` Worker出错：MasterNode会将正在运行中的任务置为无效，并等待下一轮调度
* MasterNode：任务直接退出。

### `Map`和`Reduce`的数目

`Mapper`的数目会比节点数目多许多，这样就能保证一台节点上会被会到多个任务，即使节点出错，也只会影响到正在运行中的小块任务，对于其它被分配到此节点但是还没有运行的任务来说，可以很方便的调度到其它节点上。如果任务很大，而又有一个节点在任务运行时故障的话，就需要回滚较多的部分。而且大任务也不方便充分利用空闲的机器资源。

`Reduce`的数目没有具体要求，但是一般会比`Mapper`的数目要少。


### 改良

还是拿WC作为例子。在`Reduce`函数处，原始的办法需要传一堆`<word, 1>`对到处理端，这会占用较多的带宽和传输时间，所以一个改良就是在传给`Reduce`之前先归并一下，相当于多级`Reduce`，而这一中间处理是在本地完成的，这样就可以减少对网络带宽的占用，以及乱序Mapper结果时的计算量。

注意，此种方式并不适用所有计算，例如对多个数取平均值就不可以使用。


