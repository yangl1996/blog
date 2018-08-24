---
date: "2015-09-20T00:11:17Z"
aliases:
    - /2015/09/20/imported_from_old_blog_67.html
title: 在TK1上写CUDA遇到的几个坑
---

最近在TK1上优化一个图像程序，用CUDA做并行，原来预期提速5-10倍的，结果跑下来不到两倍，觉得很奇怪，其实是被TK1上CUDA的一系列坑坑惨了。

首先是TK1的GPU频率，我估计因为K1（TK1上的SoC）说到底还是个移动处理器（尽管Nvidia宣传TK1是个超级计算机），所以默认锁在一个非常低功耗的GPU频率，以至于我多次优化CUDA代码，却仍然完全没有加速效果。组里一个师兄提醒之后改了（他也被坑过），速度立刻快了不止两倍。

另外就是TK1的CPU性能，简直不是一般的低，所以能用CUDA写的还是尽量放到GPU上跑。往往host code会成为整个程序的一个小瓶颈，这个时候用OpenMP做个简单的加速，效果还是很好的。同样，CPU的性能管理也不是默认处在最高性能模式，这也可以简单的调一下。调整方法：<a href="http://elinux.org/Jetson/Performance" target="_blank">http://elinux.org/Jetson/Performance</a>

说到OpenMP，事实上OpenMP可以和CUDA同时使用，甚至同时发挥效果。比如，一个for loop用OpenMP加速，loop中调用了CUDA的device code。这个时候两个并行方法都会发挥作用，加速会很明显。

CUDA在TK1上的内存访问也是比较慢的，造成调用device code时做的cudaMemCpy产生比较大的overhead。内存频率也可以调，但我试了试影响不大。这方面主要还是整合多个device code，尽量减少内存在CPU和GPU之间的拷贝。CUDA的shared memory也是必须好好加以利用的。

做了上述改动之后，程序的马上到了10X的提速，效果很不错。