---
title: 研究方向调研笔记2--GCN 
tags: AlgorithmLearning, WorkLog
renderNumberedHeading: true
grammar_cjkRuby: true
---
看完learning to hash之后,我已经把目前LSH系列和learningtohash系列的算法了解一遍了,当然,在这一过程中我并没有发现什么可以产生idea的点子,我发现learningtohash这个东西目前给我可以灵感的地方在于**使用一个01离散线性变换的结果进行了压缩**,但是这个操作,正如论文里所说的,在optimization的那一步很难进行,必须就行连续化的操作.在sketch的方向上都使用不好了.关于直接使用sketch的那些算法,我觉得改进需要的学术知识有点多,加上我并没有这方面太多的知识,所以仍然是难以实现,.但是,事情并不是毫无起色,比如做完李润东学长讲的,作者拿数据挖掘中的一个词频统计的结构加了四种东西,硬生生地搞出了四篇论文,并且确确实实地找到了他们的应用场景,这确实值得我这种人学习!
今天继续上一次的学习,来看一下图神经网络的这一领域的现状,希望能获得一些启发.

首先是A Comprehensive Survey of GNN 这篇论文的总结.
### 图神经网络,图卷积产生的初衷
1. 原始的深度学习主要是对欧氏空间数据进行的学习,而对于结构复杂的非结构化数据很难处理;
2. 机器学习算法都是基于独立同分布假设进行的,而实质上甚至数据向量之间常常相互关联以至于不满足上述条件

基于以上两点,就有人在CNN和RNN的基础上提出了图神经网络图卷积等诸多概念.
这些概念都是在欧几里得弓箭上发酵的来的,比如下图,描述了通过理解conv在cv上的操作来定义图上的卷积.

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577085546066.png)

在RNN方向,有人借助RNN提出了Recurrent GNN(后面详述),这个模型的想法就是学习某个节点周围节点的信息和该节点的信息进行综合,最终球的一个类似于稳定的不动点的结果.这种算法的缺点就是计算量比较大.

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577085455864.png)

在CNN方向,主要有两种approach,分别是spectral-based 和spatial-based.前者是基于spectral graph theory进行的,后者则是在RGNN上进行的改进.原文称之为:
> ![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577085791799.png)
这些后面都会详述,

### Graph neural network和network embedding的关系
什么是embedding?
embedding是一种方式,试图用一个低维度向量表征网络节点,同时保持图内节点间的拓扑结构和节点的内容信息.通过这种方式就可以把结构复杂的数据转化为规范的欧式数据之后调用其他经典算法执行相应任务.

而根据我们的类比知道,GNN是一种模型,它更多是end2end性质的,除此之外,很多GNN的输出有可能是高维向量.知道了一种是模型一种是思路之后,下面的话就很好理解了.

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577086151823.png)

### 图,有向图定义里和过去的不同之处
下图总结了图论里的常用变量记号和相关概念.这里面的大多数变量都不令人感觉到陌生,唯独有几个特殊的值得一记.

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577086408769.png)

* adjacency matrix: 这个是01矩阵,表示是否连接,这个和存储边的数值的矩阵需要进行区分
* **H**:hidden node feature vector. 这个令人奇怪的地方在于hidden究竟应该如何去理解?目前的理解方案是基于hidden layer的,
* 
