# Get information about databases, tables and data 

```
sqoop list-databases \
  --connect "jdbc:mysql://quickstart.cloudera:3306" \
  --username retail_dba \
  --password cloudera
```

```
sqoop list-tables \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera
```

Any valid sql query or command can be run using sqoop eval. 

```
sqoop eval \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera \
  --query "SELECT * FROM orders LIMIT 5"
  --query "CREATE TABLE dummy ...."
  --query "INSERT INTO orders VALUES (...)"

```


# Sqoop import

```
sqoop import-all-tables \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username=retail_dba \
  --password=cloudera \
  --warehouse-dir=/user/cloudera/sqoop_import
```

sqoop import-all-tables \
–exclude-tables, will facilitate to exclude the tables that need not be imported\
–auto-reset-to-one-mapper, will let import all tables to choose one mapper in case table does not have primary key\
-Most of the features such as –query, –boundary-query, –where etc are not available with import-all-tables.\


```
sqoop import \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username=retail_dba \
  --password=cloudera \
  --table departments \
  --target-dir /user/cloudera/sqoop_import/retail.db/departments
```

Import options \
--fields-terminated-by '|' : changes the field delimiter from the default \t to another one. \
--lines-terminated-by '\n' \
--driver is only relevant if the connection string does not begin with jdbc:mysql://. \
--null-non-string -99 : to recode NULL values from the database into -99 for non-string data types. \
--null-string "NA" : the same option for strings \
--null-non-string und null-string: If not specified, then the string "null" will be used.\
--num-mappers 1, default is 4 \


File-formats: text file (default), sequence file, avro, parquet, orc (?) \
--as-sequencefile \
--as-avrodatafile \
--as-parquetfile \



# Sqoop import to existing directories

By default sqoop import fails if target directory already exists \
Directory can be overwritten by using –delete-target-dir\
Data can be appended to existing directories by saying –append

```
sqoop import \
  --connect jdbc:mysql://ms.itversity.com:3306/retail_db \
  --username retail_user \
  --password itversity \
  --table order_items \
  --warehouse-dir /user/cloudera/sqoop_import/retail_db \
  --delete-target-dir
```


```
sqoop import \
  --connect jdbc:mysql://ms.itversity.com:3306/retail_db \
  --username retail_user \
  --password itversity \
  --table order_items \
  --warehouse-dir /user/cloudera/sqoop_import/retail_db \
  --append
```


# Compression

default: gzip \
If compression codec is not specified, gzip is used by default\
Compression codecs: SnappyCodec, BZip2Codec, GzipCodec, and DefaultCodec\

```
--compress (oder z) \
--compression-codec org.apache.hadoop.io.compress.GzipCodec
--compression-codec org.apache.hadoop.io.compress.SnappyCodec

```


# split-by / boundary-query

*Split logic will be applied on primary key, if exists.  \
*For performance reason choose a column which is indexed as part of split-by clause \
*If primary key does not exists and if we use number of mappers more than 1, then sqoop import will fail.We can use –split-by to split on a non key column or explicitly set –num-mappers to 1 or use –auto-reset-to-one-mapper  \
*If the primary key column or the column specified in split-by clause is non numeric type, then we need to use this additional argument -Dorg.apache.sqoop.splitter.allow_text_splitter=true \
*If there are null values in the column, corresponding records from the table will be ignored \
*Data in the split-by column need not be unique, but if there are duplicates then there can be skew in the data while importing (which means some files might be relatively bigger compared to other files)


```
sqoop import \
  --Dorg.apache.sqoop.splitter.allow_text_splitter=true \
  -- ...
  --split-by order_status
```


```
sqoop import \
  --connect...\
  --boundary-query 'select 1000, 2000'
```


# Import Query results, select columns, where


```
sqoop import \
  --connect ...\
  --query="select * from orders join order_items on orders.order_id = order_items.order_item_order_id where \$CONDITIONS" \
  --target-dir /user/cloudera/sqoop_import/retail.db/order_join \
  --split-by order_id \ # We must specify the column with the index (Primary, foreign)
  --num-mappers 4
```

```
sqoop import \
  --connect ...\
  --columns order_item_order_id,order_item_id,order_item_subtotal \

```


```
sqoop import \
  --connect ...\
  --where "order_date like '2014-02%'" \
  --where "product_id > 10000" can select a subset from the table that gets imported \


```


Getting delta (--where) : if there are new records in mysql database that need to be stored in HDFS.  \
--append option is used to avoid the failure (department folder already exists). \
N+1 is the index of the first new record to be stored in HDFS.\

```
sqoop import \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username=retail_dba \
  --password=cloudera \
  --table departments \
  --target-dir /user/cloudera/sqoop_import/retail.db/departments \
  --append \
  --where "department_id > N" \ 
  --outdir java_files
```


# Incremental loads

```
sqoop import \
  --connect - username - password -target-dir 
  --query "select * from orders where \$CONDITIONS and order_date like '2013-%'" \
  --split-by order_id

sqoop import \
  --connect - username - password -target-dir 
  --query "select * from orders where \$CONDITIONS and order_date like '2014-01%'" \
  --split-by order_id \
  --append
```


```
sqoop import \
  --connect - username - password -target-dir 
  --table orders \
  --where "order_date like '2014-02%'" \
  --append
```

```
sqoop import \
  --connect - username - password -target-dir 
  --table orders \
  --check-column order_date \
  --incremental append \
  --last-value '2014-02-28'

```




# Check MySQL

From command line, access a MySQL on localhost via

```
mysql -h localhost -u root -p
mysql -u retail_dba -p
mysql -h localhost -u retail_dba -p
```


Sqoop needs permissions to access the tables. 
You grant these permissions from MySQL like so:

```
GRANT ALL PRIVILEGES ON <database_name>.* to ''@'localhost';
```


```
show databases;  -- list all available databases
use retail_db;   -- switch to this database
show tables;
select * from customers limit 10;  -- query the table
```

```
create table products_replica as select * from products
alter table products_replica add primary key (product_id);
alter table products_replica add column (product_grade int, product_sentiment varchar(100))
update products_replica set product_grade = 1  where product_price > 500;
update products_replica set product_sentiment  = 'WEAK'  where product_price between 300 and  500;
```

To export a table, you must first create it in MySQL. 

```
create table retail_db.result(
	order_date varchar(255) not null,
	order_status varchar(255) not null, 
	total_orders int, 
	total_amount numeric, 
	constraint pk_order_result primary key (order_date,order_status)); 
```



# Hive


Default hive import behavior: 
---Create table if table does not exists
---If table already exists, data will be appended.

If we want to overwrite it:
-–-hive-overwrite: will replace existing data with new set of data


```
sqoop import-all-tables \
  --connect -username -password 
  --hive-import \
  --hive-overwrite \
  --hive-database "sqoop_import_test" #You can choose the database tou want
  --compress \
  --compression-codec org.apache.hadoop.io.compress.SnappyCodec \
  --outdir java_files
```

If table does not exist, we could add --hive-create-table.
–- create-hive-table: will fail hive import, if table already exists
-- Sqoop stores data in a temporary directory called the staging table under the user's home directory :  /user/cloudera before copying data into Hive table. cloudera => /user/cloudera/department should not exists. 
The temporary directory is removed after the job is done.

```
sqoop import 
 --connect -username -password 
 --table=departments 
 --hive-home=/user/hive/warehouse 
 --hive-import 
 --hive-overwrite 
 --create-hive-table 
 --hive-table=sqoop_import.departments
 --outdir java_files

```

–-hive-database can be used to specify the database
--Instead of –hive-database, we can use database name as prefix as part of –hive-table


# Export

```
sqoop export
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera \
  --table departments \
  --export-dir /user/cloudera/sqoop_export/departments  \
  --export-dir /user/hive/warehouse/<database_name>/<table_name> (if export form Hive internal table)  \
  --fields-terminated-by '\0001' (default field delimiter is ASCII value 1 there.) \
  --input-null-string   "null" \
  --input-null-non-string "null"  (to recode from the corresponding values to NULL in the database.) \
  --update-mode allowinsert  (to insert while updatin)\
  --update-key product_id   (--update key {primary_key} to update rows)\
  --columns "col1, col2, col3" \
  --num-mappers 2 \
  --batch \
  --outdir java_files
```

# Sqoop merge

```
sqoop merge \
--class-name products_replica \
--jar-file /tmp/sqoop-cloudera/compile/66b4f23796be7625138f2171a7331cd3/products_replica.jar \
--new-data /user/cloudera/problem5/products-text-part2 \
--onto /user/cloudera/problem5/products-text-part1 \
--target-dir /user/cloudera/problem5/products-text-both-parts \
--merge-key product_id;
```


# Sqoop jobs

```
sqoop job --create sqoop_job \
  -- import \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username=retail_dba \
  --table departments \
  --target-dir /user/cloudera/sqoop_import/departments \
  --fields-terminated-by '|' \
  --lines-terminated-by '\n' \
  --outdir java_files
```

`sqoop job --list`

`sqoop job --show sqoop_job`

`sqoop job --exec sqoop_job`








