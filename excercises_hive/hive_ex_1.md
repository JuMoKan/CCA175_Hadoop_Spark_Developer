# Excercise:

Import customers table from mysql to hive. 
Get customers who live in Chicago.
Save the results in HDFS /user/cloudera/hive_ex_1 as parquet file.


## Import customers table from mysql to hive
```
sqoop import \
 --connect "jdbc:mysql://localhost/retail_db" \
 --username root \
 --password cloudera \
 --table customers \
 --hive-import \
 --create-hive-table \
 --hive-database default \
 --hive-table customers \
 -m 1
```

## Get customers who live in Chicago 

```
from pyspark.sql import HiveContext
hiveContext = HiveContext(sc)

customers_chicago = hiveContext.sql("""
	 select *
	 from default.customers
	 where customer_city = "Chicago"
""")
```
## Save the results in HDFS /user/cloudera/hive_ex_1  as parquet file.

```
customers_chicago.write.parquet("/user/cloudera/hive_ex_1")
```