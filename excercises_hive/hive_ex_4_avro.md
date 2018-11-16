Import all tables from mysql into hdfs files to data warehouse directory retail_stage.db.
Import as avro data using snappy compression.

Create a metastore table named orders_sqoop that point to the orders data.
Show all orders of the day when most orders where placed. 

```
sqoop import-all-tables \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--warehouse-dir /user/hive/warehouse/retail_stage.db \
--as-avrodatafile \
--z \
--compression-codec "org.apache.hadoop.io.compress.SnappyCodec"

hadoop fs -ls /user/cloudera/retail_stage.db
hadoop fs -ls /user/cloudera/retail_stage.db/orders

hadoop fs -ls /user/hive/warehouse
hadoop fs -ls /user/hive/warehouse/retail_stage.db
```


```
hadoop fs -get /user/hive/warehouse/retail_stage.db/orders/part-m-00000.avro
hadoop fs -get /user/cloudera/retail_stage.db/orders/part-m-00000.avro

avro-tools getschema part-m-00000.avro > orders.avsc
#/user/hive/schemas does not exist
hdfs dfs rm -r /user/hive/schemas
hadoop fs -mkdir /user/hive/schemas
hadoop fs -ls /user/hive/schemas/order
hadoop fs -copyFromLocal orders.avsc /user/hive/schemas/order
```


In hive: create a table pointing to the avro data file for orders data
```
create external table orders_sqoop
STORED AS AVRO
LOCATION '/user/cloudera/retail_stage.db/orders'
TBLPROPERTIES ('avro.schema.url'='/user/hive/schemas/order/orders.avsc');
```

Could not load .avsc file into '/user/hive/schemas/order/'
Check permission on folder
https://stackoverflow.com/questions/38290847/unable-to-read-schema-from-given-path-hdfs-avsc
General on avro: https://www.cloudera.com/documentation/enterprise/5-14-x/topics/cdh_ig_avro_usage.html


```
hadoop fs -mkdir /user/cloudera/schemas
hadoop fs -copyFromLocal orders.avsc /user/cloudera/schemas
hadoop fs -ls /user/cloudera/schemas

create external table orders_sqoop
STORED AS AVRO
LOCATION '/user/hive/warehouse/retail_stage.db/orders'
TBLPROPERTIES ('avro.schema.url'='/user/cloudera/schemas/orders.avsc');

create external table orders_sqoop_v2
STORED AS AVRO
LOCATIONl '/user/hive/warehouse/retail_stage.db/orders'
TBLPROPERTIES ('avro.schema.url'='hdfs:///user/cloudera/schemas/orders.avsc');


hadoop fs -copyFromLocal orders.avsc 
hadoop fs -ls

CREATE EXTERNAL TABLE orders_sqoop_v3 
STORED AS AVRO 
LOCATION '/user/hive/warehouse/retail_stage.db/orders'
TBLPROPERTIES ('avro.schema.url'='orders.avsc');
```

Show all orders of the day when most orders where placed.
Lesson learned: specify table names orders_sqoop.order_date in ... , just order_date in doesn't workd
```
select *
from orders_sqoop 
where orders_sqoop.order_date in (
       select date_n_orders.order_date from 
        (
            select order_date, count(*) as N_orders
            from orders_sqoop
            group by order_date
            order by N_orders desc
            limit 1
        ) date_n_orders
) ;
```


