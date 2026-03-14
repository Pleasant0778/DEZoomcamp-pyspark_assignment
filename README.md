# NYC Taxi Data Pipeline with PySpark

## Overview

This project demonstrates a simple **data engineering pipeline using PySpark** to:

1. Download Nov 2025 NYC Taxi trip data from NYC website
2. Load and process the dataset using Spark. 
3. Run analytical queries using Spark SQL. 

The pipeline shows how distributed processing frameworks can be used to process large datasets efficiently.

---

# Architecture

The pipeline follows this flow:

```
Public Dataset (NYC Taxi Data)
            │
            ▼
      Python Requests
   (Streaming Download)
            │
            ▼
      Local Parquet File
            │
            ▼
        Spark Cluster
   (Data Processing + SQL)
            │
            ▼
      Analytical Results 
```

---

# Project Features

* Memory-safe file download using streaming.
 * Feature engineering with PySpark functions.
* SQL-based analytics using Spark SQL. 

---

# Project Structure

```
project-folder/
│
├── pipeline.py
├── README.md
└── yellow_tripdata_2025-11.parquet   (downloaded automatically)
```

---

# Requirements

Install the required dependencies:

```
pip install pyspark requests
```
 
---

 
The pipeline will:

1. Download the NYC taxi dataset.
2. Load the dataset into Spark.
3. Perform transformations.
4. Execute analytical queries.
5. Display results.

---

# Pipeline Implementation

## 1. Initialize Spark Session

Spark is initialized with the BigQuery connector.

```python
from pyspark.sql import SparkSession
import pyspark.sql.functions as F
from pyspark import SparkFiles

spark = SparkSession.builder \
    .master("local[*]") \
    .appName('pyspark-assignment') \
    .config("spark.sql.repl.eagerEval.enabled", True) \
    .getOrCreate()
```

---

# 2. Download Dataset (Streaming)

The dataset is downloaded using streaming to avoid memory issues.

```python

import requests
url = "https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2025-11.parquet"

filename = "yellow_tripdata_2025-11.parquet" 

def download_file(url, filename):
     with requests.get(url, stream=True) as r:
        r.raise_for_status()
        with open(filename, 'wb') as f:
            for chunk in r.iter_content(chunk_size=8192): 
                f.write(chunk)

```

---
 

# 3 Load Dataset

The file is loaded using Spark and repartitioned for parallel processing.

```python
spark_df = spark.read.parquet("yellow_tripdata_2025-11.parquet")
spark_df.show(2)

spark_df = spark_df.repartition(4)

spark_df.write.parquet('output/',mode="overwrite")
```

Repartitioning improves workload distribution across worker nodes.

---

# 4. Data Analysis with Spark

We compute trip duration and filter invalid records.

```python
spark_df.withColumn("tpep_pickup_datetime",F.to_date("tpep_pickup_datetime")) \
        .where(F.col("tpep_pickup_datetime") == '2025-11-15').count()
```
```python
(spark_df.withColumn("trip_duration_hrs",(F.col("tpep_dropoff_datetime").cast("timestamp").cast("long") -
                                                F.col("tpep_pickup_datetime").cast("timestamp").cast("long")) / 3600)  
        .select(F.max("trip_duration_hrs").alias("longest trip in hrs")).show()
)
```



---

# 5. Spark SQL Analysis

The transformed dataset is registered as a temporary SQL table.

```python
df_transformed.createOrReplaceTempView("taxi_data")
```

We run analytical queries:

```sql
spark.sql("""
select a.*, b.zone from (
SELECT PULocationID,  count(1) as cnt 
from nov_2025_tripdata a
GROUP BY PULocationID  
order by cnt asc
limit 20 ) a, zone_table b
where A.PULocationID = B.LocationID
order by cnt asc
""").show()
```
 
---

# Key Concepts Demonstrated

* Distributed data processing
* Spark DataFrame transformations
* Spark SQL analytics

---

# Possible Improvements

* Read data directly from cloud storage instead of downloading locally.
* Add orchestration using workflow tools.
* Add data validation and schema enforcement.
* Partition output tables for better query performance.
* Schedule pipeline runs.

---

# Conclusion

This pipeline demonstrates a simple but practical data engineering workflow combining:

* Python
* PySpark
* Distributed processing
* SQL analytics

It serves as a foundation for building scalable data processing pipelines.

---
