# Excercise:

Import mysql products table into hive products table (default database) 
and import into hdfs () as parquet format, lines terminated by ;

https://stackoverflow.com/questions/42107835/sqoop-import-as-parquet-file-to-target-dir-but-cant-find-the-file
https://community.hortonworks.com/questions/193934/sqoop-import-mssql-table-into-hdfs.html

### Only hive, not hdfs import
```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--table products \
--hive-import \
--create-hive-table \
--hive-table default.products
```

### hive and hdfs import
Please do not use <db>.<table> with --hive-table. This doesn't work well with Parquet import.
https://stackoverflow.com/questions/27861924/sqoop-import-as-parquetfile-with-cdh5/30473203#30473203

warehouse-dir /user/cloudera/hive_ex_2 &  --as-parquetfile  didn't work
* no folder/output in hdfs, hive table created
* created 4 parquet files in hadoop fs -ls /user/hive/warehouse/products

```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--table products \
--warehouse-dir /user/cloudera/hive_ex_2 \
--hive-import \
--create-hive-table \
--hive-database default \
--hive-table products \
--as-parquetfile 
```

warehouse-dir /user/cloudera/hive_ex_2 &  --as-parquetfile  & -m1 
* no folder/output in hdfs, hive table created
* created parquet file in /user/hive/warehouse/products

```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--table products \
--warehouse-dir /user/cloudera/hive_ex_2 \
--hive-import \
--create-hive-table \
--hive-database default \
--hive-table products \
--as-parquetfile \
-m1
```


warehouse-dir /user/cloudera/hive_ex_2 & as text-file:
* ohne -m 1: created empty user/cloudera/hive_ex_2 , hive-table, 4 text-files in /user/hive/warehouse/products
* mit -m 1: created hive table und empty /user/cloudera/hive_ex_2
```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username root \
--password cloudera \
--table products \
--warehouse-dir /user/cloudera/hive_ex_2 \
--hive-import \
--create-hive-table \
--hive-database default \
--hive-table products 
```
