# Exercise:
Import customer_id, customer_city, customer_state from mysql customer table into hfds as parquet-file.
Use: /user/cloudera/import_export/ex_1/imp_customer_parquet

Convert into csv file using gzip compression and save in HDFS.
Use: /user/cloudera/import_export/ex_1/exp_customer_csv_gzip)

### Step 1: Get the data from mysql to hdfs  
```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--table customers \
--columns "customer_id, customer_city, customer_state" \
--target-dir /user/cloudera/import_export/ex_1/imp_customer_parquet \
--as-parquetfile
```

### Step 2: Read in as pyspark.sql.dataframe.DataFrame
pyspark.sql module - Module Context - DataFrameReader

```
customer = sqlContext.read.parquet('/user/cloudera/import_export/ex_1/imp_customer_parquet/')
```

### Step 3: Convert into csv file using gzip compression
pyspark package - RDD - saveAsTextFile

```
codec = "org.apache.hadoop.io.compress.GzipCodec"

customer.repartition(1).rdd \
  .map(lambda x: str(x[0])+ "," + x[1] + "," + x[2]) \
  .saveAsTextFile("/user/cloudera/import_export/ex_1/exp_customer_csv_gzip/", codec)

 hdfs dfs -get /user/cloudera/import_export/ex_1/exp_customer_csv_gzip /home/cloudera/hdfs_output

```

# Exercise  -- avro version

Import customer_id, customer_city, customer_state from mysql customer table into hfds as avro-file.
Use: /user/cloudera/import_export/ex_1/imp_customer_avro

Convert into csv file using gzip compression and save in HDFS.
Use: /user/cloudera/import_export/ex_1/exp_customer_csv_gzip)

### Step 1: Get the data from mysql to hdfs  
```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--table customers \
--columns "customer_id, customer_city, customer_state" \
--target-dir /user/cloudera/import_export/ex_1/imp_customer_avro  \
--as-avrodatafile
```

### Step 2: Read in as pyspark.sql.dataframe.DataFrame
pyspark.sql module - Module Context - DataFrameReader -- format

```
customer_avro = sqlContext.read \
  .format("com.databricks.spark.avro") \
  .load('/user/cloudera/import_export/ex_1/imp_customer_avro/')
```

Didn't work (import com.databricks.spark.avro._)
https://github.com/databricks/spark-avro
```
import com.databricks.spark.avro._
customers_avro2 = sqlContext.read.avro(/user/cloudera/import_export/ex_1/imp_customer_avro/)
```