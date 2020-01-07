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

### GNN的分类
算法层面可以分为RecGNN，ConvGNN，Graph Autoencoders，spatial-temporal graph neural networks.

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577152229036.png)

按照输入输出分类任务层面,可以分为:节点输出,边输出和图输出.

按照训练(学习)方式划分,可以分为半监督,有监督和无监督学习.

下面是一些常见的组合:
1. semi-supervised learning for node-level classification.输入一张图中部分节点有标签部分节点没有标签进行训练,最终去预测没有标签的地方应该是什么标签,这就是半监督学习在节点层面的应用.常见架构是图卷积+softmax.可参见[这篇论文](https://arxiv.org/pdf/1609.02907.pdf)
2. supervised learning for graph-level classification. 这个比较常见,用有监督学习去分类一张完整的图.这些算法里主要用到图卷积层,图池化层和readout层.他们分别的作用是:
	* graph convolution layer: 获取节点特征表示,融合与之连接的节点信息来给出每个节点的高维度特征表示;
	* graph pooling layer: 和通常的pooling layer不同,这里主要是将一张整图池化为若干个子结构;
	* readout layer: 类似于分类作用中的全连接层,可以将若干个子图表示拼接为一个大的图表示.
	* Muti-layer Perception(MLP):多层感知机,通常后面再加一层softmax,这样就可以生成一个end2end的framework.
3. unsupervised learning for graph embedding. 关于什么是embedding以及embedding和GNN的结合方面前面已经写过了,这里就是一个具体的例子,GNN在embedding上的应用。这个工作就是输入一个没有标签的图，输出该图的embedding，但是由于其无标签特性以及end2end的结构，通常实现上述功能所构建的无监督学习主要有两种方案：
	* encoder-decoder模式。采用编码framework把图变成一个latent representation,也就是embedding，使用decoder把embedding重构成一个图，然后这二者的广义差作为最小化的优化目标进行自定义的无标签训练。
	* 第二种方式没看懂。就放一张图到此处吧。
	![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577154346190.png)
	参考论文：[方法1](https://arxiv.org/abs/1802.04407)  [方法1](https://arxiv.org/abs/1611.07308),[方法2](https://papers.nips.cc/paper/6703-inductive-representation-learning-on-large-graphs.pdf)

下面就具体介绍一下这些算法．
### Recurrent Graph Neural Network
提出较早,基于RNN提出.最初用于有向无环图中.后来出现扩展算法.比较经典的算法一个是基于information diffusion的GNN,该算法会不停地迭代每一个node的表征向量知道出现一个稳定的不动点为止.通常而言每个节点的hidden state可以被迭代地表征为:

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577174770856.png)

公式里面的X_v是指当前节点的feature vector,而X^e (v,u)是指当前节点与其邻居节点连接的边的特征向量,h_u^(t-1)是上一轮迭代产生的邻居节点u的hidden state,可以看见,神经网络f()实现了对节点特征,邻居特征,连接特征的融合封装,并且利用sigma求和来加强图之间的连接关系.

同时,为了让hidden向量h可以最终收敛到一个稳定的不定点,需要f()是一个压缩映射(contraction mapping).在神经网络中,构建一个压缩映射可以采用在Jacobian矩阵中添加惩罚项的方式,如下图所述.

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577175488378.png)

针对上述算法,为了减小step和解决"收敛"这个问题,出现了Graph echo state network,前面有一个随机出初始化的压缩映射的自动编码器,用以输出稳定的平衡状态,而后面的output layer需要进行训练.

之后出现了Gated Graph Neural Network,这个网络主要通过一个Gated Recurrent Unit (GRU)的结构去减小算法的迭代次数,同时这个结构也去掉了对收敛的压缩映射要求,当然,这时候的迭代关系已变为:

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577176339707.png)

可以看出,在这个迭代关系里面,节点v的hidden state只与自己和其他邻居节点的hidden state有关,不再和节点间边的特征或者节点的feature vector相关了.然后,与GNN初始的随机初始化不同,此处t=0时的h_v采用的就是节点v的feature vector X_v.

之后是stochastic steady-state Embedding,这个我觉得得没有什么太大意义去解释,故只给出公式,反正也比较好理解.

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577177058880.png)

总结一下: Recurrent的方法都是迭代式的,最终目的都是要找寻到一个特殊的fixed point.

### convolutional graph neural networks
convolution的GNN主要有两个研究方向,分别是基于谱图理论的spectral-based和基于空间的spatial-based,下面将分别基于以上主流思想进行下面的介绍.
#### spectral-based
这个方向具有较强的理论基础,同时,这个方向也有一些劣势.
首先,spectral-based需要假设图是**无向**的,这样一个无向图就可以用一个laplacian matrix刻画,表达式为:

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577259095322.png)

可以看出,由于这个矩阵的实对称半正定性质,他一定可以进行相似对角化,转变为
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577259167409.png)
这里面的 U就是特征向量组成的矩阵,而$\Lambda$就是特征值组成的对角矩阵.同时,这些特征向量彼此正交,也就是矩阵U与其转置的乘积为单位阵.除此之外,这些特征值就是这个graph的所谓谱.

对于这样的一张无向图,可以通过谱这样一个频率概念进行fourier transform和逆transform的定义,当把一张图映射到频率这样一个空间之后,便可以设置对应的滤波器了,这样定义出来的滤波器filter可以过滤掉特定"频率",之后再对之进行反滤波操作,便可以得到滤波之后得到的图.

而我们知道,时域下的卷积等价于频域里的乘积,所以时域下的图卷积可以定义为:

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577259760581.png)

之后,再根据CNN里面对卷积核进行参数学习的操作进行类比,就出现了第一版卷积运算的GNN即spectral CNN的定义:
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577259975257.png)
当然,这个算法面临着三个问题:
1. 脆弱性,抑或说应对外部噪声的鲁棒性.相似对角化这一操作会因为Laplacian Matrix的数值的轻微变化(这种变化多是由于某种噪声产生的)而使得特征值发生很大变化,因而不够鲁棒;
2. 不够flexible,训练得到的GNN无法处理laplacianMatrix变化的情况,包括图中添加删除了某个节点,或者表征连接关系的矩阵A内的某个元素发生了任何变化
3. 计算量大.算法复杂度是O(n^3)

这三个问题本质上都是因为需要进行eigen-decomposition这个步骤产生的,在现代控制理论里面也有类似的问题.让我们来看看GNN这块后来出现的算法是如何解决这个问题的吧!

切比雪夫GNN: chebyshev Spectral GNN对滤波器进行了近似,意图消除特征值分解这个步骤带来的问题.
初步定义的chebshev滤波器为:
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577260864734.png)
在这个公式中,theta是一个参数,而由于特征向量矩阵U的正定性,有$U \Lambda^~ U^T =I$成立,所以图卷积操作就可以被简化为
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577261038527.png)
可以看出,这时的图卷积操作就不再和U相关,仅仅需要求得Laplacian矩阵就可以了,上述的三个问题均得到了解决.

不过,对于上述公式,可以看出K是和迭代次数相关的,如果我们减小K的数值,那么就可以实现局部的强关联而非全局的强关联了.这样就完全摆脱了特征值分解的问题.下面的GCN就是这样做的:

GCN直接对上述公式进行了一阶近似,即K=1.这样,如果\lambda_max=2的话,上述公式就可以简化为
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577261567821.png)

**_至于这个公式为什么可以这样得出我并不清楚_**,不过我们还记得D是对角矩阵,对角线上的元素都是graph的度,而A则是存储了无向图连接关系的对称矩阵.

对theta取一组特殊的取值的话,可以得到
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577261792960.png)

这样就可以得到最终的结果如图中的公式12
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577261953452.png)
同时,也可用spatial的方法理解GCN表达为公式13那个样子.

> 此处不妨休息一下,想一想这些论文或者结构提出的背后,是不是有某种特殊的规律?或者说:这些论文的提出者在面对这些算法的时候,是如何想出改进的点子的? 让我们从最基本的spectral GNN开始,他的思路来自于"卷积"的定义,学过复变函数和积分变化的人都知道卷积和Laplace变换和卷积的关系,结合spectral graph theory,这种恰如其分的定义就油然而生了.所以写这篇论文的人肯定对信号处理很熟悉.那么之后的chebshevGNN呢?他抓住的改进的点是矩阵的特征值分分解,所以他提出了一种方法去"绕过"特征值分解,至于这个数学方法,我认为比不过这个改进的思路重要.那么,GCN的产生灵感? 我不认为GCN来自于chebshev的改进,这个灵感更像是先给出了上图中公式13之后,又给其找到了新的使用原因.当然,这里也有借鉴比如:凡是无穷级数\迭代+拟合的情况,统统用一阶效果就非常好.行文至此,如果让我对GCN进行改进,我可以在哪一点上做文章呢?我稍稍思索了一下,认为只能在图的结构的鲁棒性上进行增强了.

很不幸,目前的主流研究同样也是在这个方向上进行的.不过我觉得目前的研究还没有达到值得记笔记的程度,所以就暂时在此处停止,以后或许会补充.

#### spatial-based
所谓的spatial-based convGNN就是在考虑拓扑关系的基础上直接建立的,主要还是类比于conv在image上的操作,比如:这一幅我们过去曾经画过的图.
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1577264864131.png)



