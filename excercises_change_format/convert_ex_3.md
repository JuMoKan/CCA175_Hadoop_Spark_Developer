Import products table from mysql 
* as text file to hdfs convert_ex3/import/text. Fields should be terminated by a tab character (“\t”) character and lines should be terminated by new line character (“\n”).
* as avro file to hdfs convert_ex3/import/avro
* as parquet to hdfs convert_ex3/import/parquet 

Convert data at convert_ex3/import/avro :
* as parquet file with snappy compression, save to convert_ex3/parquet-snappy-compress
* save the data to hdfs using gzip compression as text file at /user/`whoami`/problem5/text-gzip-compress
* save the data to hdfs using no compression as sequence file at /user/`whoami`/problem5/sequence
* save the data to hdfs using snappy compression as text file at /user/`whoami`/problem5/text-snappy-compress



```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username retail_dba \
--password cloudera \
--table products \
--as-avrodatafile \
--target-dir /user/cloudera/convert_ex3/import/avro
```

```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username retail_dba \
--password cloudera \
--table products \
--as-textfile \
--target-dir /user/cloudera/products \
--fields-terminated-by '|'
--lines-terminated-by "\n"
```


```
#import avro.schema
#from avro.datafile import DataFileReader, DataFileWriter
products_avro = sqlContext.read.format("com.databricks.spark.avro").load("/user/cloudera/convert_ex3/import/avro")
products_avro.show(20)

products_avro.write.parquet("/user/cloudera/convert_ex3/parquet-snappy-compress)
```