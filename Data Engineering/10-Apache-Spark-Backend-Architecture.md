




# Data Engineering – Class 10 Notes
# Apache Spark Backend Architecture

## 1. Motivation for Spark

Consider a large e-commerce company such as Amazon.

During events such as:
- Prime Day
- Big Billion Days
- Black Friday

Millions of users interact with the platform simultaneously.

These interactions generate massive data:
- Product views
- Add-to-cart events
- Purchases
- Clickstream logs
- Search queries

The company wants to analyze this data quickly to make decisions such as:
- Which products are trending
- Inventory planning
- Price optimization
- Fraud detection
- Recommendation updates

If we rely on **MapReduce**, analysis may take **hours or even a full day** because MapReduce writes intermediate results to disk.

However, companies often need **near real-time analytics**. For this reason, modern data platforms use **Apache Spark**.

Spark provides **much faster execution by processing data in memory instead of repeatedly writing to disk**.

---

## 2. MapReduce Execution Model

To understand why Spark is faster, it is important to first understand how MapReduce processes data.

Typical MapReduce workflow:

1. Read data from HDFS
2. Perform MAP operations
3. Write intermediate output to disk (HDFS)
4. Perform SHUFFLE and SORT
5. Perform REDUCE operations
6. Write final result to HDFS

This means **multiple disk reads and writes occur during the execution of a single job**.

Disk I/O is slow compared to memory operations. Because of this, MapReduce jobs can take a long time to complete.

---

## 3. Spark Execution Model

Spark improves performance by performing most operations **in memory**.

Typical Spark workflow:

1. Read data from HDFS
2. Load data into memory
3. Perform transformations (map, filter, join, etc.)
4. Execute aggregations in memory
5. Perform shuffles when necessary
6. Write final result to HDFS

Because intermediate data is stored in memory rather than disk, Spark can be **10–100 times faster than MapReduce**.

---

## 4. Memory Requirement in Spark

Since Spark relies heavily on memory for processing:

- Nodes must have large RAM capacity
- Cluster infrastructure cost increases

Example:

If each node requires:
- 128 GB memory

And the cluster has:
- 2000 nodes

Total memory used:

```

128 GB × 2000 = 256 TB

````

This high memory requirement is the trade-off for faster processing.

---

## 5. Single Point of Failure Problem

If Spark stores intermediate data in memory and a node crashes, data could be lost.

Spark solves this using a concept called **RDD (Resilient Distributed Dataset)**.

RDD provides:
- Fault tolerance
- Data recovery
- Distributed storage

---

## 6. Why Companies Are Moving from MapReduce to Spark

Several limitations of MapReduce led to the adoption of Spark.

### Disk-based processing

MapReduce writes intermediate outputs to disk multiple times.

This significantly slows down processing.

### Complexity of MapReduce Code

Writing MapReduce programs in Java is complicated.

Developers must manually implement:
- Mapper logic
- Reducer logic
- Data partitioning
- Optimization

### Structured vs Unstructured Data

Hive simplifies querying structured data using SQL.

But when dealing with unstructured data, developers often had to write custom MapReduce code.

Spark simplifies this by providing APIs in multiple languages.

---

## 7. Spark Programming Languages

Spark supports multiple programming languages:

- Python (PySpark)
- Scala
- Java
- R

Because PySpark uses Python syntax, it is easier for data engineers and data scientists to write distributed programs.

---

## 8. Spark Execution Planning

Spark internally creates a **DAG (Directed Acyclic Graph)**.

DAG represents the execution plan of the job.

A DAG ensures that:
- Tasks are executed in a logical order
- Dependencies are maintained
- Parallel execution is optimized

Spark automatically generates the execution plan based on the transformations applied in the code.

---

## 9. Resource Management with YARN

Spark does not manage cluster resources directly.

Instead, it relies on cluster managers such as:

- YARN
- Kubernetes
- Standalone cluster manager

In Hadoop environments, Spark usually runs on **YARN**.

YARN handles:
- Resource allocation
- Node management
- Failure handling

Data is still stored in **HDFS**, so Spark reads data from HDFS through YARN.

---

## 10. Spark Architecture Overview

Spark architecture consists of the following main components:

1. Driver Program
2. Spark Context / Spark Session
3. Cluster Manager
4. Worker Nodes
5. Executors

---

## 11. Driver Program

The **Driver Program** is the central coordinator of a Spark application.

It acts like the **project manager** of the Spark job.

Responsibilities of the driver:

- Executes the user program
- Creates the Spark session
- Builds the logical execution plan
- Optimizes the query plan
- Generates the physical execution DAG
- Distributes tasks to executors
- Monitors task execution
- Handles failure recovery
- Collects results from executors

Without the driver program, Spark applications cannot run.

---

## 12. Spark Context / Spark Session

Spark applications begin by creating a **SparkSession**.

SparkSession acts as the entry point for all Spark operations.

Example:

```python
from pyspark.sql import SparkSession
````

SparkSession is responsible for:

* Connecting the driver to the cluster manager
* Managing distributed execution
* Handling resource allocation
* Coordinating Spark components

---

## 13. Example Spark Configuration

Example configuration used in large production environments:

```python
spark = SparkSession.builder \
   .appName("Walmart-BlackFriday-Analytics") \
   .master("yarn") \
   .config("spark.executor.instances", "2000") \
   .config("spark.executor.cores", "32") \
   .config("spark.executor.memory", "128g") \
   .config("spark.executor.memoryOverhead", "32g") \
   .config("spark.driver.memory", "64g") \
   .config("spark.driver.cores", "16") \
   .config("spark.sql.adaptive.enabled", "true") \
   .config("spark.sql.adaptive.skewJoin.enabled", "true") \
   .config("spark.sql.shuffle.partitions", "40000") \
   .config("spark.dynamicAllocation.enabled", "false") \
   .config("spark.network.timeout", "600s") \
   .config("spark.sql.parquet.compression.codec", "snappy") \
   .enableHiveSupport() \
   .getOrCreate()
```

---

## 14. Understanding Spark Configuration Parameters

### Application Name

```
.appName("Walmart-BlackFriday-Analytics")
```

Identifies the Spark job in cluster monitoring tools.

---

### Cluster Manager

```
.master("yarn")
```

Specifies that YARN will manage cluster resources.

---

### Executor Instances

```
spark.executor.instances = 2000
```

Requests 2000 executor nodes.

If the cluster cannot provide these resources, the job may be rejected.

---

### Executor Cores

```
spark.executor.cores = 32
```

Each executor will use 32 CPU cores.

---

### Executor Memory

```
spark.executor.memory = 128g
```

Each executor has 128 GB memory.

Total memory used:

```
128 GB × 2000 executors = 256 TB
```

---

### Memory Overhead

```
spark.executor.memoryOverhead = 32g
```

Extra memory used for:

* Network operations
* Shuffle operations
* Internal processing

---

### Adaptive Query Execution

```
spark.sql.adaptive.enabled = true
```

Allows Spark to dynamically optimize queries during runtime.

Example improvements include:

* Better join strategies
* Handling skewed data

---

### Shuffle Partitions

```
spark.sql.shuffle.partitions = 40000
```

Defines how many partitions are created during shuffle operations.

More partitions allow greater parallelism.

---

### Hive Support

```
.enableHiveSupport()
```

Allows Spark to read and write Hive tables.

---

## 15. Cluster Manager Components

In a YARN environment, resource management involves:

### Resource Manager (RM)

Responsibilities:

* Receives resource requests
* Allocates cluster resources
* Schedules jobs

### Node Manager (NM)

Responsibilities:

* Runs on each cluster node
* Launches executors
* Monitors resource usage
* Kills processes if memory limits are exceeded

---

## 16. Executors

Executors are processes running on worker nodes.

Responsibilities:

* Receive tasks from the driver
* Load data from HDFS
* Perform transformations
* Shuffle data across nodes
* Return results to the driver

Executors enable **parallel data processing**.

---

## 17. Directed Acyclic Graph (DAG)

Spark represents job execution as a DAG.

Example Spark query:

```
df.filter(amount > 100)
  .groupBy(category)
  .sum(amount)
  .orderBy(sum(amount))
```

Spark converts this into a DAG consisting of stages:

1. Read data
2. Filter records
3. Group by category
4. Aggregate values
5. Sort results
6. Write output

Each stage is executed in parallel across executors.

---

## 18. Spark Transformations

Transformations define how data is processed.

There are two types:

### Narrow Transformations

A narrow transformation does not require data movement across partitions.

Examples:

* map()
* filter()

Data stays within the same partition.

These operations are fast because no network communication occurs.

---

### Wide Transformations

Wide transformations require data movement between nodes.

Examples:

* groupBy()
* join()
* orderBy()

During these operations, Spark performs a **shuffle**.

Shuffle is expensive because it involves network transfer.

---

## 19. RDD (Resilient Distributed Dataset)

RDD is the fundamental data structure in Spark.

RDD properties:

### Resilient

RDDs are fault-tolerant.

They track lineage, meaning Spark knows how the data was created.

If a partition fails, Spark recomputes it using the lineage.

This avoids the need for data replication.

---

### Distributed

Data is divided into partitions and distributed across cluster nodes.

Each partition can be processed independently.

---

### Dataset

RDD can contain:

* Structured data
* Semi-structured data
* Unstructured data

RDDs are immutable, meaning they cannot be modified once created.

Instead, new RDDs are created through transformations.

---

## 20. Summary

Spark is a distributed computing engine designed for fast data processing.

Key advantages over MapReduce include:

* In-memory processing
* DAG-based execution
* Simplified programming model
* Support for multiple languages
* Faster execution for analytics and machine learning workloads

Spark architecture consists of:

* Driver Program
* Spark Session
* Cluster Manager
* Executors

RDD provides fault tolerance and distributed processing capabilities, making Spark a powerful platform for large-scale data engineering workloads.

```
```

-----
-----
----------
-----
------
----
Sparks backend

imagine you working with amazon e commerce
they have big billion days / prime day

if i rely on mr technique => for batch processing  => we will get analysis in 24 hrs
they want the analysis in near realtime .for this we will use spark.
spark brings high speed of execution.


how map reduce work 
read the data from hdfs => then MAP => then write all the mappers to HDFS. Disk write => reduce => again write to the hdfs.


spark 
read hdfs => disk read  => post which MAP => which is stored in memory in disk => incase you do a join => in memory any aggregation or sort in memory 


after all this done... the output is written back to the hdfs.
we need a heavy memory for spark. so computational expence increase...
and since we just write the output to hdfs so the speed drastically increases.


but if  the inmemory node fails... then it will lead to single point of failure.

but sparks avoids this  by RDD.
RDD => Resilent Distributed Dataset


why everycompany is moving to spark 
1. in disk write
2. writing the MR code => complex to write...
while dealing with structed data hive wirtes it.. but what if the data is unstructed data.. we have to wrtie the mr code .
the person who writes the mr code also have to take care about the optimization.


whereas in my spark architecture => its very similar to python  code 
i dont want to write a very long code for structed or unstructerd data.
spark runs on python .
spark operates on DAG .. it creates its own plan of action.

for Resource management we will use yarn only and yarn read from hdfs ... this part will remain the same.

![alt text](Images\10-image-01.png)

so the first we have is driver parogram which has the spark context.

consider the driver program as somebody who controls the plane of spark architecture.
main responsibility is to run  the user code and orchestrates the distributed execution. 
you cannot run the spark without initializing the driver program.
what does it does ?
main responsibility =>
execute the pyspark code
creates particular session called as spark context/ spark session.
from the code => it will create the logical plan of exectuion
now after that it will try to optimize the logical plan and this is called as catalyst optimizer.
then it will generate the physical plan of execution (DAG).
once the plan is finalizaed it will schedule the task across executors .
once the workflow starts to run then it will monitor the execution. 
ensure it restarts and it will try to handle the failures.
collect the result from the executor and report the metrics output.
it acts like a manager .


now we have something called as spark context 
when we write spark code we always write configuration code  setting and this is called spark context. 
main repsonsibility =>
1. connect driver and the cluster mean connect the driver to the yarn
2. spark context acts as a entrypoint for all spark exectution.
3. to manage the cluster and the resources once it gets from the yarn.


in jupyter we write

        from pyspark.sql import SparkSession
    
then we do the configuation setting

    spark = SparkSession.builder \
   .appName("Walmart-BlackFriday-Analytics") \
   .master("yarn") \
   .config("spark.executor.instances", "2000") \
   .config("spark.executor.cores", "32") \
   .config("spark.executor.memory", "128g") \
   .config("spark.executor.memoryOverhead", "32g") \
   .config("spark.driver.memory", "64g") \
   .config("spark.driver.cores", "16") \
   .config("spark.sql.adaptive.enabled", "true") \
   .config("spark.sql.adaptive.skewJoin.enabled", "true") \
   .config("spark.sql.shuffle.partitions", "40000") \
   .config("spark.dynamicAllocation.enabled", "false") \
   .config("spark.network.timeout", "600s") \
   .config("spark.sql.parquet.compression.codec", "snappy") \
   .enableHiveSupport() \
   .getOrCreate()


1. session => name = appname
2. resources it goes to yarn
3. configure executor  => executor.intances = 2000 
we are requesting 2000 nodes... if yarn dont have 2000 nodes it will get rejected.
4. then we are setting up the core which are asking 32 cores.
5. memory => 128 gb per node  * 2000 =  256 tb
6. you have memory overhead => we need memory for network transfers so this is 32 gb
7. since  we do parallel processing => there can be overload on only one node 
we use spark3.0 i can use something called as adaptive query exectuion.
8. partitiion = 400000 .. whateever the data comes we aredividing it into 40000 partitions.
9. enable hive.
10. read.parquet 
11. if it is not avaiable create a session . 

see inside spark alos..
map => shuffle => reduce
 but how many shuffle at once ... this is decided by the no. of partition. 


cluster manager => recives resource request 
allocate  => node manager
monitor health
failure.

RM => resource managers recives the request, allocates 
NM => report the available resources and launch the executors and will also monitor the node if its working fine or not and if the memeory exceeds it will kill the process.

finally comes nodes => executor
here extution of parallel processing will happen
so..
executors will recive the task from the driver program.. because its the project mnager here.
it will load the data to the memory from hdfs , 
it will execute transformation .
shuffle the data to other exector 
results will go t the driver .
..

we are doing heavy paralllel processing and DAG is the main component which help us does this .
df. filter (amount > 100) 
        .groupby(category)/
        .sum(amount)/
        .order by (sum(amount))

the dag will read the file ... filter amount > 100 then group by cateogry.. then aggregate  sum amount then order by then wrtie the result.


in spark we have two kinds of transformation
1. narrow transformation
2. wide transformation
 narrow transformation means the data doesnt get shuffle then it is this.
 when you are doing filtering then the data gets shuffle from one node to the other node ? the answer is no.. so this will be narrow
filter(), mapped()

in wide transformation the data will get shuffled . 
join() , group by , order by 


spark has its own preference 
=> RDD (Resilient Distributed Dataset)

resilient means it has the property if fault tolerance , which it executes via linage tracking .
form wehre the data comes from it keeps the track. and will quickly restart it.
partition => recompute
here no need to replicate the data.

D => distrutubed that means the data is split into parttion 
process data in parallel. 

D => dataset => structred or unstructred  and uses the datasets as object and they are inmutable .

how it enables the property will study in next class.

