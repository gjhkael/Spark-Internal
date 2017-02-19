# Spark-Internal
[deployDoc](https://github.com/gjhkael/deployDoc) 项目是在研究生期间做的一些工作的总结和学习，时间过了近两年了，Spark也由当时的1.2.0到现在的
2.0.x了，Spark在整个过程中，经过了好多次大改，akka通信被废除，取而代之的是Netty,Alluxio的数据传输也是采用的Netty,在下面相应的模块下会进行介绍分析,
功能也更加趋于完善，许久没接触Spark，现在重新开始学习Spark，对Spark来个较为透彻的学习。以前很多模糊的地方也就一带而过，现在开始需要追根究底。

##简单介绍

Spark项目太火了，经过这几年的发展，已经渐渐的成熟，许多公司都开始运用到生产上，最主要的有三大模块：1.Spark Streaming用来从kafka接入并过滤数据然后落地到
数据仓库；2.Spark SQL从数据仓库中查询数据生成报表；3.Spark MLlib进行机器学习相关的运算。本项目首先会对Spark core进行深入的分析，然后分析Spark Streaming
和Spark SQL。机器学习水太深，并且平时工作中接触不到，就不涉水了。

##主要内容
对Spark进行比较全面的分析，将从以下几个方面着手。

1. [RDD](https://github.com/gjhkael/Spark-Internal/blob/master/markdown/RDD.md) RDD的分析
2. [Job and Task Scheduling]() Spark的作业和任务调度
3. [Architecture]() Spark的架构分析
4. [Shuffle]() Spark Shuffle分析
5. [Spark Streaming]() Spark Streaming源码分析
6. [Spark SQL]() Spark SQL源码分析
