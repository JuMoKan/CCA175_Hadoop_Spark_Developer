### Get information about databases, tables and data 
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

### Sqoop eval: run sql queries
```
sqoop eval \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera \
  --query "SELECT * FROM orders LIMIT 5"
  --query "CREATE TABLE dummy ...."
  --query "INSERT INTO orders VALUES (...)"

```


### Sqoop import all tables
```
sqoop import-all-tables \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera \
  --warehouse-dir /user/cloudera/sqoop_import
```

sqoop import-all-tables \
-- exclude-tables, will facilitate to exclude the tables that need not be imported\
-- auto-reset-to-one-mapper, will let import all tables to choose one mapper in case table does not have primary key\
-- Most of the features such as –query, –boundary-query, –where etc are not available with import-all-tables.\


### Sqoop import 
```
sqoop import \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera \
  --table departments \
  --target-dir /user/cloudera/sqoop_import/retail.db/departments \
  --where "id > 400"
```

Import options \
--fields-terminated-by '|' : changes the field delimiter from the default \t to another one. \
--lines-terminated-by '\n' \
--null-non-string -99 : to recode NULL values from the database into -99 for non-string data types. \
--null-string "NA" : the same option for strings \
--null-non-string und null-string: If not specified, then the string "null" will be used.\
--num-mappers 1, default is 4 \
--driver is only relevant if the connection string does not begin with jdbc:mysql://. \

File-formats: text file (default)\
--as-sequencefile \
--as-avrodatafile \
--as-parquetfile

target-dir vs. warehouse-dir \
https://stackoverflow.com/questions/37415989/difference-between-warehouse-dir-and-target-dir-commands-in-sqoop/48277876


### Compression

Compression codecs: GzipCodec, SnappyCodec, BZip2Codec, DefaultCodec(?)\
Default: gzip\
https://hadoop.apache.org/docs/r1.2.1/api/org/apache/hadoop/io/compress/package-summary.html

```
--compress (oder z) \
--compression-codec org.apache.hadoop.io.compress.GzipCodec
--compression-codec org.apache.hadoop.io.compress.SnappyCodec
--compression-codec org.apache.hadoop.io.compress.BZip2Codec

```



### Hive

hive-import:
---hive-import: will both create table in Hive and import data from the source table.
---Create table, if table does not exists
---If table already exists, data will be appended.

---hive-overwrite: will replace existing data with new set of data

---hive-database can be used to specify the database
---Instead of –hive-database, we can use database name as prefix as part of –hive-table

-- If you have multiple Hive installations, or hive is not in your $PATH, use the --hive-home option to identify the Hive installation directory.
   --hive-home=/user/hive/warehouse 


create-hive-table 
--- create-hive-table will create table in Hive based on the source table in database but will NOT transfer any data
--- create-hive-table: will fail hive import, if table already exists
--- Sqoop stores data in a temporary directory called the staging table under the user's home directory: /user/cloudera before copying data into Hive 
    table. cloudera => /user/cloudera/products  should not exists. 
    The temporary directory is removed after the job is done.


(see https://stackoverflow.com/questions/31515498/when-to-use-sqoop-create-hive-table)

```
 --outdir java_files (??)


sqoop import
  --connect -username -password 
  --table products 
  --warehouse-dir /user/cloudera/problem3/products_replica/input 
  --hive-import 
  --hive-database default 
  --hive-table product_replica 
   -m 1


sqoop import
  --connect -username -password 
  --table products 
  --create-hive-table 


sqoop create-hive-table 
--connect jdbc:mysql://localhost:3306/hadoopexample 
--table employees 


```

Sonderfall: import-all-tables
```
sqoop import-all-tables \
  --connect -username -password 
  --hive-import \
  --hive-overwrite \
  --hive-database "sqoop_import_test" #Select database
  --compress \
  --compression-codec org.apache.hadoop.io.compress.SnappyCodec \
  --outdir java_files
```


### Sqoop import to existing directories: append or –delete-target-dir

By default sqoop import fails if target directory already exists \
Directory can be overwritten by using –delete-target-dir\
Data can be appended to existing directories by saying –append

```
sqoop import \
  --connect --username --password \
  --table order_items \
  --warehouse-dir /user/cloudera/sqoop_import/retail_db \
  --delete-target-dir
```


```
sqoop import \
  --connect --username --password \
  --table order_items \
  --warehouse-dir /user/cloudera/sqoop_import/retail_db \
  --append
```


### Split-by / boundary-query

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


### Select columns, import query results, use where

```
sqoop import \
  --connect ...\
  --columns "order_item_order_id,order_item_id,order_item_subtotal" \
```

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
  --where "order_date like '2014-02%'" \
  --where "product_id > 10000" can select a subset from the table that gets imported \


```


### Incremental loads

Getting delta (--where) : if there are new records in mysql database that need to be stored in HDFS.  \
--append option is used to avoid the failure (departments folder already exists). \
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
```

```
sqoop import \
  --connect - username - password -target-dir 
  --table orders \
  --where "order_date like '2014-02%'" \
  --append
```


\$CONDITIONS and split-by


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


check-column & incremental append & last-value

```
sqoop import \
  --connect - username - password -target-dir 
  --table orders \
  --check-column order_date \
  --incremental append \
  --last-value '2014-02-28'

```



### Export

```
sqoop export
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera \
  --table departments \
  --export-dir /user/cloudera/sqoop_export/departments  \
  --export-dir /user/hive/warehouse/<database_name>/<table_name> (export Hive internal table)  \
```
Export hive table
* export hive internal table, export-dir: /user/hive/warehouse/...
* export hive table: --hcatalog-table name_of_hive_table


```
  --fields-terminated-by '\0001' (when exporting from Hive, in Hive default field delimiter is ASCII value 1) \
  (to recode from the corresponding values to NULL in the database.)
  --input-null-string   "null" \
  --input-null-non-string "null" \
  --update-mode allowinsert  (to insert while updatin)\
  --update-key product_id   (--update key {primary_key} to update rows)\
  --columns "col1, col2, col3" \
  --num-mappers 2 \
  --batch \
```



### Sqoop merge
```
sqoop merge \
--class-name products_replica \
--jar-file /tmp/sqoop-cloudera/compile/66b4f23796be7625138f2171a7331cd3/products_replica.jar \
--new-data /user/cloudera/problem5/products-text-part2 \
--onto /user/cloudera/problem5/products-text-part1 \
--target-dir /user/cloudera/problem5/products-text-both-parts \
--merge-key product_id;
```


### Sqoop jobs

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








