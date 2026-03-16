
# Data Engineering – Class Notes
# From Spark Processing to Streaming Architecture

## 1. Recap: What We Have Learned in PySpark

Before moving into streaming systems, it is important to understand what we already covered in PySpark. Most of our work until now focused on **batch data processing using Spark**.

In the previous classes we learned how to:

- Perform data processing using **PySpark DataFrames**
- Use different **aggregation functions** such as:
  - `sum`
  - `avg`
  - `count`
  - `min`
  - `max`
  - `stddev`
- Use **groupBy**, **orderBy**, and **reduceByKey**
- Perform **joins** between datasets

Spark supports multiple join types:

- Inner Join
- Left Join
- Right Join
- Full Join
- Left Semi Join
- Left Anti Join

We also learned about **broadcast joins**, which optimize joins when one dataset is much smaller than the other.

Other concepts covered:

- Creating **User Defined Functions (UDFs)**
- Applying custom logic to Spark DataFrames
- Data structuring using functions like **explode**

These operations allow Spark to transform and process large datasets efficiently.

However, all the processing done so far belongs to **batch processing**.

---

# 2. Batch Processing

In batch processing, data is collected and stored first, and then processed later.

Example workflow:

1. Data arrives from multiple sources.
2. Data is stored in HDFS or another storage system.
3. A Spark job runs periodically.
4. The processed results are stored in a data warehouse or reporting system.

Example architecture:

```

Data Sources → HDFS → Spark Processing → Output Storage → Reporting

```

In most production systems, Spark jobs are not executed manually.

Instead, they are scheduled using tools such as:

- Apache Airflow
- Oozie
- Cron jobs

Example batch pipeline:

1. Scheduler checks if new data is available.
2. If data exists, it triggers the Spark job.
3. Spark reads the data from HDFS.
4. Spark performs transformations and aggregations.
5. Output is stored in the destination system.

This approach works well when:

- Data is static
- Data arrives periodically
- Real-time updates are not required

Example use cases:

- Daily sales reports
- Weekly marketing reports
- Monthly financial reports

---

# 3. Limitations of Batch Processing

Batch processing has some limitations.

The main issue is **latency**.

Example:

If batch processing runs every 6 hours, then insights are available only after the batch job finishes.

For many modern applications this delay is unacceptable.

Examples where faster processing is needed:

- Fraud detection
- Real-time recommendations
- Inventory updates
- Live dashboards
- Online monitoring systems

This requirement leads to **streaming data processing**.

---

# 4. What is Streaming?

Streaming refers to processing data **continuously as it is generated**.

Unlike batch processing:

- Data does not arrive in fixed batches.
- Data is continuously flowing.

Key characteristics of streaming:

1. Data is processed immediately after arrival.
2. Data pipeline runs continuously.
3. Data streams are unbounded.

Example:

User activity on an e-commerce website:

```

User Click → Event Generated → Processing → Recommendation Update

```

This entire pipeline happens continuously.

Streaming pipelines typically have:

- A start time
- No defined stop time

---

# 5. When to Use Batch vs Streaming

Choosing between batch and streaming depends on the nature of the data.

### Use Batch Processing When

- Data is finite and static
- Processing can happen at intervals
- Real-time updates are not necessary

Examples:

- Historical analysis
- Monthly reports
- Data warehouse ETL pipelines

---

### Use Streaming When

- Data is continuous
- Data arrives in real time
- Immediate insights are required

Examples:

- Fraud detection
- Recommendation systems
- Real-time analytics
- AI/ML model updates

Many AI/ML systems require **real-time data updates**, which makes streaming extremely important.

---

# 6. Typical Data Sources in Modern Systems

Large organizations receive data from multiple sources.

Examples:

1. **RDBMS systems**
   - Customer databases
   - Transaction databases

2. **Clickstream data**
   - Website user behavior
   - Mobile app activity

3. **Third-party APIs**
   - External data providers
   - Payment systems
   - Logistics platforms

These sources generate massive amounts of data.

---

# 7. Data Ingestion into Hadoop Ecosystem

To move data from source systems into Hadoop, specialized tools are used.

Examples:

### Apache Sqoop

Used for transferring data between:

- RDBMS systems
- Hadoop ecosystem

Example:

```

MySQL → Sqoop → HDFS

```

---

### Apache Kafka

Used for collecting **streaming data** such as:

- Clickstream events
- Application logs
- IoT sensor data

Example:

```

Website Clicks → Kafka → HDFS

```

---

# 8. Traditional Streaming Architecture (Without Messaging Layer)

Initially, streaming pipelines looked like this:

```

Data Source → Spark Streaming → Output Storage

```

Example:

```

Amazon Orders → Spark Streaming → Analytics Dashboard

```

This architecture has a major weakness.

If Spark crashes or runs out of memory:

- Data being processed may be lost.

This creates a **single point of failure**.

---

# 9. Introducing an Intermediate Layer

To solve the reliability problem, an intermediate system is introduced.

New architecture:

```

Data Source → Messaging Layer → Spark Streaming → Output

```

The messaging layer acts as a **buffer between producers and consumers**.

Responsibilities of this layer:

1. Temporarily store incoming data
2. Track processed vs unprocessed data
3. Ensure reliable data delivery

Example scenario:

1. Customer places an order.
2. Order event goes to messaging layer.
3. Spark streaming reads from messaging layer.
4. Data is processed.

If Spark crashes:

- Data remains stored in the messaging system.
- Processing resumes later.

This prevents data loss.

---

# 10. Messaging Queue Responsibilities

Messaging queues provide two major functions.

### 1. Temporary Storage

Incoming data is stored temporarily before processing.

### 2. Coordination

They track:

- What data has arrived
- What data has been processed
- What data is still pending

This makes streaming systems **fault tolerant**.

---

# 11. Apache Kafka

Kafka is the most widely used messaging system in modern data architectures.

Kafka introduces the concept of **topics**.

A topic is a logical channel where messages are stored.

Example:

```

Topic: user_clicks
Topic: orders
Topic: payments

```

Kafka stores messages temporarily inside these topics.

---

# 12. Kafka Workflow

Streaming architecture with Kafka:

```

Data Source → Kafka Topic → Spark Streaming → Output

```

Processing flow:

1. Data arrives from source.
2. Kafka stores it in a topic.
3. Spark reads messages from Kafka.
4. Spark performs transformations.
5. Processed data is stored in destination.

Important point:

Spark processes **copies of data stored in Kafka**, not the original source.

---

# 13. Advantages of Kafka Layer

Adding Kafka improves streaming systems in several ways.

### Reliable Data Delivery

Messages are stored until consumers process them.

### Replay Capability

Data can be reprocessed if necessary.

### Multiple Consumers

Different systems can read the same data stream.

Example:

```

Kafka Topic → Spark Analytics
Kafka Topic → Fraud Detection
Kafka Topic → ML Model

```

---

# 14. Streaming Processing Engines

Several engines can process streaming data.

The choice depends on the use case.

---

## Spark Streaming

Spark Streaming processes data in **micro-batches**.

Example:

Instead of processing every 6 hours like batch processing:

- Spark Streaming processes every few seconds or minutes.

This is called **near real-time processing**.

Typical latency:

```

2–3 minutes

```

Spark streaming is popular because many organizations already use Spark for batch processing.

---

## Apache Flume

Flume is mainly used for **data ingestion**.

Its role is simply to move data from one system to another.

Example:

```

Application Logs → Flume → HDFS

```

Flume is not a processing engine.

---

## Apache Storm

Storm provides **very low latency streaming**.

Latency can be as low as:

```

~1 millisecond

```

Storm is suitable for applications requiring extremely fast processing.

---

## Apache Flink

Flink is designed as a **stream-first architecture**.

Unlike Spark streaming:

- Flink processes **events directly**
- No micro-batch model

Important features:

- Event-driven processing
- Stateful computations
- Very low latency

---

# 15. Messaging Framework Options

Besides Kafka, other messaging systems exist.

### Apache Kafka

Most widely used.

Features:

- High throughput
- Supports multiple consumers
- Retains data for replay
- Integrates with Spark and Flink

---

### Apache Pulsar

Designed for multi-tenant environments.

Useful when many independent applications generate events.

---

### RabbitMQ

Traditional message queue system used for application communication.

---

### AWS Kinesis

Managed streaming service provided by AWS.

Advantages:

- Fully managed
- Scalable

Disadvantages:

- Expensive for large-scale data pipelines

---

# 16. Final Streaming Architecture

A reliable streaming architecture typically looks like this:

```

Data Sources
↓
Data Ingestion (Kafka / Flume / Sqoop)
↓
Messaging Queue (Kafka Topics)
↓
Stream Processing (Spark / Flink / Storm)
↓
Output Storage
↓
Reporting / Analytics / ML Systems

```

This architecture ensures:

- Fault tolerance
- Data reliability
- Real-time insights
- Scalability for large data volumes







-----
----
----

Below is a **visual architecture style version of the same notes** designed for **VS Code Markdown preview**.
These diagrams help a lot in **remembering pipelines for interviews and real systems**.

You can paste this directly into your `.md` file.

---


# Data Engineering – Streaming Architecture (Visual Notes)

## 1. Recap: What We Did Before Streaming

Before learning streaming, we focused on **Spark batch processing**.

We learned how to perform:

- Aggregations (`sum`, `avg`, `count`, `min`, `max`)
- `groupBy`, `orderBy`, `reduceByKey`
- Joins  
  - inner join  
  - left join  
  - right join  
  - full join  
  - left semi join  
  - left anti join
- Broadcast joins (optimization)
- User Defined Functions (UDF)
- Data transformations (`explode`, column operations)

All these operations were done using **Spark DataFrames**.

However, these operations were performed on **data that already existed in storage**.

This type of processing is called **Batch Processing**.

---

# 2. Batch Processing Architecture

Batch processing means:

Data arrives → stored → processed later.

Example pipeline:

```

```
      +-------------------+
      |   Data Sources     |
      |--------------------|
      | RDBMS              |
      | Clickstream        |
      | Third-party APIs   |
      +---------+----------+
                |
                v
      +-------------------+
      |   Data Ingestion   |
      |--------------------|
      | Sqoop              |
      | Kafka              |
      +---------+----------+
                |
                v
      +-------------------+
      |      Storage       |
      |--------------------|
      | HDFS / Data Lake   |
      +---------+----------+
                |
                v
      +-------------------+
      |   Processing       |
      |--------------------|
      | Spark / MapReduce  |
      +---------+----------+
                |
                v
      +-------------------+
      |  Output Storage    |
      |--------------------|
      | Data Warehouse     |
      | CSV / Tables       |
      +---------+----------+
                |
                v
      +-------------------+
      | Reporting Layer    |
      |--------------------|
      | BI / Dashboards    |
      +-------------------+
```

```

Batch processing example:

- Daily sales reports
- Weekly marketing reports
- Monthly financial analytics

---

# 3. Limitation of Batch Processing

Batch processing has **high latency**.

Example:

If a batch job runs every **6 hours**, insights are delayed by 6 hours.

For modern systems this delay is unacceptable.

Examples needing faster insights:

- Fraud detection
- Live user analytics
- Recommendation systems
- Inventory updates
- Real-time monitoring

To solve this problem we use **Streaming Processing**.

---

# 4. What is Streaming?

Streaming means processing data **as soon as it is generated**.

Key properties:

- Data is continuously flowing
- Data pipeline never stops
- Data is processed immediately

Example event flow:

```

User Click → Event Generated → Stream Processing → Dashboard Update

```

Unlike batch processing, streaming pipelines are **unbounded**.

Meaning:

```

Start time exists
End time does NOT exist

```

---

# 5. Batch vs Streaming

| Feature | Batch Processing | Streaming |
|------|------|------|
| Data Type | Static | Continuous |
| Latency | Minutes / Hours | Seconds / Milliseconds |
| Execution | Scheduled | Continuous |
| Use Case | Historical analysis | Real-time analytics |

---

# 6. Traditional Streaming Architecture (Problem)

Initially streaming pipelines looked like this:

```

Data Source
|
v
Spark Streaming
|
v
Output Storage

```

Example:

```

Amazon Orders → Spark Streaming → Dashboard

```

Problem:

If Spark fails or runs out of memory, **data can be lost**.

This creates a **Single Point of Failure (SPOF)**.

---

# 7. Improved Streaming Architecture (With Messaging Layer)

To prevent data loss, a **Messaging Queue Layer** is introduced.

```

```
        +----------------+
        |  Data Sources  |
        +--------+-------+
                 |
                 v
        +----------------+
        | Messaging Layer|
        |----------------|
        | Kafka Topics   |
        +--------+-------+
                 |
                 v
        +----------------+
        | Stream Engine  |
        |----------------|
        | Spark Streaming|
        | Flink / Storm  |
        +--------+-------+
                 |
                 v
        +----------------+
        | Output Storage |
        |----------------|
        | HDFS / DW      |
        +--------+-------+
                 |
                 v
        +----------------+
        | Reporting / ML |
        +----------------+
```

```

---

# 8. Role of Messaging Queue

Messaging systems provide two important services.

## Storage

Incoming events are stored temporarily.

## Coordination

They track:

- Incoming data
- Processed data
- Pending data

This prevents data loss.

---

# 9. Kafka Concepts

Kafka organizes data into **Topics**.

Example:

```

Topic: user_clicks
Topic: orders
Topic: payments

```

Architecture:

```

Producers → Kafka Topics → Consumers

```

Example:

```

Website Events → Kafka → Spark Streaming

```

Spark processes **copies of events stored in Kafka**.

---

# 10. Kafka Buffer Layer

Kafka acts like a **buffer**.

```

Event Source
|
v
Kafka Topic
|
v
Temporary Storage Buffer
|
v
Spark Streaming Processing

```

If Spark fails:

Kafka still stores the events.

Processing resumes later.

---

# 11. Advantages of Messaging Layer

Adding Kafka improves reliability.

Benefits:

### Fault Tolerance

Data is not lost even if Spark crashes.

### Replay Capability

Data can be replayed if needed.

### Multiple Consumers

One topic can feed multiple systems.

Example:

```

Kafka Topic
|
+----> Spark Analytics
|
+----> Fraud Detection
|
+----> ML Model

```

---

# 12. Streaming Processing Engines

Several engines process streaming data.

---

## Spark Streaming

Spark streaming processes data as **micro-batches**.

Example:

```

Batch Job → every 6 hours
Spark Streaming → every few seconds

```

Latency:

```

2–3 minutes

```

Often called **Near Real Time (NRT)**.

---

## Apache Flume

Flume is used only for **data ingestion**.

Example:

```

Application Logs → Flume → HDFS

```

It does not process data.

---

## Apache Storm

Storm provides **very low latency processing**.

Latency:

```

~1 millisecond

```

Used for:

- financial trading systems
- fraud detection

---

## Apache Flink

Flink is designed as **stream-first architecture**.

Key features:

- Event-driven processing
- Stateful computations
- Very low latency
- True streaming

---

# 13. Messaging Systems Used in Streaming

## Apache Kafka

Most popular streaming platform.

Features:

- High throughput
- Supports multiple consumers
- Data retention for replay
- Integrates with Spark and Flink

---

## Apache Pulsar

Designed for multi-tenant systems.

Used when many independent services generate events.

---

## RabbitMQ

Traditional message queue used for application communication.

---

## AWS Kinesis

Managed streaming service on AWS.

Advantages:

- Fully managed
- Highly scalable

Disadvantage:

- Expensive

---

# 14. Complete Modern Streaming Architecture

```

```
          +-------------------+
          |    Data Sources    |
          |--------------------|
          | RDBMS              |
          | Clickstream        |
          | IoT Sensors        |
          +----------+---------+
                     |
                     v
          +-------------------+
          |   Data Ingestion   |
          |--------------------|
          | Kafka / Flume      |
          +----------+---------+
                     |
                     v
          +-------------------+
          | Messaging Queue    |
          |--------------------|
          | Kafka Topics       |
          +----------+---------+
                     |
                     v
          +-------------------+
          | Stream Processing  |
          |--------------------|
          | Spark / Flink      |
          | Storm              |
          +----------+---------+
                     |
                     v
          +-------------------+
          | Output Storage     |
          |--------------------|
          | HDFS / Data Lake   |
          | Data Warehouse     |
          +----------+---------+
                     |
                     v
          +-------------------+
          | Analytics / ML     |
          | Dashboards         |
          +-------------------+
```

```

---

# 15. Key Takeaways

- Batch processing works on stored datasets.
- Streaming processes data continuously.
- Messaging systems prevent data loss.
- Kafka is the most widely used messaging system.
- Spark Streaming uses micro-batch processing.
- Flink and Storm provide lower latency streaming.

Streaming systems are essential for **real-time analytics, recommendation systems, fraud detection, and modern ML pipelines**.


----
-----
-----

we have covered operations is pyspark
how does spark optimize its query
we learnt about different aggregated fucntions order by group by recude by 
we covered about joins  all the four joins that we know + left anti , left semi
we learnt about broadcast join 
we saw how to create udf and then apply 
for data structuring and data dtransformation we have  explode function 


now we will transition to streaming

when we dont get the data in batches or in hours... we get data in few seconds.. that is what the concept of streaming is .

previouslly we had batch processing and this was done using the processing engine of mr called spark.
mr is slowest, spark is faster than mr .

So lets say the data comes in hdfs the data gets dump here, from here we will creat a spark code we will be writing the data in the hdfs or data warehouse or csv file  , we wont be running this code manually 
we would add a scheduler and that scheduler will get the data , will check whether the data is available in the hdfs, run the spark code and upload it anywhere where the destination is .
this kind of processing where the data comes, resides , after that you do some processing and then upload it , it is happeing in the batches this is the batch processing .


streaming helps you update the data realtime . 
-> streaming process the data as it is genereted.
->streaming assumes that data is always in motion , means it is contineously flowing .
-> core idea about streaming is that data is never fully completed ..
this means the data pipeline though which the data is coming as soon as it is generated .. this pipeline wil have a start time but it wont have stop time .

the question arises that when we should do batch processing and when we should do streaming

so if the data is finite and static ... which process is better ? => batch processing 
if data is contineous and unbounded  use streaming
if we want the data to be avaiable after certain interval so batch processing
most of the ai/ml model prefer to use realtime data thats why we should know streaming .
in batch processing we process the data at once and then show the ouput .
in straming data is processed whenever the data arrives


we have data soruces
1. RDBMS data                 
2. clickstream data    --->---->------>--- hdfs 
3. 3rd party 


now we want pipelines frokm data sources to hdfs  
so for rdbms we use scoop for clickstream we use kafka streaming  


after this we have processing engine we have data processing we have two options here MR / spark 
then this is again stored to the desired location and then used for analysis or reporting .


now lets understand the streaming workflow

we have data sources  then the data will move from data soruces and we will do ingestion whch is data ingestion 
now the schedule of the ingestion will be realtime  and from here i will have a stream processing this parti will take care of all your transfomration . aggregations.joins   and then finally it will be adding to the outptu storage it can be anywhere and then is can be used for reporting .

the drawback of this ar citecutre is that we me miss on some data .
the point of failure happens when spark runs out of memeory , any streaming job that is processing so other information runs out , something happens to the streaming process na dit fails .

so we need a middlelayer in between who can keep track of what has come what has processed and what is still to be processed , if we have that layer in between who can keep the track of what has come , what has processed and what is still to be processed , if we have that layer in between , then whenever streaming process goes down  it knows from where to exactly start so the above architecture is currently leading to the single point of failure .

to solve this we need an intermediate layer , what we want from that layer is, it should recieve the data from the source contineously , 
it should store the data for short peroid of time  .
this will allow the streaming process to read the data .

so previously we had a system which is the source , it would directly go to spark streaming 
now insted of doing this, i will have a source system then i will have IL then it will be connected to the spark streaming . because pf this layer we will not have information loss , for eg :
on amazon the customer places the order , that information goes to the source system and then as soon as the order is placed, the spark will pick it up and process the data , but if the spark is busy there is a chance that the order will be lost
so now what happens is once the order is placed , this information is stored temporarily in the IL layer and then the spark will kind of process, so at anypoint if anything happens to the spark, IL layer has the backup and you can check the data from there. and if the IL layer goes down then we have logs . 
this particualr intermediate layer ensures that no data is lost if spark is down , this ensures that the source systems are protected from heavy read and write , this also ensures that spark does not overload itself .

this intermediaate layer will be further break down based on the roles and responsibilites , so that again we can achieve better fault tolerance . we can do this using something called messaging queue and this will divide the repsonsibilites into two part 1. will be the storage 2. coordination 
so inside IL one person will take care of storage and other will take coordination.

and the whole point is... at any point the data should no be lost .


we have the data sources and the data is in hdfs this is where you will have you IL layer or messaging queue layer ,
just imagine that it will be storing the data and it is also doing coordination part of it and then it will go to spark and then the output will be again stored to the desired location .


now this messaging queue ensures that the streaming is reliable 
2. the recovery is gracefull .
3. now because of these mq services the realtime analysis becomes trustful .

in this messaging quueue only the kafka comes into the picture.

now the question here arises is
how does it control the queue

just imagine that you have a data in hdfs say its 2024/march/01 and this is the csv file 
when we are creating a realtime streaming what i am expecting is instead of giving the file name that i want to process i am saying any data that keeps on coming in this particualr partition you need to process it .
now what happens is... this particular csv data lets say the data you are updating is the inventory data .. can this inventory data be used in multiple flow.
now if one file gets updated so all the flows should also get updated .

so if a new file comes and we need to process this , before it starts processing .. it will go to mq and we know mq has two things storage and coordination
now this mq will create something called as topics=> what are these topics ?
thsese topics are nothing but ...
once you have the topic you have somthing called buffer layer and this buffer will store the temperory data and this temporary storage will go .. this will go to streaming ... it is not processing this file , it is processing the copy of this file which the kafka will store .

so the streaming will first read the infromation from topics , then it will read the data from the buffer data and then finally it will perform its transformation , aggregations , join and then this will go to your output and then this will go to any reporting layer that you have .


now we will see the streaming services

this streaming services that we know 
we have different streaming options and we need to decide based on usecase which streaming service we have to use 
1. we have is spark streaming 
 it is very similar to the batch streaming that we know  but the difference is what does happen in the streaming part spark will treat the data as small chunk of batches like in batch processing it triggers after every  6 hrs in case of streaming it triggers after every 1min so instead of treating this as batch process it ensures that it is doing the same thing just at a very faster rates and the reason why it is popular is bause most of the legacy code were on spark and people dont want to move for streaming to different tool thats why they came with this concept .
disadvantage is the latency is about 2 to 3 mins this is more of a near real time nrt .


then the next tool that we can use in our streaming is the apache flume
this apache flume is used for only one thing which is transportation of data .
this has to be used only when you have to move you data from point A to ppoint b .
i have the source system and if i have to move this data to the hdfs so in this process i can use flume .
so it is mostly used to move when you need to move large amount of data from one source system to another source system..

the third option you have for streaming is apache storm.
it has very low latency (1ms).


then comes
Apache flink
this is streaming first architecture 
it does not relies on file or anything it relies on event  as soon as the event happens.. it strats processing so its trigger points are events 
it also has stateful computations 


spark streaming is used whenever we have to do some analytics kind of thing 
flume is used data ingestion
storm for realtime analysis

the messeging framework that is added to the streaming architecture 
1. apache kafka for massive data volumes , support mu.ltipleconsumer

2. retain data for replay
3.spark + flink


then they built somethign called as apache pulser
see the events are generated because of multiple tennants means where one event is triggering multiple event s.


then we have something called as rabbit mq
then we have aws kinesis very expensive 