

# Introduction to Hive, MapReduce & YARN Architecture (07)

---

# 1️⃣ Why Do We Need Distributed Processing?

## 🔹 Problem: Processing on a Single Machine

If data processing happens on only **one machine**, we face major problems:

### ❌ Drawbacks

1. **Slow Speed**

   * Large datasets take hours or days.
   * CPU and RAM are limited.

2. **Single Point of Failure (SPOF)**

   * If the system crashes → Entire job fails.
   * No redundancy.

3. **Restart Issue**

   * If failure occurs → Job restarts from beginning.
   * No fault tolerance.

---

# 2️⃣ Solution: Parallel Processing

To solve these issues, we need:

* Distributed storage
* Distributed processing
* Fault tolerance
* Load balancing

---

# 3️⃣ Birth of MapReduce (2004 – Google)

In 2004, Google introduced:

> **MapReduce Programming Model**

Core idea:

> Divide and Conquer

Instead of:

* One machine processing entire data

We:

* Divide data into blocks
* Process blocks in parallel
* Combine results

---

# 4️⃣ What is MapReduce?

MapReduce has 2 major phases:

## 🔹 1. Map Phase

* Processes each data block independently.
* Produces intermediate key-value pairs.

## 🔹 2. Reduce Phase

* Combines results by key.
* Produces final output.

---

# 5️⃣ Example 1: Count Orders by Category

### Scenario:

* 5 Petabytes of data
* HDFS block size = 128MB

### Calculation:

```
5 PB ≈ 40,000 blocks
```

So:

* 40,000 map tasks
* Each block processed independently

---

# 6️⃣ Example 2: Product Pairing (Amazon Orders)

File location in HDFS:

```
/amazon/orders/2026-02-01
```

Suppose 3 million orders
Total rows = 9 million

---

## 🔹 Map Function Example

```python
def map(linenr, order_line):
    parts = order_line.split(',')
    order_id = parts[0]
    product_id = parts[1]
    sales = parts[2]

    return (order_id, product_id)
```

### Input:

```
1, p1, 100
```

### Output:

```
(1, p1)
```

This is the **mapping phase**.

---

# 7️⃣ Shuffle + Sort Phase

Between Map and Reduce:

1. Data is grouped by key.
2. All identical keys go to same reducer.
3. Data is sorted.

This phase involves heavy network transfer.

---

# 8️⃣ Distributed Architecture Example

Imagine 3 Machines:

| Machine   | Data Node | Orders Stored |
| --------- | --------- | ------------- |
| Machine 1 | DN1       | 10,000        |
| Machine 2 | DN2       | 10,000        |
| Machine 3 | DN3       | 10,000        |

Each node:

* Has RAM (processing)
* Has local HDFS block

---

### Flow:

1. YARN assigns tasks.
2. Map tasks execute locally.
3. Intermediate results written.
4. Shuffle occurs.
5. Reduce phase starts.

---

# 9️⃣ Important: MapReduce is NOT Real-Time

❗ MapReduce is batch-oriented.

* Good for large historical data
* Not suitable for real-time processing
* Slow but scalable

---

# 🔟 Example: 90GB Dataset

Suppose:

* 900 million rows
* 90 GB
* 128MB block size

Blocks ≈ 700

---

## Ideal Scenario

* 700 map tasks
* 700 parallel containers

Time:

* ~10 minutes

Without parallelism:

* 48 hours

This shows power of distributed processing.

---

# 1️⃣1️⃣ Hadoop 2.0 – Data Locality

### What is Data Locality?

Instead of:

* Moving data to processing

We:

* Move processing to data

---

### Example:

Block 1 → Stored in DN1, DN23, DN47
Block 2 → Stored in DN2, DN1, DN24

System tries to:

* Run task on node where data exists

---

### Why Important?

If:

* Block size = 128MB
* Network speed = 100MB/s

Fetching from remote node:

* Takes ~2 seconds
* With overhead ~10 seconds

Multiply by 40,000 blocks → huge delay

Data locality reduces network cost drastically.

---

# 1️⃣2️⃣ Introduction to YARN

YARN = Yet Another Resource Negotiator

It manages:

* Cluster resources
* Task scheduling
* Monitoring

---

# 1️⃣3️⃣ YARN Architecture

## 🔹 Components

### 1️⃣ Resource Manager (RM)

Responsibilities:

* Allocate containers
* Schedule tasks
* Monitor cluster resources

---

### 2️⃣ Node Manager (NM)

* Runs on each machine
* Manages containers
* Reports health to RM

---

### 3️⃣ Application Master (AM)

* Created per job
* Coordinates map & reduce tasks
* Talks to Resource Manager

---

# 1️⃣4️⃣ Product Recommendation Example

### Input:

```
/orders/2026-01-02.csv
```

Size:

* 5 PB
* 40,000 blocks

---

## Step 1: Client Submits Job

Client → YARN
Says:

> I need to run a MapReduce job.

---

## Step 2: Resource Manager Allocates Container

Example:

* 2 CPU cores
* 4GB RAM

Allocated to:

* Node Manager v42

Node Manager launches:

* Application Master

---

## Step 3: Map Phase Planning

We have:

* 40,000 blocks
* So 40,000 map tasks needed

Application Master requests:

* 40,000 containers

Cluster Stats:

* Total cores: 320,000
* Requested: 40,000

Resource Manager:

* Approves

---

## Step 4: Data Local Assignment

Application Master:

* Suggests best node for each block

Example:

* Container 1 → Node Manager 5 (DN5)
* Container 2 → Node Manager 2 (DN4)

Map tasks run in parallel.

Each task:

* Reports status to Application Master
* Enables fault tolerance

---

# 1️⃣5️⃣ Shuffle Phase – Network Heavy

If:

* 40,000 map tasks
* Each produces 10 reduce partitions

Total intermediate files:

```
40,000 × 10 = 400,000 files
```

If each file = 100MB

Total transfer:

```
400,000 × 100MB
```

This is huge network movement.

YARN now allocates reduce containers.

---

# 1️⃣6️⃣ Reduce Phase Allocation

Application Master → Resource Manager:

Requests:

* 10,000 reduce containers
* Each with 2 CPU + 4GB RAM

If available:

* Approved
* Reduce tasks run in parallel

---

# 1️⃣7️⃣ Final Step

1. Reducers combine grouped data.
2. Final output written to HDFS.
3. Application Master reports job completion.
4. Containers released.

---

# 1️⃣8️⃣ Complete Execution Flow Summary

```
Client
   ↓
Resource Manager
   ↓
Application Master
   ↓
Map Containers (Parallel)
   ↓
Shuffle + Sort
   ↓
Reduce Containers (Parallel)
   ↓
Final Output in HDFS
```

---

# 1️⃣9️⃣ Why Hive Comes Into Picture?

Writing MapReduce in Java is complex.

Hive provides:

* SQL-like interface
* Converts SQL → MapReduce jobs
* Runs on Hadoop cluster

So instead of writing Java:

```sql
SELECT category, COUNT(*)
FROM orders
GROUP BY category;
```

Hive internally:

* Creates MapReduce plan
* Executes via YARN

---

---
---
---
















# Introduction to HIVE (07)

if the processing is happening on only one machine
drawbacks
1. speed is slow
2. problem of spof
3. restart issue (we have to start all the processing from the start itself)


solution is parallel processing
2004 google came with the idea of map reduce
and the ideation is processing can be in parallel
load balencing  can be handled and 
the failure can be avoided .


in simple words map reduce means divide and conqure
in this what 
map do is we process each piece of data independently 
and reduce combines results by key .


in example of find out the number of orders by each cateogires
i have 5 petabytes of data to process
so i will split  this in 40,000 blocks ... because 1 block has 128mb



and the 2nd example was product pairing and how will map reduce will actually work.
3 million order we have with 9 million rows and in hdfs  it is stored at /amazon/orders/2026-02-01


def map (linenr, orderline)
    part = order_line.split(',')
    order_id = part[0]
    product_id = part[1]


order_id, product_id, sales
=> 1, p1,100


return (orderid, product_id)


this whole is the map function we are doing here the mapping
and the second step in with we do map reduce is 
shuffle + sort


after shuffle and sort .





lets say we have a machine 1 
it has ram which do processing
and it has data node
this dn1 has data of the first 10,000 orders
dn2 node has data of another 10,0000 orders
dn3 node has data of another 10,0000 orders

yarn assigns the cluster



after all 3 nodes completed the task... 
we need another processor to transfer this data to process the data node.
ek baar write hoga .
once this is done.. that is when the reduce will come into action 



map reduce is not made for real time processing , its a slow process.



therfore we do parallel processing .. and the engine is called map reduce 
map reduces creates the plan ... it creates the execution for the plan.

imagine that we hav 900m data which is 90 gb around

hdfs has ... 
700 block


map phase 
now in an ideal condition how many task should be launchd in parallel which is 700 
each process is one block so all 700 block should have a processor(RAM) to execute task.

we will get this output in 10 mins ...
otherwise it would we 48 hours

 Hadoop 2.0 has the major advancement of data locality
 block 1 => DN1, DN23, DN47
 block 2 => DN2, DN1, DN24


with data locality the issue that will happen is 
a random processing server will be assigned to the block 
and then whennever it will fetch... it will fetch the data from each block
eg : 128 mb file with network speed of 100mb -> it will take 2 sec
usaully for every block it will take 10 secs .








how yarn actually work.
yarn architecture


YARN resource manager
=> we will allocate the containers 
=> it will schdeule the task
=> it will monitor the resources (important)
it works very closely with node manager 
and node manager has -> application manager . it will run map task and it will run reduce task as well
for this it will use 32 cpu

calculate product recommendation 
=> input -> order/2026-01-02.csv (5 petabites, 40000)

client will request the data  => it will write job => request the data
cleint will submit it to yarn  "i need to run a map reduce jib"

yarn will assign the container for app master
if the container that you can use should be of 2 CPU and 4GB Ram
it will assign this to the node maneger v42
this node manager will launch app master


so we ahve the 5 PB file and 40000 blocks
map -> 40000 map task (needed )
how many container will be requred for parallel processing  => 40,000
so the app master will request the resource manager that we need 40,000 containers
and what it will also do is, it will send the recommendation
DN -> 5 (block 1) => best

the Resource manager will allocate
total core  => 3,20,000
reques => 40,000 cores
Resource manager(RM) => approved

and then once this is done the the app master will now assign the container locally 
contianer 1 will work with nodemanager 5  dn 5
c2 will work will node manager 2 on dn 4

map task will happen parallel that is the execution will be parallel.
when these task are getting executed it will report to the app master... all status to app master 
to avaiod any failure this thing is done .


next step is shuffle + sort\
this is where the data is transfered between the container
if i have 40,00 map task that are running parallel
then i consider that each map task will give me 
1 map task results into 10 reduce file 
total transfer will happen 40,000*10 =  4 lakh 100mb  = 4 lakh*100mb transfer
we need a container for this.
now yarn will talk with apllication master that we need to do reduce containr allocation

so the application master will talk to Resourcr manager and say that i need 10000 containers w with 2cpu core and 4 gb ram .. if it is avaiable then it will be approved anf the allocaiton will happen

once the file is collected and stores that is where the  execution of reduce will take place
the containers will run parallel .

once the execution is complete we will combine the final results



