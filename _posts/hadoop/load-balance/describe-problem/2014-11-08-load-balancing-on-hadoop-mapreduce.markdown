---
layout: post
title: "Hadoop MapReduce中的负载均衡问题 Load Balancing on Hadoop MapReduce"
date: 2014-11-08 11:34:58 +0800
img: elephants.png
tags: [load balance, hadoop, mapreduce]
---
做了一段时间的Hadoop中的负载均衡问题，记下些流水账。

## 一、负载均衡 -- 一个广泛而普遍存在的问题

负载均衡问题是一个广泛而普遍存在的问题。在所有的分布式系统中几乎都会提及到“长尾问题(Long Tail Problem)”，其实也就是大家常说的“短板理论”，系统的整体表现取决于表现最差的一部分。常见的分布式系统如分布式缓存，分布式存储，分布式计算，分布式数据库等等，都存在这个问题。分布式缓存中可能会遇到短时间内集中访问同一个缓存的情况；分布式存储可能单机磁盘使用过度；分布式计算可能会有单点的计算负担过重；分布式数据库可能会有单机访问量过大。如此总总，只要是分布的，想完全端平一碗水几乎是不大可能的。

在此，总结我对负载均衡的定义：在多点协作的系统中由于不合理的任务分配导致某个或者少量的某些节点处理负担过重，最终拖延整个系统对外的响应效率。

负载均衡的解决思路主要有：

	1. 被动解决，发现负载倾斜后，将负载迁移到空闲节点
	2. 主动预防，防止倾斜的发生
		2.1 系统任务分配方式主动预防
			2.1.1 静态负载均衡
			2.1.2 动态负载均衡
		2.2 用户先验知识的介入

负载均衡问题的解决在大多数情况下是存在一个极限的，这取决于具体问题的可划分性([2.2中对此有讨论](/blog/load-balancing-on-hadoop-mapreduce/#upbound))。
分布式系统的负载均衡问题已经研究了多年了，有些问题早有了较成熟的解决方案，像分布式缓存系统中常见的一致性哈希算法等。在这里主要讨论的是分布式计算平台Hadoop里的负载均衡问题。

[后文](/blog/analyzing-load-balancing-on-hadoop/)会对这几种方式一一讨论，下面先对本文的主要讨论对象Hadoop的背景做个简要介绍。
<!--more-->
## 二、Hadoop 

Hadoop自推出以后在互联网快速发展的背景下得到了许多公司的认可，已然成为大数据的基础处理平台甚至是行业标准。Facebook，Amazon，Yahoo等等公司都在自己的系统中构建了基于Hadoop的处理平台。除了最基本的数据处理功能，在Hadoop之上现在已经发展出来一套生态系统，应用最广泛的莫过于Hive和HBase了。在Hadoop之上构建的系统都直接或间接地使用了了Hadoop的分布式存储模块HDFS和计算模块MapReduce。本文的问题关注计算模块MapReduce的均衡问题。

### 2.1 HDFS
HDFS借鉴的是Google的GFS系统，是一个基于Key/Value的分布式存储系统。HDFS是为了大文件、一次写多次读的应用场景而设计的。所有要存储在HDFS中的文件需要按块（默认64M）切分，每个数据块有在不同的机器上（默认是本机，本机架，不同机架）有多个备份（默认为3份）。系统通过对失败机器数据文件的再分配、复制来自动保证文件的数据安全。HDFS并不适合大量小文件或者对写要求高的场景。这样，我们可以有个概念，Hadoop中处理的数据会分块备份三分存在不同的机器上。

HDFS本身也存在负载均衡问题，这个负载的均衡主要只每台机器的磁盘使用率。假如有一台机器存储了大量的数据，而其他机器存储了很少，这就是一个倾斜的情形。HDFS的存储倾斜不仅仅只影响到磁盘使用情况，同时由于Hadoop的Map的执行依赖于输入数据在磁盘上的分布情况（Hadoop期望达到最好的数据本地化处理）它也影响到Map计算过程中的均衡。HDFS存储的不均很有可能导致Map计算的分布不均（注意是有可能，因为HDFS上的输入数据有多个备份，Map的输入只需要一份备份，因此不一定会导致Map计算不均）。

对于HDFS的倾斜问题，Hadoop本身提供了一套机制来限制不均衡的程度。Hadoop自带的工具`bin/start-balancer.sh`可以通过参数指定系统中均衡的标准如10%，这就保证了系统中磁盘的使用率的偏差在10%之内。如果超过了这个值，系统将自动执行数据块的重分布工作使之达到偏差限额。

### 2.2 MapReduce

MapReduce是Hadoop的数据处理模块，算是函数式编程的巅峰之作了吧。Hadoop对数据的处理都被抽象成Map和Reduce这两个函数的操作。

#### Map

通常地，Map函数的工作是从HDFS中读取上输入文件，读入的数据是一个个（MapInputKey/MapInputValue）对，根据作业需求处理后输出一个个（MapOutputKey/MapOutputValue)对，后台的输出线程会把输出的文件按照MapOutputKey把对应的MapOutputValue合并起来（MapOutputKey-->MapOutputValue0,MapOutputValue1,...)，同时还会将输出按照MapOutputKey排序（注意，每一个Map都会有同样的样的输出，不同的Map会有同样的Key值输出）。逻辑上，我们可以将不同Map输出的同一个Key的数据合起来看做一个小Partition（Finer Partition，FP）。

#### Shuffle

Shuffle被称为Hadoop的核心，但是对应一般用户并不会涉及到这部分的细节。系统提供一个Partition接口使得用户可以决定Map的输出Key该如何聚合（比如想把Key为奇数的FP放在一起，把偶数放在一起）。Shffle的主要工作是将各个Map的FP的各部分按照用户指定的方式将数据从Map的输出端拷贝到对应的Reduce执行端。

在大多数研究中Shuffle的倾斜问题都没有具体考虑，事实上Shuffle这个过程本身也是存在倾斜问题的。主要原因是各个机器上运行的reduce任务的处理数据量的不均。各个机器上运行的Reduce都要从Map端拷贝相应数据，如果这些要拷贝的数据在本地的话那么必然会拷贝得快些（虽然对于本地数据Hadoop的Shuffle还是使用的Http协议向本地Servlet请求下载）；另一方面，一台机器上有多个Reduce在同时下载数据，这台机器的网卡速度及磁盘读写能力都成为这台机器上的Shuffle过程的瓶颈因素。

然而Shuffle这部分倾斜被忽略并不是没有道理的。从企业生产的角度，我们并不关心单个机器的处理时间、通信量。事实上我们最关注的是，对于提交任务的用户而言有最快的响应速度；对整个系统集群而言网络通信量最小，单位时间内处理的任务最多。Shuffle的倾斜问题是中间的一个过渡状态，它是由Map数据的输出不均、任务分配不均导致的；同时它也有可能导致最终Reduce的任务处理时间差异。Shuffle过程中的倾斜并不一定导致最终的倾斜，相反在有些推测执行任务出现的时候，Shuffle的不均有可能还会提升最终性能表现。

总之，Shuffle的均衡既不是目标，也不是高性能的必要条件，因此对于这部分的研究意义不大。

#### Reduce

Reduce将Map输出的各个FP拷贝到本地(拷贝过程中还是保证键值对的有序性)，然后对于每个键值对序列(ReduceInputKey--> ReduceInputValue0，ReduceInputValue1...)做处理。对于MapReduce模型本身，如果要保证计算的正确性，我认为至少要保证的是：

>单独一个键的FP必须要保证完整的拷贝到同一台机器上。而不是看起来的，同一个Hash值对应到的Partition的多个键的FP数据要保证到同一台机器上。

多数计算中即使多个键的FP被Shuffle到同一台机器上，处理时我们还是每次以一个键的FP作为独立的计算输入单元。因此，我们在写MapReduce程序时，应当需要注意的是：

>不应当将程序计算的正确性依赖于Partition函数的实现，而只应将Partition函数作为一个提升系统数据均衡性的用户接口。

上面这一点也是我们下面提出的各种负载均衡算法的基本依据。如果用户程序不满足上的条件，那么对于这种应用只能做Reduce任务分配级别的均衡，再低层的均衡会影响程序的正确性。而这种问题，Hadoop本身的推测执行机制基本能够满足需求，因此下文不做讨论。这也是均衡算法的能达到的上界(单个Key的FP是Reduce输入数据的最小不可分单元)。在此，下面都假设用户作业满足上面的条件。

## 结束语
本来准备继续写的，但是一篇博文过长感觉不太好，本文先到这里结束。文章简要介绍了负载均衡问题，并针对Hadoop平台上各个部分的运行机制分析了各个部分出现倾斜的情况。指出Hadoop平台上的负载均衡问题重点在Reduce部分。[下一篇将详细分析Hadoop倾斜发生的原因及解决方案](/blog/analyzing-load-balancing-on-hadoop/)。

