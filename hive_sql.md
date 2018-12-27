## Mysql
mysql -uroot -pcloudera  
mysql -u retail_dba -p
mysql -h localhost -u retail_db -p  
GRANT ALL PRIVILEGES ON <database_name>.* to ''@'localhost';

Sqoop export: To export check / create table in MySQL. 
```
create table retail_db.result(
	order_status varchar(255) not null, 
	order_date date not null,
	total_orders int, 
	total_amount numeric, 
	constraint pk_order_result primary key (order_date,order_status)); 
```

```
create table products_replica as select * from products;
alter table products_replica add primary key (product_id);
alter table products_replica add column (product_grade int, product_sentiment varchar(100))
update products_replica set product_grade = 1  where product_price > 500;
update products_replica set product_sentiment  = 'WEAK'  where product_price between 300 and  500;
```

## Mysql employees db
https://github.com/datacharmer/test_db
https://dev.mysql.com/doc/employee/en/


## Hive: check settings
https://stackoverflow.com/questions/42239139/how-can-i-check-the-settings-in-hive-cli
SET hive.default.fileformat;

## Hive: avro:AvroSerDe
* https://cwiki.apache.org/confluence/display/Hive/AvroSerDe

## Hive: Work with dates
* http://dwgeek.com/hadoop-hive-date-functions-examples.html/
* https://stackoverflow.com/questions/45014172/how-to-convert-bigint-to-datetime-in-hive
* https://community.hortonworks.com/questions/82797/hive-convert-int-timestamp-to-date.html


## sqlContext / hiveContet: show Tables
all_tables = sqlContext.tables()
all_tables.show()

hive_all_tables = hiveContext.tables()
hive_all_tables.show()

## Hive: Create tables

Documentation: Hive Documentation --> User Documentation --> DDL  
https://www.dezyre.com/hadoop-tutorial/apache-hive-tutorial-tables

hive-shell
```
create database retail_db;  
use retail_db;

create table orders (  
  order_id int,  
  order_date string,  
  order_customer_id int,    
  order_status string    
)  
row format delimited fields terminated by ','  
stored as textfile;  
```

pyspark-shell
```
hive_context = HiveContext(sc)

hive_context.sql("CREATE TABLE IF NOT EXISTS default.myTab2 (plz INT, city STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ','")
hive_context.sql("CREATE TABLE IF NOT EXISTS retail_db.myTab2 (plz INT, city STRING)")

```

## Hive: Load data into tables

Load data from hdfs
```
hdfs dfs -copyFromLocal /home/cloudera/TEST_text_from_local.csv /user/cloudera/TEST_text_from_local.csv

hive_context.sql("load data inpath '/user/cloudera/TEST_text_from_local.csv' (overwrite) into table default.myTab ")
```

Load local data
```
hive_context.sql("load data local inpath '/home/cloudera/TEST_text_from_local.csv' (overwrite) into table default.myTab2 ")
```

Load from other tables in hive
```
hive_context.sql("create table if not exists default.myTab3 AS SELECT * FROM default.myTab")
hive_context.sql("insert into table default.myTab2 SELECT * FROM default.myTab")
```

Write from spark to hive

Create dataframe in spark
```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username retail_dba \
--password cloudera \
--table products \
--columns "product_id, product_category_id, product_name"

from pyspark.sql import Row

products=  sc.textFile("/user/cloudera/products") \
.map(lambda l: l.split(",")) \
.map(lambda p: Row(id=int(p[0]), category=p[1], name=p[2])).toDF()

products=products["id", "category", "name"]
products.show(20)
```

create empty table in hive
```
from pyspark.sql import HiveContext
hiveContext = HiveContext(sc)
hiveContext.sql("CREATE TABLE default.products_created_table (id string, category STRING,name STRING)")
hiveContext.sql("CREATE TABLE problem6.products_created_table (id string, category STRING,name STRING)")
```

load data into hive
```
#products = hive_context.createDataFrame(products)
products.registerTempTable("products")
#products.write.insertInto("products_cat_name")

products.write.mode("append").saveAsTable("products_cat_name2")
```



