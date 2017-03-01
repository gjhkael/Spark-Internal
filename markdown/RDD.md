# RDD

##RDD是什么

RDD的全名是Resilient Distributed Dataset，意思是容错的分布式数据集，每一个RDD都会有5个特征：

    1、有一个分片列表。就是能被切分，和hadoop一样的，能够切分的数据才能并行计算。
    2、有一个函数计算每一个分片，这里指的是下面会提到的compute函数。
    3、对其他的RDD的依赖列表，依赖还具体分为宽依赖和窄依赖，但并不是所有的RDD都有依赖。
    4、可选：key-value型的RDD是根据哈希来分区的，类似于mapreduce当中的Paritioner接口，控制key分到哪个reduce。
    5、可选：每一个分片的优先计算位置（preferred locations），比如HDFS的block的所在位置应该是优先计算的位置。
要是每个RDD都可以把上述的5个特征搞清楚，那么RDD也就搞的很透彻了，简单的说，RDD就是一个抽象类，上面有各种实现，每一种实现，就是一个RDD，
RDD可以转换成另外一种RDD（前提是可以转换），简单的说，RDD其实封装了数据的源头、数据如何计算、计算完返回上面结果。RDD根据功能可以分为：
transformation和action两大类。

##RDD分类

| Transformation | Meaning |
|:-----------|:-------------|
| map(func) | 对每个元素进行func的计算，然后返回一个新的RDD |
| flatMap(func) | 对每个元素进行func的计算，然后返回一个新的RDD 与map的不同之处返回的结果集合类型不一样 |
| filter(func) | 对每个元素进行func的计算，结果为true的元素得到保存，然后返回一个新的RDD |
| mapPartitions(func) | 和map类似，但是它是对partition为单位进行计算 |
| mapPartitionsWithIndex(func) |和mapPartitions类似，但是对每个partition进行了索引 |
| sample(withReplacement, fraction, seed)| 对一个集合的取样操作，第一个参数运行结果是否重复，第二是生成集合的大小，第三个是随机种子 |
| union(otherDataset) | 与另一个RDD进行合并操作 |
| intersection(otherDataset) | 与另外一个RDD取交集|
| distinct([numTasks])) | 返回去重后的RDD， numTasks是生成RDD的partition个数，不设采用RDD中partitions.length |
| groupByKey([numTasks]) | 按key进行聚合操作，< k,v > ==>  < k,Iterable < V > > |
| reduceByKey(func, [numTasks]) | 按key进行reduce操作，< k,v > ==> < k,V > v通过func算出来|
| aggregateByKey(zeroValue)(seqOp, combOp, [numTasks]) | |
| sortByKey([ascending], [numTasks]) | 按key进行排序，< k,v > ==> < k,V >|
| join(otherDataset, [numTasks]) | 与另外一个RDD进行Join操作  |
| cogroup(otherDataset, [numTasks]) | 与另外一个RDD进行按key聚合< k,v > < k,u > ==> < k,(Iterable< V >, Iterable< W >) > |
| cartesian(otherDataset) | 笛卡尔积操作 k, u => < k,u > |
| pipe(command, [envVars]) | 每个partition经过一个shell command操作返回新的RDD |
| coalesce(numPartitions) | 对RDD进行重新分区，主要用来减少分区，默认不开启shuffle|
| repartition(numPartitions) |对RDD进行重新分区，一种特殊coalesce，开启shuffle|
| repartitionAndSortWithinPartitions(partitioner)  | 重分区并且按key进行了排序 |


| Action | Meaning |
|:-----------|:-------------|
| reduce(func) | |
| collect() | 对上一个RDD的元素返回一个集合 |
| count()| 求和 |
| first() | 返回第一个元素 |
| take(n) | 返回地n个元素 |
| takeSample(withReplacement, num, [seed]) | 对结果进行取样返回 |
| takeOrdered(n, [ordering]) | 返回前n，并且这些数据是排好序的 |
| saveAsTextFile(path) | 将结果保存到一个路径 |
| saveAsSequenceFile(path)  | 以Hadoop SequenceFile文件形式存储文件  |
| saveAsObjectFile(path)  | 保存java序列化后的数据，可以配合 SparkContext.objectFile()使用  |
| countByKey()| 对key进行求和，<k,v> ==> <k, count> |
| foreach(func)| 对每个元素进行func的操作常用来和外部存储系统打交道 |



## RDD demos
#####demo1

textFile、map、flatMap、filter、reduceByKey、foreach、sortByKey、takeOrdered
``` java
val file = sc.textFile(args(0))       //如果是本地文件：file:///home/xxx/xxx
val result = file.flatMap(_.split(" ")).filter(s => s.length>10).map(x => (x, 1)).reduceByKey(_ + _).cache()
result.foreach{x => println(x._1 + " " + x._2)}
val sorted=result.map{ case(key,value)=>(value,key)}.sortByKey(true,1)
val topk=sorted.top(10)
//sorted.takeOrdered(10)(Ordering[Int].reverse.on(x=>x._1))
topk.foreach(println)
```

#####demo2
parallelize、cartesian、collect、foreach、coalesce、parallelize、sample、join、cogroup、union、pipe
```java
//求笛卡尔积
val x = sc.parallelize(List(1, 2, 3, 4, 5))
val y = sc.parallelize(List(6, 7, 8, 9, 10))
val cartesian = x.cartesian(y)
cartesian.foreach(println)
//求交集
val intersection = x.intersection(y)
intersection.foreach(println)

//重分区
val t = sc.parallelize(1 to 100, 10)
println(t.partitions.size)
val coalesce = t.coalesce(2, true)
//val coalesce = y.coalesce(12, false)  //无效
val repartition = y.repartition(12);
println(coalesce.partitions.size)
println(repartition.partitions.size)

//去重
val distinct = t.distinct(2);
distinct.collect().foreach(println)

//sample
val sample = t.sample(false, 0.2, 0)
sample.collect.foreach(x => print(x + " "))

//pipe
//val pipe = t.pipe("head -n 2")  //window下执行不了

//union
val union = x.union(y)
union.foreach(print)
//count
println(sample.count)
//cogroup
val d = x.map((_, "b"))
val e = y.map((_, "c"))
val cogroup = d.cogroup(e)
cogroup.foreach(println)
//join
val data1 = Array[(Int, Char)]((1, 'a'), (2, 'b'), (3, 'c'), (4, 'd'), (5, 'e'), (3, 'f'), (2, 'g'), (1, 'h'))
val pairs1 = sc.parallelize(data1, 3)
val data2 = Array[(Int, Char)]((1, 'A'), (2, 'B'), (3, 'C'), (4, 'D'))
val pairs2 = sc.parallelize(data2, 2)
val result = pairs1.join(pairs2)
result.foreach(x=>println(x._1, x._2))

```

## one demo

通过一个实例来讲讲RDD内部情况
``` java
import org.apache.spark._

object HdfsWordCount {
 def main(args: Array[String]) {
    if (args.length < 1) {
      System.err.println("Usage: HdfsTest <file>")
      System.exit(1)
    }
    val sparkConf = new SparkConf().setAppName("HdfsTest").setMaster("local")
    val sc = new SparkContext(sparkConf)
    val file = sc.textFile(args(0))       //如果是本地文件：file:///home/xxx/xxx
    val result = file.flatMap(_.split(" ")).map(x => (x, 1)).reduceByKey(_ + _).cache()
    result.foreach{x => println(x._1 + " " + x._2)}
    sc.stop()
  }
}
```
整个job先经过多个RDD的transformation操作，从textFile开始，textFile方法会new 一个hadoopRDD,然后通过flatMap操作变成MapPartitionsRDD，再经过map
仍然是MapPartitionsRDD，在经过reduceByKey变成ShuffleRDD，最后通过foreach方法触发作业的执行。

###HadoopRDD
我们首先看textFile的这个方法，进入SparkContext，找到该方法。

```java
  def textFile(
      path: String,
      minPartitions: Int = defaultMinPartitions): RDD[String] = withScope {
    assertNotStopped()
    hadoopFile(path, classOf[TextInputFormat], classOf[LongWritable], classOf[Text],
      minPartitions).map(pair => pair._2.toString).setName(path)
  }

```
不难发现：传给hadoopFile方法的前四个参数：path,TextInputFormat,LongWritable,Text和我们写mapreduce程序类似。

通过hadoopFile函数，首先会new一个hadoopRDD，紧接着调用了map方法（MapPartitionsRDD）并吧<k,v>的第二个v值输出。其实此处的<k,v>就是通过hadoop的
TextFileInputFormat的recordReader的getNext()方法不断的生成k:行数、v:改行数据。接着hadoopFile方法中new了一个HadoopRDD,接下来看看。

还是从上述的5个方面来看HadoopRDD:compute、getPartitions、getDependencies、getPreferredLocations、partitioner。

 **getPartitions**

```java
  override def getPartitions: Array[Partition] = {
    val inputFormat = getInputFormat(jobConf)
    val inputSplits = inputFormat.getSplits(jobConf, minPartitions)
    val array = new Array[Partition](inputSplits.size)
    for (i <- 0 until inputSplits.size) {
      array(i) = new HadoopPartition(id, i, inputSplits(i))
    }
    array
  }
```
核心代码如上，首先利用了反射技术获得TextInputFormat实例，然后通过FileInputFormat的getSplit方法获取splits，最后将split数组进行了包装，返回HadoopPartition.

**compute**

```java
private val split = theSplit.asInstanceOf[HadoopPartition]
     
      private val jobConf = getJobConf()
      private val existingBytesRead = inputMetrics.bytesRead
      private var reader: RecordReader[K, V] = null
      private val inputFormat = getInputFormat(jobConf)

      reader =
        try {
          inputFormat.getRecordReader(split.inputSplit.value, jobConf, Reporter.NULL)
        } catch {
          case e: IOException if ignoreCorruptFiles =>
            logWarning(s"Skipped the rest content in the corrupted file: ${split.inputSplit}", e)
            finished = true
            null
        }
      // Register an on-task-completion callback to close the input stream.
      context.addTaskCompletionListener{ context => closeIfNeeded() }
      private val key: K = if (reader == null) null.asInstanceOf[K] else reader.createKey()
      private val value: V = if (reader == null) null.asInstanceOf[V] else reader.createValue()

      override def getNext(): (K, V) = {
        try {
          finished = !reader.next(key, value)
        } catch {
          case e: IOException if ignoreCorruptFiles =>
            logWarning(s"Skipped the rest content in the corrupted file: ${split.inputSplit}", e)
            finished = true
        }
        if (!finished) {
          inputMetrics.incRecordsRead(1)
        }
        if (inputMetrics.recordsRead % SparkHadoopUtil.UPDATE_INPUT_METRICS_INTERVAL_RECORDS == 0) {
          updateBytesRead()
        }
        (key, value)
      }

```
核心代码如上，可以看到，compute是如何获取数据的，每个partition到compute函数，先获取inputFormat实例，然后通过RecordReader进行实际数据的读写，
然后上层调度框架不断getNext方法来获得<k,v>的数据。

**partitioner**

partitioner没有覆盖RDD抽象类，所以是None

**getPreferredLocations**
```
locs.getOrElse(hsplit.getLocations.filter(_ != "localhost"))
```
返回InputSplit返回localtion信息。

**getDependencies**
``` java
class HadoopRDD[K, V](
    sc: SparkContext,
    broadcastedConf: Broadcast[SerializableConfiguration],
    initLocalJobConfFuncOpt: Option[JobConf => Unit],
    inputFormatClass: Class[_ <: InputFormat[K, V]],
    keyClass: Class[K],
    valueClass: Class[V],
    minPartitions: Int)
  extends RDD[(K, V)](sc, Nil) with Logging {

```
首先hadoopRDD会调用父类构造器，将空的List传给抽象类RDD，这样getDependencies返回改数组
```
protected def getDependencies: Seq[Dependency[_]] = deps
```
对于HadoopRDD来说，因为它是第一个RDD，所有没有前依赖，所以deps是空数组

###MapPartitionsRDD
刚刚提到，HadoopRDD之后会调用map(pair => pair._2.toString)来获得value的值，然后我们程序中显示的调用了flatMap和map还有reduceByKey,最后调用foreach结束，
因为map和flatMap都是使用的MapPartitionsRDD（在spark早期版本，flatMap和Map分别对应了两个RDD，一定程度上有代码的冗余，后面统一使用MapPartitionsRDD更简洁），
reduceByKey则生成了ShuffleRDD。所以接下来看看MapPartitionsRDD：

```java

  /**
   * Return a new RDD by applying a function to all elements of this RDD.
   */
  def map[U: ClassTag](f: T => U): RDD[U] = withScope {
    val cleanF = sc.clean(f)
    new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.map(cleanF))
  }

  /**
   *  Return a new RDD by first applying a function to all elements of this
   *  RDD, and then flattening the results.
   */
  def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U] = withScope {
    val cleanF = sc.clean(f)
    new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.flatMap(cleanF))
  }

```
如上，RDD中map和flatMap都是通过MapPartitionsRDD来进行计算的。最终的区别就是iter这个迭代器使用的是map还是flatMap计算。
```java
private[spark] class MapPartitionsRDD[U: ClassTag, T: ClassTag](
    var prev: RDD[T],
    f: (TaskContext, Int, Iterator[T]) => Iterator[U],  // (TaskContext, partition index, iterator)
    preservesPartitioning: Boolean = false)
  extends RDD[U](prev) {

  override val partitioner = if (preservesPartitioning) firstParent[T].partitioner else None

  override def getPartitions: Array[Partition] = firstParent[T].partitions

  override def compute(split: Partition, context: TaskContext): Iterator[U] =
    f(context, split.index, firstParent[T].iterator(split, context))

  override def clearDependencies() {
    super.clearDependencies()
    prev = null
  }
}
```
很简单，getPartitions通过获取第一个父RDD的Partitions来作为自己的partitions。

getDependencies:注意到该类的构造函数，extends RDD[U] (prev)调用超类构造函数：
```java
  def this(@transient oneParent: RDD[_]) =
    this(oneParent.context, List(new OneToOneDependency(oneParent)))
```
可以看到，把传过来的RDD构造一个OneToOneDependency,也就是说，MapPartitionsRDD的依赖是一个OneToOneDependency，并且它的父依赖是HadoopRDD。
从demo中，依赖关系可以看到是HadoopRDD<--MapPartitionsRDD<--MapPartitionsRDD<--MapPartitionsRDD,因为有三个MapPartitionsRDD操作。

Partitoner：如果preservesPartitioning设置为true，使用第一个RDD的partitioner，否则就是None

Compute: compute就是将前面一个RDD的Iterator调用f函数计算一下。

###reduceByKey

reduceBykey在PairRDDFunctions类里面，RDD里面的方法通过隐式转换得到
```
  def reduceByKey(partitioner: Partitioner, func: (V, V) => V): RDD[(K, V)] = self.withScope {
    combineByKeyWithClassTag[V]((v: V) => v, func, func, partitioner)
  }
```

reduceByKey最终调用
```
def combineByKeyWithClassTag[C](
      createCombiner: V => C,
      mergeValue: (C, V) => C,
      mergeCombiners: (C, C) => C,
      partitioner: Partitioner,
      mapSideCombine: Boolean = true,
      serializer: Serializer = null)(implicit ct: ClassTag[C]): RDD[(K, C)] = self.withScope {
    require(mergeCombiners != null, "mergeCombiners must be defined") // required as of Spark 0.9.0
    if (keyClass.isArray) {
      if (mapSideCombine) {
        throw new SparkException("Cannot use map-side combining with array keys.")
      }
      if (partitioner.isInstanceOf[HashPartitioner]) {
        throw new SparkException("HashPartitioner cannot partition array keys.")
      }
    }
    val aggregator = new Aggregator[K, V, C](
      self.context.clean(createCombiner),
      self.context.clean(mergeValue),
      self.context.clean(mergeCombiners))
    if (self.partitioner == Some(partitioner)) {
      self.mapPartitions(iter => {
        val context = TaskContext.get()
        new InterruptibleIterator(context, aggregator.combineValuesByKey(iter, context))
      }, preservesPartitioning = true)
    } else {
      new ShuffledRDD[K, V, C](self, partitioner)
        .setSerializer(serializer)
        .setAggregator(aggregator)
        .setMapSideCombine(mapSideCombine)
    }
  }
```
解释一下什么函数：首先mergeCombiners必须不为空，也就是（c,v）=> c函数不为空，其次对createCombiner等参数进行clean操作，清除不能序列化的数据。
最后，如果当前的partitioner存在的话，进行mapPartitions操作，如果partitioner不存在的话，进行ShuffledRDD操作，其中将Partitioner（默认是基于Hash）传递给ShuffledRDD.

接下来看一下ShuffleRDD

### ShuffleRDD



## RDD之间联系


