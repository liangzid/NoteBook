---
title: HyperMinHash的学习 
tags: AlgorithmLearning, sketch
renderNumberedHeading: true
grammar_cjkRuby: true
grammar_mathjax: true
grammar_mindmap: true
---
为了做实验,今天下午和晚上又学习了HyperMinHash算法.该算法是一种采用OPH类似的想法获取sketch用_浮点数格式存储_(其实就是一个元组而已)的算法,因此可以保证在时间复杂度(OPH的贡献)和空间复杂度(floating-point store)上的双优.说来容易,或许是某种特殊的排斥心理,我在读这篇论文时产生了巨大的困难.下面尝试把他学明白,毕竟是需要我自我实现以做实验的.
## sketch的生成

该算法的大意如下图所示.
![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1575895437488.png)
上述图片的运行步骤为:
* 将集合内的元素映射到(0,1)之间;
* 对于每一个小数,将其分割为前**p**bit和剩余bit,其中参数p决定了sketch的长度,也就是2^p=k;
* 对于上述提到的"剩余比特",如果是minhash,则会在前p bit相同的情形下寻找剩余比特里最小的若干位(如64位,也就是上图中间所示);但是,在本算法里,为了降低存储空间,仍然需要对每一位存储的数值进行进一步的压缩,本文采取的策略是: **指数+小数**法,亦即记录第一次出现1的位数(可以使用q位存储,如q=6),如图蓝色 (+与) 剩下一定位数(可以用r位存储,如r=4),如图红色.这样仅仅使用**p*(q+r) bit**就可以完成了!

把上述Hash思想整理成伪代码,如下:

![enter description here](https://raw.githubusercontent.com/liangzid/LittleBook/master/小书匠/1575897017761.png)

很显然,这个元素序列B就是需要的sketch了!

## 并集的处理
但是