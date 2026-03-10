
# Data Engineering – PySpark Notes 
# Spark DataFrames: Structured Processing in PySpark

## 1. Why Move from RDD to DataFrames?

In earlier Spark programming, **RDD (Resilient Distributed Dataset)** was the primary abstraction used for distributed data processing.

However, RDDs store data in a **raw, unstructured format**, typically as lists or tuples.

Example representation in RDD:

```

[1,100,d1]
[2,200,d2]
[3,300,d3]

```

Here:
- `1` → emp_id
- `100` → salary
- `d1` → department

The problem with this representation is that **column names are not available**. Data is accessed using **index positions**.

Example:

```

salary = rdd[2]

````

This makes the code:
- Harder to read
- Harder to maintain
- Error-prone for large datasets

Another problem arises when reading CSV files. The header row must be separated manually.

Typical RDD workflow:

1. Read file
2. Extract header
3. Filter header
4. Split rows
5. Map fields to structure

Because of these limitations, Spark introduced **DataFrames**, which provide a structured format similar to **Pandas DataFrames or SQL tables**.

---

# 2. What is a Spark DataFrame?

A **DataFrame** is a distributed collection of data organized into **named columns**.

It is conceptually similar to:

- Pandas DataFrame
- SQL table
- Excel sheet

A DataFrame contains:

- Column names
- Data types
- Row records

Example structure:

| first_name | last_name | gender | age | salary | department |
|-------------|-----------|--------|-----|--------|-------------|
| Sneha | Singh | Male | 32 | 47737 | Marketing |
| Amit | Mehta | Male | 43 | 10956 | Marketing |

Advantages of DataFrames:

- Easier to read and write
- Schema support
- SQL-style operations
- Optimized execution using Catalyst optimizer
- Better integration with Spark SQL

Although performance differences between RDD and DataFrame may not always appear obvious in small examples, **DataFrames are generally preferred in production systems**.

---

# 3. Installing PySpark

If working in environments such as Google Colab or local Jupyter notebooks, PySpark must first be installed.

Example:

```python
!pip install pyspark
````

---

# 4. Creating Spark Session

Spark operations begin with creating a **SparkSession**.

Example:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .master("local[*]") \
    .appName("ColabSpark") \
    .getOrCreate()
```

Explanation:

* `master("local[*]")`
  Runs Spark locally using all available CPU cores.

* `appName("ColabSpark")`
  Name of the Spark application.

* `getOrCreate()`
  Creates a new session or returns an existing one.

---

# 5. Spark Context

SparkSession internally contains a SparkContext.

Example:

```python
sc = spark.sparkContext
```

SparkContext manages:

* Cluster communication
* Task distribution
* Resource allocation

---

# 6. Data Types in Spark

Spark DataFrames require a **schema**, which defines the structure of the data.

To define schemas, Spark provides types in `pyspark.sql.types`.

Example imports:

```python
from pyspark.sql.types import (
    StructType, StructField, StringType,
    IntegerType, DoubleType, TimestampType
)

from pyspark.sql.functions import col, sum as sum_, avg, count
```

---

# 7. Defining Schema Using StructType

Schema defines:

* Column name
* Data type
* Nullability

Example schema:

```python
schema = StructType([
    StructField("first_name",  StringType(),  True),
    StructField("last_name",   StringType(),  True),
    StructField("gender",      StringType(),  True),
    StructField("age",         IntegerType(), True),
    StructField("job_title",   StringType(),  True),
    StructField("experience",  IntegerType(), True),
    StructField("salary",      DoubleType(),  False),
    StructField("department",  StringType(),  True),
])
```

Explanation:

* `StructType` defines the table schema.
* `StructField` defines each column.

Example field:

```
StructField("salary", DoubleType(), False)
```

Meaning:

* Column name: salary
* Data type: double
* Cannot be null

---

# 8. Sample Dataset

Example in-memory dataset:

```python
in_memory_data = [
    ("Sneha","Singh","Male",32,"Marketing Executive",7,47737.0,"Marketing"),
    ("Amit","Mehta","Male",43,"Marketing Executive",7,10956.0,"Marketing"),
    ("Pooja","Patel","Female",47,"Sales Executive",18,113096.0,"Sales"),
    ("Amit","Joshi","Male",36,"Finance Analyst",9,132588.0,"Finance"),
    ("Priya","Sharma","Female",29,"HR Executive",4,56000.0,"HR"),
    ("Ravi","Kumar","Male",38,"Finance Analyst",12,145000.0,"Finance"),
    ("Neha","Gupta","Female",26,"Sales Executive",3,42000.0,"Sales"),
]
```

In real systems, this data usually comes from:

* CSV files
* HDFS
* Parquet files
* Hive tables

---

# 9. Creating a DataFrame

DataFrame can be created by combining schema and data.

Example:

```python
df = spark.createDataFrame(in_memory_data, schema)
df.show()
```

Output will display the structured table.

---

# 10. Lazy Evaluation

Spark uses **lazy evaluation**.

This means transformations are not executed immediately.

Execution occurs only when an **action** is called.

Example:

```
df.select(...)
df.filter(...)
df.groupBy(...)
```

No execution happens until:

```
df.show()
df.collect()
```

---

# 11. Selecting Columns

To display specific columns:

```python
df.select("first_name","salary").show()
```

Because DataFrames have column names, we do not need to reference columns using indexes.

---

# 12. Summary Statistics

Spark provides a describe function.

Example:

```python
df.describe().show()
```

This returns:

* count
* mean
* stddev
* min
* max

Usually applied to **numerical columns**.

---

# 13. Renaming Columns

Columns can be renamed in two ways.

Using alias:

```python
df.select(col("first_name").alias("name")).show()
```

Using withColumnRenamed:

```python
df.withColumnRenamed("first_name","fname").select("fname").show()
```

---

# 14. Filtering Rows

Filtering in DataFrames is easier compared to RDD.

Example:

```python
df.filter(col("department") == "Marketing").show()
```

---

# 15. Filtering with Multiple Conditions

Example using `isin`:

```python
df.filter(col("department").isin("Marketing","HR")).show()
```

Using SQL-style conditions:

```python
df.where("salary > 56000 AND gender = 'Female'").show()
```

Selecting specific columns:

```python
df.where("salary > 56000 AND gender = 'Female'") \
  .select("first_name","gender","salary") \
  .show()
```

---

# 16. Range Filtering

Example:

```python
df.filter(col("salary").between(50000,150000)) \
  .select("first_name","salary") \
  .show()
```

---

# 17. String Filtering

Example:

```python
df.filter(col("job_title").contains("Executive")) \
  .select("first_name","job_title") \
  .show()
```

---

# 18. Creating New Columns

Spark allows creating derived columns.

Example salary band classification:

```python
from pyspark.sql.functions import when

df.withColumn(
    "salary_band",
    when(col("salary") < 60000,"low")
    .when(col("salary") < 120000,"mid")
    .when(col("salary") < 180000,"high")
).select("first_name","last_name","salary","salary_band").show()
```

---

# 19. Aggregations

Example: department salary analysis

```python
df.groupBy("department") \
  .agg(
      count("*").alias("headcount"),
      sum_("salary").alias("total_salary"),
      round(avg("salary"),2).alias("avg_salary"),
      min("salary").alias("min_salary"),
      max("salary").alias("max_salary"),
  ) \
  .orderBy(col("total_salary").desc()) \
  .show()
```

---

# 20. Statistical Aggregations

Standard deviation and variance:

```python
df.groupBy("department") \
  .agg(
      round(stddev("salary"),2).alias("salary_stddev"),
      round(variance("salary"),2).alias("salary_variance")
  ).show()
```

---

# 21. Distinct Count

Example:

```python
df.groupBy("department") \
  .agg(countDistinct("job_title").alias("unique_roles")) \
  .show()
```

---

# 22. Joins in Spark DataFrames

Example department dataset:

```python
dept_data = [
   ("Marketing","Mumbai",500000),
   ("Sales","Delhi",800000),
   ("Finance","Bangalore",600000),
   ("HR","Chennai",300000),
   ("IT","Hyderabad",950000),
]
```

Create DataFrame:

```python
dept_df = spark.createDataFrame(dept_data, ["department","city","budget"])
```

---

# 23. Performing Join

Example inner join:

```python
df_all = df.join(dept_df, on="department", how="inner")
df_all.show()
```

Join types supported:

* inner
* left
* right
* outer
* left_semi
* left_anti

---

# 24. Spark SQL

Spark also allows running SQL queries directly.

Example:

```
df.createOrReplaceTempView("employees")

spark.sql("SELECT department, AVG(salary) FROM employees GROUP BY department")
```

This allows data engineers familiar with SQL to work easily with Spark.

---

# 25. Summary

DataFrames provide a structured way of processing data in Spark.

Advantages over RDD:

* Column names available
* Easier filtering
* SQL-like operations
* Schema support
* Better readability
* Integration with Spark SQL

Because of these advantages, **most production Spark pipelines use DataFrames instead of RDDs**.

```
```


------
----
----
----
----
---
---
-----
--



can we do the data processing using dataframe in spark ?

in RDD we are storing the data like i have the emp_id, salary, department 
i have to store the data in the form like [1,100,d1],[2,200,d2],[3,300,d3]
and thats the reaso n why i was not able to reference the header column using the filter and map column
first we seperate the header and store it seperately then store this particular data seperately using the function that we have seen .
it becomes slightly difficult to find the average salary you have to give rdd[2].
so the better alternative here is ... instead of using RDD we move to this particular thing called as dataframe this is exactly similar to that we have in pandas .
RDD is just the normal data being stored in a comma seperated value whereas dataframe gives the structre to the table and it gives the column names and it also gives the index number .
any difference in speed in RDD vs dataframe the answer is no it remains the same not major differences. advised to use dataframe because its more easy to understand .


so now we will do
in pyspark

        !pip install pyspark

        from pyspark.sql import SparkSession
        spark = SparkSession.builder \
            .master("local[*]") \
            .appName("ColabSpark") \
            .getOrCreate()

        sc = sc.sparkContext
        print (spark)

we need to use this thing called struct type 

        from pyspark.sql.types import (
        StructType, StructField, StringType,
        IntegerType, DoubleType, TimestampType
        )
        from pyspark.sql.functions import col, sum as sum_, avg, count


first we will define the schema 

    schema = StructType([
        StructField("first_name",  StringType(),  True),
        StructField("last_name",   StringType(),  True),
        StructField("gender",      StringType(),  True),
        StructField("age",         IntegerType(), True),
        StructField("job_title",   StringType(),  True),
        StructField("experience",  IntegerType(), True),
        StructField("salary",      DoubleType(),  False),   # Non-nullable
        StructField("department",  StringType(),  True),
    ])

and lets day i have some data 

        in_memory_data = [
            ("Sneha",  "Singh",  "Male",   32, "Marketing Executive", 7,  47737.0, "Marketing"),
            ("Amit",   "Mehta",  "Male",   43, "Marketing Executive", 7,  10956.0, "Marketing"),
            ("Pooja",  "Patel",  "Female", 47, "Sales Executive",     18, 113096.0,"Sales"),
            ("Amit",   "Joshi",  "Male",   36, "Finance Analyst",     9,  132588.0,"Finance"),
            ("Priya",  "Sharma", "Female", 29, "HR Executive",        4,  56000.0, "HR"),
            ("Ravi",   "Kumar",  "Male",   38, "Finance Analyst",     12, 145000.0,"Finance"),
            ("Neha",   "Gupta",  "Female", 26, "Sales Executive",     3,  42000.0, "Sales"),
        ]

this is not like your file data  consider this to  be like your csv file which is lying in hdfs , now what i can do is i can combine the two this schema and inmemory data and create a dataframe so 

        df = spark.createDataFrame(in_memory_data, schema )
        df.show()


and this will be the dataframe in spark... bit more structured , easier to read and understand 

again now we are following the lazy operation , so until and unless you say show it will not do anything , it will just store the code 


lets understand what we did
 in rdd we do not have this schema  , now we have a schhema qnd because we have a schema so our coding becomes easier and understadning to the data becomes easier and processing becomes much easier 



now if i need to select only few columns so what we need to do is 
we will write 

        df.select("column names which you want to see").show

since we have the column names now so we dont need to give the index 

some of the functions that comes with the dataframe .
df.describe.show()
mostly this describe is done on numerical columnsn

for example if i have to rename the colmun i have to display the column name differently 
we can do this by

        df.select(col("first_name").alias("name")).show
        df.withColumnRenamed("first_name", "fname").select("fname").show()


for filtering in RDD there is a big process but in dataframe we haev

        df.filter(col("department")== "Marketing").show()

for filtering with multiple conditions we have

        df.filter(col("department").isin ("Marketing", "HR")).show()

or we can do 

        df.where("salary >  56000 AND gender = 'Female'").show()
        df.where("salary >  56000 AND gender = 'Female'")\
        .select("first_name", "gender", "salary").show()

        df.filter(col("salary").between(50000, 150000)) \
        .select("first_name", "salary").show()

        df.filter(col("job_title").contains("Executive")) \
        .select("first_name", "job_title").show()


everything like sql is now available because of creating dataframe 

        df.withColumn("salary_band",
             when(col("salary") <60000 , "low")
             .when(col("salary") <120000 , "mid")
             .when(col("salary") <180000 , "high")).select("first_name", "last_name", "salary", "salary_band").show()


now we will do aggregations 

        df.groupBy("department") \
            .agg(
                count("*").alias("headcount"),
                sum_("salary").alias("total_salary"),
                round(avg("salary"), 2).alias("avg_salary"),
                min("salary").alias("min_salary"),
                max("salary").alias("max_salary"),
            ) \
            .orderBy(col("total_salary").desc()) \
            .show()

to know the std deviation and variance 

        df.groupBy("department") \
            .agg(
                round(stddev("salary"), 2).alias("salary_stddev"),
                round(variance("salary"), 2).alias("salary_variance")
            ).show()

using count distinct

        df.groupBy("department") \
            .agg(countDistinct("job_title").alias("unique_roles")) \
            .show()
    


let say i have a data  and now we will learn how to do joins using spark

        
dept_data = [
   ("Marketing", "Mumbai",    500000),
   ("Sales",     "Delhi",     800000),
   ("Finance",   "Bangalore", 600000),
   ("HR",        "Chennai",   300000),
   ("IT",        "Hyderabad", 950000),
]


dept_df = spark.createDataFrame(dept_data, ["department", "city", "budget"])


dept_df.show()
df_all = df.join(dept_df, on= "department", how ="inner" )
df_all.show()


and inside spark.sql we   can write any sql code and make it easy






