---
author: hzmangel
date: "2015-06-17T08:02:10+08:00"

categories:
- Happy coding

tags:
- Programming
- mmds
- StudyLog

title: |
  MMDS Notes: W2 - Locality-Sensitive Hashing
---

Locality-Sensitive Hashing，LSH，局部敏感hash或叫位置敏感hash。它的想法是在对原始数据空间的数据做Hash后，让位置相邻的数据有很大概率被放到同一个或者相近的bucket中，而不相邻的点放在一起的概率要很小。这样就会减少后期数据处理的数据集，从而简化后续的工作。

<!--more-->



