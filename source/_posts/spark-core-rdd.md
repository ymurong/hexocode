---
title: Spark Core - RDD
date:  2019-08-14 12:00:00
tags:
- spark
---
> An introduction to the basic abstraction in spark - RDD and several RDD programming examples. 

## What is RDD
[Github Spark RDD source code](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rdd/RDD.scala)
```
abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging
```
By reading the RDD code sourcem, we could know that: 
- RDD is an abstract class
- RDD a class with generic types [String, Person, User ...]

RDD refers to **Resilient Distributed Dataset**, represents an **immutable**, **partitioned collection of elements** that can be operated on in parallel.
- Resilient: how to be fault tolerant during distributed computing => capability of tracing dependencies and repairing if a node failed or a partition data has been lost
- Distributed: Similar to MapReduce, all in-memory data, programs are stored and computed across the cluster. 
- Dataset: collection of elements
- Partitioned collection of elements: Array(1,2,3,4,5,6,7,8,9,10) => (1,2,3),(4,5),(7,8,9,10) 3 partitions that could be operated on 3 worker nodes in parallel.

## 5 characteristics of RDD 
Internally, each RDD is characterized by five main properties:

- A list of partitions
  In HDFS, a file could be separated into multiple blocks of data. Similarlly, in-memory dataset in spark are also partitioned across difference nodes.

```
/**
   * Implemented by subclasses to return the set of partitions in this RDD. This method will only
   * be called once, so it is safe to implement a time-consuming computation in it.
   *
   * The partitions in this array must satisfy the following property:
   *   `rdd.partitions.zipWithIndex.forall { case (partition, index) => partition.index == index }`
   */
protected def getPartitions: Array[Partition]
```


- A function for computing each split/partition
  y = f(x)
  rdd.map(_+1) => every operation will be done to each partition of RDD
```
/**
   * :: DeveloperApi ::
   * Implemented by subclasses to compute a given partition.
   */
@DeveloperApi
def compute(split: Partition, context: TaskContext): Iterator[T]
```

- A list of dependencies on other RDDs
  rdd1 => rdd2 => rdd3 => rdd4
  dependencies: *****
  rdda = 5 partitions
  ==> map
  rddb = 5 partitions 
  During the rddb computation, if the 3rd partition of rdda has been lost, spark will only recalculate the 3rd partition from the file system.

```  
/**
   * Implemented by subclasses to return how this RDD depends on parent RDDs. This method will only
   * be called once, so it is safe to implement a time-consuming computation in it.
   */
protected def getDependencies: Seq[Dependency[_]] = deps
```


- Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
```
/** Optionally overridden by subclasses to specify how they are partitioned. */
@transient val partitioner: Option[Partitioner] = None
```
- Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)
  Computation will be calculated on the node where there are data needed. (Move computation > Move data)
```  
 /**
   * Optionally overridden by subclasses to specify placement preferences.
   */
protected def getPreferredLocations(split: Partition): Seq[String] = Nil
```

## RDD creation (pyspark)
1. via Parallelized Collections
```
In [1]: data = [1,2,3,4,5]                                                                                                                                                                                                                            

In [2]: distData = sc.parallelize(data)                                                                                                                                                                                                               

In [3]: distData.collect()                                                                                                                                                                                                                            
Out[3]: [1, 2, 3, 4, 5]

```
> Note that the number of data partition depends on the number of spark cluster worker threads. 
pyspark --master local[2] means that by default, RDD will have 2 partitions. As one partition corresponds to one task, the spark job will have two tasks.

2. via External Datasets
Text file RDDs can be created using SparkContext’s textFile method. This method takes an URI for the file (either a local path on the machine, or a hdfs://, s3a://, etc URI) and reads it as a collection of lines.
```
In [2]: sc.textFile("file:///home/yanchao_murong/data/hello.txt").collect()                                                                                                                                                                           
Out[2]: ['hello\tworld\twelcome', 'hello\twelcome']
```

## RDD operations
> RDDs support two types of operations: **transformations**, which create a new dataset from an existing one, and **actions**, which return a value to the driver program after running a computation on the dataset. 

> All transformations in Spark are lazy, in that they do not compute their results right away.Instead, they just remember the transformations applied to some base dataset (e.g. a file). The transformations are only computed when an action requires a result to be returned to the driver program.  

rdda -> transformation -> rddb 
lazy(*****) => rdda.map().filter()......collect

##### key concepts
1. transformations in Spark are lazy, nothing actually happens until an action is called
2. action triggers the computation 
3. action returns values to driver or writes data to external storage

##### key operators
- transformation
map, flatmap, filter, groupByKey, reduceByKey, sortByKey, join ...
- action
collect, count, max, min, sum, take, reduce, saveAsTextFile, foreach ...


## RDD programming examples
#### word count
- input 1/n file directory suffix
```
hello,spark
hello,hadoop
hello,welcome
```
- steps
  - split each line to singel words ==> flatmap
  - word => (word, 1) ==> map
  - sum of counts of same word ==> reduceByKey
  
```
counts = sc.textFile(sys.argv[1]) \
            .flatMap(lambda line: line.split(",")) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(lambda a, b: a + b)

output = counts.collect()

for (word, count) in output:
    print("%s: %i" % (word, count))

```

#### topN (5 most visited videos)
- input 1/n file directory suffix
```
2017-05-11 14:09:14	http://www.xxxx.com/video/4500	304	218.75.35.226
2017-05-11 15:25:05	http://www.xxxx.com/video/14623	69	202.96.134.13
```

- steps
  - filter only video visit => filter
  - log => (videoid, 1) => map
  - sum of counts of same video visits => reduceByKey
  - (videoid, count) => (count => videoid) => map
  - sort desc => sortByKey
  - (count => videoid) => (videoid, count)  => map

```
arr = sc.textFile(sys.argv[1]) \
            .filter(lambda line: line.split("\t")[1].split("/")[3] == "video") \
            .map(lambda line: (line.split("\t")[1].split("/")[4], 1)) \
            .reduceByKey(lambda a, b: a + b) \
            .map(lambda x: (x[1], x[0])) \
            .sortByKey(False) \
            .map(lambda x: (x[1], x[0])) \
            .take(5)
for (id, counts) in arr:
    print("%s: %i" % (id, counts))
```