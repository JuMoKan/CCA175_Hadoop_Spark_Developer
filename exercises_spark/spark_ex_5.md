Import orders and order_items table from mysl to hdfs to /user/cloudera/spark_ex_5/orders resp. order-items.                             
Import files as Avro File and use snappy compression.
Load data into spark.
Calculate total orders and total amount per status per day.
Save result as as parquet file and as csv file.

```
sqoop-import \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--table orders  \
--target-dir /user/cloudera/spark_ex_5/orders  \
--as-avrodatafile \
--compress \
--compression-codec org.apache.hadoop.io.compress.SnappyCodec 
```

```
sqoop-import \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--table order_items \
--target-dir /user/cloudera/spark_ex_5/order_items \
--as-avrodatafile \
--compress \
--compression-codec org.apache.hadoop.io.compress.SnappyCodec 
```

```
from pyspark.sql import SQLContext
sqlContext = SQLContext(sc)
orders = sqlContext.read.format("com.databricks.spark.avro").load("/user/cloudera/spark_ex_5/orders")
orders.registerTempTable("orders")
order_items = sqlContext.read.format("com.databricks.spark.avro").load("/user/cloudera/spark_ex_5/order_items")
order_items.registerTempTable("order_items")
```

Calculate total orders and total amount per status per day.
```
N_Total_per_day = sqlContext.sql("""
    select o.order_date, count(distinct o.order_id) as N_orders, sum(oi.order_item_subtotal) as Total_amount
    from orders o
    join order_items oi
    on o.order_id = oi.order_item_order_id
    group by o.order_date
    order by o.order_date desc 
    """)

N_Total_per_day.head(20)
```

Save as parquet/csv file
```
N_Total_per_day.write.parquet("/user/cloudera/spark_ex_5/N_total_per_day")

N_Total_per_day.repartition(1).rdd.map(lambda x: str(x[0])+ "," + str(x[1]) + "," + str(x[2])) \
    .saveAsTextFile("/user/cloudera/spark_ex_5/csv_N_total_per_day")

hdfs dfs -get /user/cloudera/spark_ex_5/csv_N_total_per_day /home/cloudera/hdfs_output

```