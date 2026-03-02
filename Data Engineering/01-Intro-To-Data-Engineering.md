

--------------------------------------------

# Intro to Data Engineering  (01)

---

# 1️⃣ What is Data Engineering?

## 🔹 Basic Definition

**Data Engineering** is the field of designing, building, and maintaining systems that collect, store, transform, and deliver data in a usable format for analysis and decision-making.

Whenever a human interacts with a machine:

* Browses a website
* Clicks a product
* Makes a payment
* Watches a video
* Uses an app

➡️ **Data is generated.**

But raw data is:

* Unstructured
* Messy
* Incomplete
* Distributed across multiple systems

Data Engineering ensures:

> Raw data → Clean → Structured → Stored → Available for analytics

---

# 2️⃣ Data Flow Overview

```
Human Interaction → Data Generated → Storage Layer → ETL → Data Warehouse → Data Marts → BI Layer
```

---

# 3️⃣ Storage Layer vs Consumption Layer

## 🔹 Storage Layer

* Raw data is stored here
* May contain structured + unstructured data
* Not optimized for analytics
* Cheap storage

## 🔹 Consumption Layer

* Data is cleaned and structured
* Optimized for analytics and reporting
* Used by business, analysts, ML teams

---

# 4️⃣ ETL – Extract, Transform, Load

ETL is the backbone of Data Engineering.

## 🔹 1. Extract

* Collect data from multiple sources:

  * Databases
  * APIs
  * Logs
  * Applications
  * Third-party systems

Example:

* Website clicks
* Payment records
* ERP systems
* CRM data

---

## 🔹 2. Transform

Raw data may be:

* Missing values
* In different formats
* Duplicated
* Unstructured

Transformation includes:

* Cleaning
* Removing duplicates
* Formatting dates
* Aggregations
* Converting unstructured → structured
* Business logic application

Example:

* Convert "12-02-25" → proper timestamp
* Remove null values
* Join multiple tables

---

## 🔹 3. Load

After transformation:

* Store data into a target system
* Usually a **Data Warehouse**

---

# 5️⃣ Tech Stack for Data Engineering

## 🔹 Programming Languages

* Python
* SQL (Very important)
* Java (optional)

## 🔹 Databases

* MySQL
* PostgreSQL
* Snowflake
* Redshift

## 🔹 Cloud Platforms

* AWS
* Azure
* GCP

## 🔹 ETL Tools

* Apache Airflow
* Talend
* Informatica
* dbt

---

# 6️⃣ What is Big Data?

## 🔹 Normal Data

Data that can be processed using:

* Single machine
* Traditional databases
* Limited scale

## 🔹 Big Data

When data:

* Is too large for a single machine
* Cannot be processed using traditional tools
* Requires distributed computing

---

## 🔹 Example: Meesho

If:

* 15 Billion views per day
* 365 days storage
* Billions of user interactions

Traditional databases cannot handle:

* That volume
* That speed
* That scale

This is **Big Data**.

---

## 🔹 The 4 V's of Big Data

1. **Volume** – Huge data size
2. **Velocity** – Speed of generation
3. **Variety** – Different formats
4. **Value** – Extracting meaningful insights
5. **Veracity** (sometimes added) – Data reliability

---

# 7️⃣ Core Responsibilities of Data Engineering

1. Handling large volumes
2. Ensuring fast processing
3. Managing different data types
4. Optimizing cost
5. Ensuring data reliability
6. Building scalable pipelines

---

# 8️⃣ Modern Data Architecture

## 🔹 Step 1: Data Sources

Examples:

* Website interactions
* Mobile apps
* APIs
* Payment systems
* ERP systems
* Third-party vendors

---

## 🔹 Step 2: Two Storage Paths

### Path 1 → Direct ETL → Data Warehouse

If data is structured:

* Direct ETL
* Stored in Data Warehouse

---

### Path 2 → Data Lake → ETL → Data Warehouse

If data is unstructured:

* Dump into Data Lake (raw format)
* Later process via ETL
* Move to Data Warehouse

---

# 9️⃣ What is a Data Lake?

A **Data Lake** stores:

* Raw data
* Structured + semi-structured + unstructured
* Cheap storage (usually cloud-based)

Examples:

* Images
* Videos
* Logs
* JSON
* CSV

Usually stored in:

* AWS S3
* Azure Blob Storage
* Google Cloud Storage

---

# 🔟 What is a Data Warehouse?

A **Data Warehouse**:

* Stores structured data
* Optimized for analytics
* Stores historical data (5–10 years)
* Supports complex queries

Examples:

* Amazon Redshift
* Snowflake
* BigQuery

---

# 1️⃣1️⃣ What are Data Marts?

A **Data Mart** is:

* A subset of Data Warehouse
* Designed for a specific team

Example:

* Finance Data Mart
* Marketing Data Mart
* Sales Data Mart

Why?
Because entire warehouse is too big for daily queries.

---

# 1️⃣2️⃣ BI Layer

Business Intelligence tools sit on top:

* Tableau
* Power BI
* Looker

They:

* Generate dashboards
* Create reports
* Help business take decisions

---

# 1️⃣3️⃣ Final Architecture Summary

```
Data Sources
      ↓
  (Option 1) ETL → Data Warehouse
      ↓
  Data Marts
      ↓
   BI Tools

OR

Data Sources
      ↓
  Data Lake (raw dump)
      ↓
  ETL
      ↓
  Data Warehouse
      ↓
  Data Marts
      ↓
  BI Layer
```

---

# 1️⃣4️⃣ OLTP vs OLAP

## 🔹 OLTP – Online Transaction Processing

Used for:

* Real-time transactions

Example:

* Amazon order placement
* Inventory update
* Payment processing

Characteristics:

* Fast inserts
* High concurrency
* Small queries
* ACID compliance

Stored in:

* RDBMS (MySQL, PostgreSQL)

---

## 🔹 OLAP – Online Analytical Processing

Used for:

* Analytical queries
* Historical data analysis

Example:

* Total sales last year
* Region-wise performance
* Monthly revenue trend

Characteristics:

* Complex joins
* Aggregations
* Large scans

Stored in:

* Data Warehouse

---

##  Key Difference

| Feature | OLTP         | OLAP       |
| ------- | ------------ | ---------- |
| Purpose | Transactions | Analytics  |
| Data    | Current      | Historical |
| Queries | Small        | Complex    |
| Users   | End users    | Analysts   |

---

# 1️⃣5️⃣ How Do Companies Store Data?

Modern companies use:

* Cloud storage (AWS S3)
* Distributed systems
* Data Lakes
* Data Warehouses
* ETL orchestration tools
* Data governance tools

They focus on:

* Scalability
* Cost optimization
* Fault tolerance
* Data quality

---

# 1️⃣6️⃣ What is Hadoop?

Hadoop is a **distributed computing framework** that allows:

* Storing huge data across multiple machines
* Processing data in parallel

It solves:

* Big data storage
* Big data processing

---

# 1️⃣7️⃣ What is HDFS?

HDFS = Hadoop Distributed File System

It:

* Stores large files across multiple machines
* Breaks files into blocks
* Replicates blocks for fault tolerance

Example:
If file = 1 TB
HDFS:

* Break into 128MB blocks
* Store across cluster
* Replicate 3 times

If one machine fails:

* Data is safe

---
---
---
what is Data Engineering ?
=>
whenever a human interacts with the machine, data is generated.and it requires a placed to be stored.for now .. somewhere the data gets stored. but the data which is stored it is not in the usable format .

we have the storage layer, we use that layer and try to ensure that data resides in the consumption layer .
in the conversion process they do something known as 
ETL (Extract Transform Load) => what does it do =>
we have to extract , so this could be multiple sources , from these sources we need to figure out which data we need for the consumption layer. after that we dont know whether the data is in clean structured form that can be consumed , so we need to apply the transformation layer . when we say transform , its the unstructured data which is bought into the structured format . now this structured data needs to be stored and that is where this load thing comes into picture.

Techstacks to know :
programming language : python , sql (preferred) or java
tools : ETL pipelines , cloud => (AWS, Azure, GCP)
databases : mysql postgress


what is the difference between big data and data ?
= > what is the threshold to call the data as a big data.

---
 lets take an example of meesho =>

 the total number of views that we get in one day is around 15B views in total of different products that a user sees in the platform . this is just for 1 day.
 just imagine if we want to store the data for 365 days  and if i want to analyze this data. this is called as a big data where in the volume that is coming every single day is not something that the normal computers can process .


as a part of data engineering  =>
1. we need to deal with large volume of data. 
2. speed of processing  (economics of scale blog)
3. variety of data which are (images, videos , txt files, csv files , json , xml)
4. cost ? => optimize for cost 

-------


Architecture diagram 
How does every company stores the data and processes the data .
1. we need to define the sources of data (eg: interaction on website, APIs , transactions, EAP, 3rd party data )
now this data is genrated and we need to do the processing of this data  and we will do it by ETL .
this data will be dumped to this location called as  => it the data is structured if i want to store at one particular location that location which will have all the historic data is called as Datawarehouse.
any datapoint which is currently not in the structured format we will dump it on the location called as DataLake.
can we integrate the output of datalake to datawarehouse the answer is yes. the step is ETL

for any data that i dont need on day to day basis, we will put that into aws , so datalake ki saare cheeze aws mei hogi.


Datawarehouse we can consider to have almost 10 years of data .
we will create another layer and that layer is called as data marks .
what is datamarks ?



so in short the whole architecture is =>
first we have data sources, the data sources has two options, one is to do etl and go to datawarehouse and the second option is datalake , when i am dumping to the datalake, then i will not be doinf ETL because its a simple dump , but when going from datalake to datawarehouse then we have to do etl , infront of datawarehouse in order to create like small chunks of data which can be used on day to day purposes this is called as datamarks. and the BI layer stands on top of it . 








-------------------

what is the difference betwereen OLTP and OLAP ?
OLTP stands for Online Transaction Processing
OLAP stands for Online Analytical Processing

OLTP =>
mysql is an OLTP database 
eg :  you go to amazon and you place the order, as soon as you place the order you want the inventory table to reduce the quantity by one . and then you are placing the order 
The idea of OLTP database is to capture all the transaction that are happening realtime.
so anytime a transaction happens and that transaction is leading to the chain of events that kind of data is processed using oltp and data where it is stored is called as  RDBMS systems .

OLAP =>
the data residing on the datawarehouse can be used for porcessing anaytical queries which is your OLAP and all your historic structured data resides on datawarehouse 



-------

how does all the company in the world store the data ?



what is HADOOP 
what is HDFS 

---