---
title: Spark Core - Intro
date:  2019-08-15 12:00:00
tags:
- spark
---

> An introduction to spark core: including its key concepts, architecture, cache system, lineage mecanism, dependency mecanism and tuning techinques.

### Architecture
![spark-cluster-overview](/img/2019/8/spark-cluster-overview-8c86832f8e3a4d2d8a8eb3df38f79999.png)
- Each application gets its own executor processes and run tasks in multiple threads
  - 1 executor = 1 process = 1 jvm 
  - 1 task = 1 thread of the executor process
  - good isolation 
  - data cannot be shared across different Spark applications (instances of SparkContext) without writing it to an external storage system
- Driver program must listen for and accept incoming connections from its executors throughout its lifetime 
  - driver needs to send code and tasks to executors
  - driver needs to get heartbeat information from executors, if one node failed, ask for another executor in another worker node to run the failed task
  
### Key concepts
- Application
  - 1 application spark = 1 driver + n executors
  - User program built on Spark. 
  - Consists of a driver program and executors on the cluster.
    - spark0402.py 
    - pyspark/spark-shell

- Driver program 
  - The process running the main() function of the application 
  - creating the SparkContext
	
- Cluster manager: An external service for acquiring resources on the cluster (e.g. standalone manager, Mesos, YARN)	
```
spark-submit --master local[2] 
spark-submit --master spark://hadoop000:7077/yarn
```

- Deploy mode: Distinguishes where the driver process runs. 
  - In "cluster" mode, the framework launches the driver inside of the cluster. 
    - Cluster is in charge of scheduling tasks on the cluster as well as allocating resources. 
  - In "client" mode, the submitter launches the driver outside of the cluster. 
    - Cluster is only in charge of allocating resources.

- Worker node: Any node that can run application code in the cluster
  - standalone: slave node slaves.conf
  - yarn: nodemanager

- Executor: A process launched for an application on a worker node
  - runs tasks 
  - keeps data in memory or disk storage across them
  - Each application has its own executors.
	
- Task: A unit of work that will be sent to one executor
	
- Stage: Each job gets divided into smaller sets of tasks called stages that depend on each other (similar to the map and reduce stages in MapReduce) 
  - the stage usually starts from taking data and ends with shuffle.

- Job: A parallel computation consisting of multiple tasks that 
gets spawned in response to a Spark action (e.g. save, collect); 
  - one action = one job


### Comparison with Hadoop
- Spark
  - Application = Driver + Executors 
  - 1 applcation = 0-N jobs
  - 1 action = 1 job = 1-N stages
  - 1 stage = 1-N tasks
  - 1 task = 1 thread
- Hadoop 
  - 1 MR = 1 job
  - 1 job = N Tasks
  - 1 Task = 1 process


### Spark Cache

Caching or persistence are optimisation techniques for (iterative and interactive) Spark computations. They help saving interim partial results so they can be reused in subsequent stages. These interim results as RDDs are thus kept in memory (default) or more solid storages like disk and/or replicated across nodes.

```
/**
   * Persist this RDD with the default storage level (`MEMORY_ONLY`).
   */
  def cache(): this.type = persist()

  /**
   * Set this RDD's storage level to persist its values across operations after the first time
   * it is computed. This can only be used to assign a new storage level if the RDD does not
   * have a storage level set yet. Local checkpointing is an exception.
   */
  def persist(newLevel: StorageLevel): this.type = {
    if (isLocallyCheckpointed) {
   persist(LocalRDDCheckpointData.transformStorageLevel(newLevel), allowOverride = true)
    } else {
      persist(newLevel, allowOverride = false)
    }
  }
```

Cache is recommended if one RDD could be reused in upcoming computations. Like transformation, cache operation is lazy, it won't be delivered to executors until there is an action operator while unpersist will be executed immediately.

### Spark lineage 
RDDs are fault tolerant as they track data lineage information to rebuild lost data automatically on failure. When a transformation(map or filter etc) is called, it is not executed by Spark immediately, instead a lineage is created for each transformation. A lineage will keep track of what all transformations has to be applied on that RDD, including the location from where it has to read the data.

### Spark Dependency 
- Narrow Dependency (pipeline-able): A parent partition will only be used by child partition for one time. (map, filter, union ...)

 - Wide dependency (shuffle): A parent partition will be used by child partition for multiple times. (groupByKey, reduceByKey, join with inputs not co-partitioned, repartition, coalesce, cogroup ...)

> Certain operations within Spark trigger an event known as the shuffle. The shuffle is Spark’s mechanism for re-distributing data so that it’s grouped differently across partitions. This typically involves copying data across executors and machines, making the shuffle a complex and costly operation.

### Tuning 
#### Data Serialization
  By default, Spark serializes objects using Java’s ObjectOutputStream framework. and can work with any class you create that implements java.io.Serializable. 

  Alternatively, Kryo is significantly faster and more compact than Java serialization (often as much as 10x), but does not support all Serializable types and requires you to register the classes you’ll use in the program in advance for best performance. Since Spark 2.0.0, we internally use Kryo serializer when shuffling RDDs with simple types, arrays of simple types, or string type.

  To register your own custom classes with Kryo, use the registerKryoClasses method.
```
val conf = new SparkConf().setMaster(...).setAppName(...)
conf.registerKryoClasses(Array(classOf[MyClass1], classOf[MyClass2]))
val sc = new SparkContext(conf)
```

#### Memory tuning

By default, Java objects are fast to access, but can easily consume a factor of 2-5x more space than the “raw” data inside their fields.

Memory usage in Spark largely falls under one of two categories: **execution** and **storage**. 
  - Execution memory: used for computation in shuffles, joins, sorts and aggregations
  - Storage memory: used for caching and propagating internal data across the cluster.

> When no execution memory is used, storage can acquire all the available memory and vice versa. Execution may evict storage if necessary, but only until total storage memory usage falls under a certain threshold (R).

#### Determining Memory Consumption
- The best way to size the amount of memory consumption a dataset will require is to create an RDD, put it into cache, and look at the “Storage” page in the web UI. 
- To estimate the memory consumption of a particular object, use SizeEstimator’s estimate method. This is useful for experimenting with different data layouts to trim memory usage, as well as determining the amount of space a broadcast variable will occupy on each executor heap.

#### Broadcasting Large Variables
Using the broadcast functionality available in SparkContext can greatly reduce the size of each serialized task, and the cost of launching a job over a cluster. If your tasks use any large object from the driver program inside of them (e.g. a static lookup table), consider turning it into a broadcast variable. In general tasks larger than about 20 KB are probably worth optimizing.

#### Data Locality
Data locality can have a major impact on the performance of Spark jobs. If data and the code that operates on it are together then computation tends to be fast. But if code and data are separated, one must move to the other. Typically it is faster to ship serialized code from place to place than a chunk of data because code size is much smaller than data. 

- PROCESS_LOCAL data is in the same JVM as the running code. This is the best locality possible
- NODE_LOCAL data is on the same node. Examples might be in HDFS on the same node, or in another executor on the same node. This is a little slower than PROCESS_LOCAL because the data has to travel between processes
- NO_PREF data is accessed equally quickly from anywhere and has no locality preference
- RACK_LOCAL data is on the same rack of servers. Data is on a different server on the same rack so needs to be sent over the network, typically through a single switch
- ANY data is elsewhere on the network and not in the same rack