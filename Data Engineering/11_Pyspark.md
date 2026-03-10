

---

# Data Engineering – PySpark Notes
# Working with RDD in PySpark

## 1. Introduction to PySpark

PySpark is the Python API for Apache Spark that allows developers to perform distributed data processing using Python.

Spark is designed to process large datasets efficiently using:

- Distributed computing
- In-memory processing
- Parallel execution

PySpark enables data engineers to write scalable data processing pipelines using Python instead of complex MapReduce code.

Spark supports three main abstractions:

1. RDD (Resilient Distributed Dataset)
2. DataFrame
3. Dataset (mainly in Scala)

RDD is the **lowest-level abstraction** in Spark.

---

# 2. Spark Session Initialization

Before performing any Spark operation, we must create a **SparkSession**.

SparkSession acts as the entry point to Spark functionality.

Example:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MyFirstSparkApp") \
    .master("local[*]") \
    .getOrCreate()
````

Explanation:

* `appName("MyFirstSparkApp")`
  Assigns a name to the Spark application.

* `master("local[*]")`
  Runs Spark locally using all available CPU cores.

* `getOrCreate()`
  Creates a new Spark session or returns an existing one.

---

# 3. Spark Context

SparkContext connects the Spark application to the cluster.

It allows us to create RDDs and execute distributed operations.

Example:

sc = spark.sparkContext


SparkContext responsibilities:

* Connects driver program to cluster
* Creates RDDs
* Distributes tasks to worker nodes

---

# 4. Creating RDDs

RDDs can be created in multiple ways.

## 4.1 Creating RDD from Python Collection

Example:

```python
rdd1 = sc.parallelize([1,2,3,4])
```

Explanation:

* Converts a Python list into a distributed dataset.
* Data is split into partitions.

---

## 4.2 Creating RDD from File

Example:

```python
rdd2 = sc.textFile("path")
```

This reads a file from:

* HDFS
* Local file system
* Cloud storage

Each line of the file becomes one element in the RDD.

---

# 5. Transformations and Actions

Spark operations fall into two categories:

1. Transformations
2. Actions

---

# 6. Transformations

Transformations are operations that create a **new RDD from an existing RDD**.

Important property:

Transformations are **lazy**.

This means Spark does not execute them immediately.

Execution happens only when an action is called.

Examples of transformations:

* map()
* filter()
* groupByKey()
* reduceByKey()
* join()

Example transformation:

```python
filtered = rdd1.filter(lambda x: x > 2)
double = filtered.map(lambda x: x * 2)
```

Here:

1. Filter values greater than 2
2. Multiply each value by 2

These operations are not executed until an action is called.

---

# 7. Actions

Actions trigger execution of transformations and return results.

Examples:

* collect()
* count()
* take(n)
* reduce()

Example:

```python
result = double.collect()
```

This triggers the entire Spark execution.

---

# 8. Common Action Operations

## collect()

Returns all elements of the RDD to the driver program.

Example:

```python
rdd.collect()
```

Important:

This should be used carefully because it can cause memory issues if the dataset is large.

---

## count()

Returns the total number of elements in the RDD.

Example:

```python
rdd.count()
```

---

## take(n)

Returns the first `n` elements.

Example:

```python
rdd.take(5)
```

---

## reduce()

Aggregates the elements of the RDD using a function.

Example:

```python
rdd.reduce(lambda a,b: a+b)
```

---

# 9. Example Workflow

Suppose we have a text file.

Processing steps:

1. Read file
2. Apply transformations
3. Perform action

Example:

```python
rdd = sc.textFile("file.txt")

filtered = rdd.filter(lambda line: "error" in line)
mapped = filtered.map(lambda line: line.split(","))

result = mapped.collect()
```

Execution occurs only when `collect()` is called.

---

# 10. Example: Processing CSV File

Reading CSV file into RDD:

```python
rdd = sc.textFile("/user/scalerhdpfeb26019/employees.csv")
```

Check first few rows:

```python
print(rdd.take(5))
```

Other useful functions:

```python
rdd.getNumPartitions()
rdd.count()
rdd.first()
rdd.collect()
```

Explanation:

* `getNumPartitions()` → number of partitions
* `count()` → number of rows
* `first()` → first row
* `collect()` → return entire dataset

---

# 11. Removing Header

Often CSV files contain a header row.

Example:

```
first_name,last_name,email,phone,...
```

We remove it using filter.

```python
header = rdd.first()

data_rdd = rdd.filter(lambda line: line != header)
```

---

# 12. Splitting Rows

Convert each CSV line into a list.

```python
data_rdd = data_rdd.map(lambda line: line.split(","))
```

Example output:

```
["John","Doe","john@example.com","12345"]
```

---

# 13. Parsing Rows into Dictionary

To make data easier to work with, convert each row into a dictionary.

Example function:

```python
def parse_row(fields):
    return {
        "first_name": fields[0],
        "last_name": fields[1],
        "email": fields[2],
        "phone": fields[3],
        "gender": fields[4],
        "age": int(fields[5]),
        "job_title": fields[6],
        "years_of_experience": int(fields[7]),
        "salary": float(fields[8]),
        "department": fields[9]
    }
```

Apply transformation:

```python
parsed_rdd = data_rdd.map(parse_row)
```

Now each record becomes structured data.

---

# 14. Filtering Data

Example: Filter female employees.

```python
female_rdd = parsed_rdd.filter(lambda emp: emp["gender"] == "Female")
```

Example:

```python
female_rdd.take(5)
```

---

# 15. Filtering Based on Salary

Example: Employees with salary greater than 50000.

```python
high_salary_rdd = parsed_rdd.filter(lambda emp: emp["salary"] > 50000)
```

---

# 16. Mapping Data

Extract name and salary.

Example:

```python
name_salary_rdd = parsed_rdd.map(
    lambda emp: (f"{emp['first_name']} {emp['last_name']}", emp["salary"])
)
```

Output format:

```
("John Doe", 65000)
```

---

# 17. Key-Value RDD

Many Spark operations require key-value pairs.

Example:

```python
dept_salary_rdd = parsed_rdd.map(lambda emp: (emp["department"], emp["salary"]))
```

Example output:

```
("Engineering", 75000)
("HR", 50000)
```

---

# 18. Aggregation with reduceByKey

Calculate total salary per department.

```python
total_salary_by_dept = dept_salary_rdd.reduceByKey(lambda a,b: a+b)
```

Result:

```
("Engineering", 5000000)
("HR", 2000000)
```

Display result:

```python
total_salary_by_dept.collect()
```

---

# 19. groupByKey

Group values by key.

Example:

```python
grouped = dept_salary_rdd.groupByKey()
```

However, `groupByKey()` is usually avoided in production because it moves large amounts of data during shuffle.

Instead, `reduceByKey()` is preferred.

---

# 20. mapValues

Apply transformation only to values.

Example:

```python
after_tax_rdd = dept_salary_rdd.mapValues(lambda salary: salary * 0.8)
```

This calculates salary after tax.

---

# 21. Extracting Salary Data

Example:

```python
salary_rdd = parsed_rdd.map(lambda emp: emp["salary"])
```

Now the RDD contains only salary values.

---

# 22. Statistical Operations

Spark provides built-in statistical functions.

Example:

```python
salary_rdd.sum()
salary_rdd.min()
salary_rdd.max()
salary_rdd.mean()
salary_rdd.stdev()
```

These help analyze salary distribution.

---

# 23. stats()

Spark can return multiple statistics at once.

Example:

```python
salary_rdd.stats()
```

Output includes:

* count
* mean
* standard deviation
* min
* max

---

# 24. Example Output

After-tax salary sample:

```python
after_tax_rdd.take(5)


Example result:


("Engineering", 60000)
("Finance", 52000)




---

# 25. Spark Execution Model

When executing transformations:

Spark builds a **DAG (Directed Acyclic Graph)**.

Example pipeline:

```
Read CSV
    ↓
Remove header
    ↓
Parse rows
    ↓
Filter employees
    ↓
Aggregate salaries
    ↓
Return results


Execution starts only when an action is triggered.

---

# 26. Important Best Practices

1. Avoid using `collect()` for large datasets.
2. Prefer `reduceByKey()` over `groupByKey()`.
3. Use partitioning for large datasets.
4. Cache frequently used RDDs.

Example caching:


parsed_rdd.cache()


---

```
```
