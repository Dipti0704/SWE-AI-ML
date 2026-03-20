# Data Engineering – Spark Streaming (Structured Streaming)
# From Micro-Batch Model to DataFrame-Based Streaming

## 1. The Story Continues: From DStreams to DataFrames

In the previous class, we understood how Spark Streaming works using **DStreams**.

We saw:
- Streaming = continuous data flow
- Spark breaks streaming data into **micro-batches**
- Each micro-batch = one RDD
- DStream = sequence of RDDs

Now the problem:

DStreams are based on **RDDs**, which:
- Are low-level
- Harder to work with
- Do not support advanced optimizations
- Do not integrate well with SQL

To solve this, Spark introduced:

```

Structured Streaming (DataFrame-based streaming)

```

This is the modern and recommended way of doing streaming in Spark.

---

# 2. Core Idea: Micro-Batch Model

Spark Streaming uses a **micro-batch processing model**.

Instead of processing data row by row:

- Data is collected for a small duration (e.g., 5 seconds)
- Then processed as a batch

Example:



Time 0–5 sec   → Batch 1
Time 5–10 sec  → Batch 2
Time 10–15 sec → Batch 3


Each batch is processed independently.

---

## Trade-off: Cost vs Latency

When defining batch duration:

```

ssc = StreamingContext(sc, 10)

````

You must balance:

- **Smaller batch duration**
  - Lower latency
  - Higher cost (more frequent jobs)

- **Larger batch duration**
  - Higher latency
  - Lower cost

This is a key design decision in real systems.

---

# 3. Streaming Lifecycle

Streaming application lifecycle:

### Start Streaming

```python
ssc.start()
````

### Stop Streaming

```python
ssc.stop(stopSparkContext=True, stopGracefully=True)


* `stopGracefully=True` ensures all running batches complete before stopping

```

---

# 4. End-to-End Streaming Flow

```
Live Data Sources
        ↓
Data Ingestion (HDFS / Kafka)
        ↓
Receiver
        ↓
Micro-batch Creation
        ↓
Transformations
        ↓
Output Sink
        ↓
Checkpoint (for fault tolerance)

``` 

---

# 5. Internal Working of Spark Streaming

Spark converts streaming data into micro-batches using the following components:

## 1. Receiver

- Continuously reads data from sources
- Never stops unless system fails
- Sources:
  - HDFS
  - Kafka
  - Socket streams

---

## 2. Block Generation

- Data is divided into blocks within each batch interval
- Blocks are distributed across cluster nodes

---

## 3. RDD Creation

- Each block is converted into an RDD
- Each batch = collection of RDDs

---

## 4. Transformation

- Transformations (map, filter, reduce) are applied to each RDD
- Execution is distributed across executors

---

# 6. Window Operations

Streaming often requires **time-based analytics**.

Example:

Every 5 seconds we get:

```

Batch 1 → 3 orders
Batch 2 → 4 orders
Batch 3 → 6 orders

```

Question:

How to calculate total orders in last 1 minute?

---

## Solution: Window Functions

Example:

```python
windowed_wh_revenue = wh_Revenue.reduceByKeyAndWindow(
    lambda a, b: a + b,
    lambda a, b: a - b,
    windowDuration=60,
    slideDuration=20,
)
````

---

### Explanation

* `windowDuration = 60`

  * Looks at last 60 seconds of data
  * Combines multiple batches

* `slideDuration = 20`

  * Updates result every 20 seconds

---

## Types of Windows

### Tumbling Window

* Fixed, non-overlapping windows
* Example: every 1 minute

---

### Sliding Window

* Overlapping windows
* Example:

  * Window = 60 sec
  * Slide = 20 sec

---

# 7. Late Data Problem

In real systems, data may arrive late.

Two important concepts:

## Event Time

* When event actually happened
* Example: order placed at 10:00:00

---

## Processing Time

* When Spark processes the data
* Example: file arrives at 10:00:30

---

## Late Data Example

```
Event Time      = 10:00:00
Processing Time = 10:00:45
``` 

This is **late data**.

---

# 8. Handling Late Data – Watermark

Spark uses **watermarking** to handle late data.

Example:

- Window size = 10 minutes
- Watermark = 5 minutes

Meaning:

- Spark waits 5 extra minutes for late data
- After that, window is finalized

Late data beyond watermark is:

```

Discarded

```


---

# 9. Transition to Structured Streaming

Instead of using DStreams, we now use:

```

DataFrame-based Streaming

````

This is called **Structured Streaming**.

---

# 10. Reading Streaming Data

Example:

```

order_stream_df = spark.readStream \
    .format("csv") \
    .option("header", "true") \
    .schema(order_schema) \
    .load("/user/scltnrlab01/amazon_orders")

```

```

---

## Key Points

* `readStream` creates a **streaming DataFrame**
* Data is treated as an **unbounded table**
* New data keeps getting appended

---

# 11. Schema Definition

Structured Streaming requires schema.

Example:

```python
order_schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("category", StringType(), True),
    StructField("order_amount", DoubleType(), True),
    StructField("status", StringType(), True),
    StructField("event_timestamp", StringType(), True),
])
```

---

# 12. Event Time Column

Convert timestamp to proper type:

```python
order_stream_df = order_stream_df.withColumn(
    "event_time",
    col("event_timestamp").cast(TimestampType())
)
```

---

# 13. Transformations on Streaming DataFrame

Example:

```python
avg_revenue_df = order_stream_df \
    .filter(col("status") == "DELIVERED") \
    .groupBy("category") \
    .agg(avg("order_amount").alias("avg_order_value"))
```

---

# 14. Execution Model (Very Important)

When using DataFrames:

1. Spark builds a **logical plan**
2. Catalyst optimizer optimizes it
3. Physical plan (DAG) is created
4. Same DAG is reused for every micro-batch

---

# 15. Trigger Mechanism

```
Trigger fires every N seconds
```

At each trigger:

* New Spark job is created
* Only new data is processed

---

# 16. Writing Streaming Output

Example:

```python
query = avg_revenue_df.writeStream \
    .format("console") \
    .outputMode("complete") \
    .queryName("architecture_demo") \
    .start()
```

---

## Output Modes

### Complete Mode

* Outputs entire result table every time

### Append Mode

* Outputs only new rows

### Update Mode

* Outputs only changed rows

---

# 17. Final Structured Streaming Pipeline

```
Data Source (HDFS / Kafka)
        ↓
readStream (DataFrame)
        ↓
Logical Plan
        ↓
Catalyst Optimization
        ↓
Physical Plan (DAG)
        ↓
Micro-batch Execution
        ↓
writeStream (Sink)
        ↓
Console / Parquet / Kafka


```

---

# 18. Key Takeaways

- Spark uses **micro-batch model** for streaming
- DStreams are older approach (RDD-based)
- Structured Streaming uses **DataFrames**
- Catalyst optimizer improves performance
- Watermark handles late data
- Window functions enable time-based analytics
- Streaming pipelines run continuously

Structured Streaming is the **recommended approach in modern data engineering systems**.





-----------------------
--------------------------------
---------------------------------------------

spark uses micro batch evaluation model.

now we will understand how to use dataframes for streaming.

when we are creating the spark context we have to   spcify the duration in streamingCOntext

ssc = StreamingContext(sc,10)

we have to find the balence between cost and latency .

when we have to  start we do ssc.start()
but when we have to stop we do ssc.stop(StopSparkContext= True, stopGracefully=True)

the data arrives from the live sources, it is received at HDFS after that it converts into microbatches then you do transformation and finally  we do Sink/Output 
and for fault tolerance we us checkpoints .

How spark Converts Streaming data into mini batches 

the first main component it has is the receiver and this recerivers main wrok is to ensure and never stop and continueosly read the data from the file or the source.

the it goes to block generation
data is colledcted in the batch interval is divided into blocks distrubutred across the cluster.

each block will create a rdd for its processing .
then comes transformation , this is applied to each of the rdd of the particular batch. 

------

window operations : rolling revenue analytics
like in every 5 secs we are creating a file and 
we have  
  5 sec  -> 3 orders
  5 secs -> 4 orders
  5 secs -> 6 orders

  now how to find the total number of orders in one minute ?

for this we will use window function FOO

windowed_wh_revenue = wh_Revenue.reduceByKeyAndWindow(
    lambda a, b: a+b,
    lambda a , b: a - b,
    windowDuration= 60
    slideDuration=20,

)

windowduation 60 seconds is combining data from last 60 seconds (12 RDDs * 5s each)
slide duration is like restrating the process after a specified time or produce the updated result after every 20 secs 

new conceprt

LATE DATA HANDLING
there are two times
1. event time  : when the order was placed (timestamp in the csv data)

2. processig time : when the order file actually arrvies at the spark.

for every proceesing : for eg : the order is placed at 10 am and it arrvies at 10:00:30 am
if the files arrives at 10 am i will process at 10 am 30 secs
but it any file arrives at 10:00:45 am this will be called late data
now it will go to the next trail.. 10:01:00 am 

but we have to count that 10:00:45 but the processing time of that is 10:01:00 
so for this we use something called as Watermark()

for true event-time late data handling => use structured streaming with watermark()

but if a data came t 10 : 58 am and it reaaly got loraded  at 11. 15  but at 11 10 we have run the processing so for this we givr a 5 min water mark .

if a system uses a 10 minute window and a 5 minute watermark
spark waits 5 extra minutes for late records
after that old window state is declared 
late records beyong the watermark are discarded



again for this rollign window we use somethign called as 
tumbling window and sliding window


--------------------


now we will see how to use dataframes for streamings

streamingPDF = spark.readStream \
        .format("csv")
        .option("header", "true")
        .schema(order_Schema)
        .load("amazon_structured_stream")


for readming we are doing read stream ..
for writing we are doing write stream



readstream builds streaming DataFrame in an unbounded table

then the catalyst creates one logical plan

it will also try to optimize is and once it optimizes it .. physical plan get creats  this is where the DAG si also created .
that DAG will be used by every micro batch that is created

trigger fires (every N seconds) -> new spark job
only new rows from latest files are proceseed
output wrttine to sink console/parquet


