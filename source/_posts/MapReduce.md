---
title: MapReduce
date: 2024-09-30 15:42:33
tags:
category_bar: true
categories: Realtime Big Data Analyic
---

首先对MapReduce进行概述式介绍，接着介绍MapReduce执行过程中的数据流，最后讲解MapReduce的系统架构。

<!-- more -->

# MapReduce

## 1. 概述

MapReduce是一个用来对数据进行批处理的框架，**可扩展性**和**可靠性**高，能够提供**差错容忍**的空间。在MapReduce框架下，输入数据并不一定要按照某种数据模型，因此该框架能处理无模式的数据。MapReduce框架将数据分为若干数据块，能够并行处理和运算，从每个数据块得到的运算结果经过汇总得到最终结果。

传统的数据处理需要将数据迁移至处理数据的结点上，然而MapReduce则是将**数据处理程序迁移至储存数据的结点**上，这样避免了大批量数据的移动，同时也能够**节省带宽**、**减少处理大数据的时间**。

一个MapReduce作业（Job）是一系列需要完成的工作，Hadoop将一项作业分为两个任务（task）：**map**和**reduce**，然后把MapReduce作业所需要的输入数据分块，并且为每一块创建一个map任务，对这一块中的每一条记录执行map函数。


