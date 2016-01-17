---
author: hzmangel
date: "2016-01-17T07:22:32+08:00"

categories:
- Happy coding

tags:
- Programming
- Hadoop
- StudyLog

title: |
  Hadoop Study Log 1: Env Setup
---

最近想折腾数据，于是决定从基础的Hadoop开始。

<!--more-->

## 基本概念

这里只是一些我之前可能弄混淆的概念，其它的一些东西可能需要去hadoop官网看了。

* Hadoop是一个编程框架或者叫库，主要用于下面类型程序的编写
  * 大规模数据集
  * 分布式
  * 简单任务模型
* Hadoop的核心项目[包含下面几个](https://hadoop.apache.org/)：
  * Common: 一些通用组件，工具类
  * HDFS：分布式文件系统
  * YARN：任务调度及资源管理
  * MapReduce：并行编程模型
* 还有一些衍生项目，具体[参见网页](https://hadoop.apache.org/)

## 环境配置

鉴于之前在osx上装东西碰上过坑，所以选择了virtualbox+ubuntu的方案。系统的版本是Ubuntu 15.05，Hadoop选择的是2.6.3（截止写文章的时候它的Release Date最新）。具体步骤如下：

1. 下载及安装

    ```bash
    # Download package from Hadoop site
    $ wget http://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.6.3/hadoop-2.6.3.tar.gz

    # Decompress it
    $ tar zxvf hadoop-2.6.3.tar.gz

    # Move the package files to system directory, and change owner
    $ sudo mv hadoop-2.6.3 /opt/hadoop
    $ sudo chown -R vagrant:vagrant /opt/hadoop
    ```

1. 配置服务（Single Node）

    ```bash
    # Export JAVA_HOME env variable
    $ export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
    ```

    并根据[此页面](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)中的内容，修改相应文件并配置SSH服务。

      * etc/hadoop/core-site.xml:
      * etc/hadoop/hdfs-site.xml:

1. 格式化namenode

    ```bash
    $ bin/hdfs namenode -format
    ```

1. 设置`etc/hadoop/hadoop-env.sh`中的`JAVA_HOME`变量

1. 开启dfs服务

    ```bash
    $ sbin/start-dfs.sh
    ```

至此，一个运行在单节点环境的Hadoop环境就ok了。
