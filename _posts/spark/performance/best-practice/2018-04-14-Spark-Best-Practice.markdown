---
layout: post
title: Spark 应用的常见性能考虑
tags: [spark, performance, best-practice]
img: dolphins.png
---

## General Idea
最近工作中用了些Spark，了解稍多些后感觉Spark和Hadoop还是一脉相承的，两者虽有着明显的不同，但是在优化方面却更多的是相通之处。考虑的因素还是减少重复计算，减少网络传输，充分利用计算资源。常见的Spark优化trick大概就下面这几点。
* 避免Shuffle
* 缓存多次使用的中间结果
* 适当调整并行度
* 使用好BroadCast Variable和Accumulator
* Tuning GC

写应用的时候不妨先画出数据转换DAG，可能帮助你一眼就找到一个不错到方案。

另外，代码优化的那句老话‘切勿提前优化’可能在数据处理方面不是很合适。提前优化或者有意识的注意性能很重要，一段毫秒级的延迟在micro sevice的一次调用来看可能没必要去影响结构去优化，而在
big data的环境中大量的数据集中并放大了可能的每一点优势，越早有意识优化越能减少后期的改动。

## 避免Shuffle
Shuffule 从Hadoop MapReduce开始就广为诟病，Spark的Shufulle虽然不像Hadoop一样强调必须做有序的Shuffle，但是在大量数据的加持下Shuffle带来的网络数据传输，Reducer内存消耗还是相当可观的。和Hadoop一样，Spark中也应当将计算前移，能在Map端做的工作尽量在Map端做，能在Map端aggregate的数据尽量在Map端aggregate。

### *ReduceByKey -> AggregateByKey -> CombineByKey -> BroupByKey*
Spark中ReduceByKey，AggregateByKey, CombineByKey and GroupByKey是常见的会导致Shuffle的操作
。大概了解下这四种操作是如何工作的就能很容易理解为甚么这四个操作从ReduceByKey到GroupByKey使用的优先级依次降低了。
#### ReduceByKey
#### AggregateByKey
#### CombineByKey
#### GroupByKey
### Coalesce Partition
### Maintain Partition
## 缓存中间结果
Cache
Persist Level
## 调整并行度
### Partition 
#### More Partition More Parallelism
#### Less Partition Less Overhead
TreeReduce & TreeAggregation
## Broadcast variable & Accumulator

## Tuning GC
### Intermediate Objects
### Serialization
## Other Tiny Points
### Collect or count large RDD
