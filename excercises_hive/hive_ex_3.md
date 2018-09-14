## Export hive table into mysql

Create hive table customers_anonym select customer_id, customer_city, customer_state, customer_zipcode.  
(customers table already exists in hive).  
Create customers_anonym table in mysql.  
Export hive table customers_anonym to customers_anonymus table in mysql.   

### Create hive table customers_anonym
```
CREATE TABLE customer_anonym  as
    select customer_id, customer_city, customer_state, customer_zipcode
    from customers;
```

### Create (empty) customers_anonym table in mysql
```
create table retail_db.customer_anonym(
    customer_id int,
    customer_city varchar(255), 
    customer_state varchar(255), 
    customer_zipcode varchar(255),
    constraint pk_order_result primary key (customer_id));
```


## Export hive table to mysql table: --hcatalog-table
export hive internal table, export-dir: /user/hive/warehouse/...
export hive table: --hcatalog-table name_of_hive_table

```
sqoop export \
--connect jdbc:mysql://localhost/retail_db \
--username root \
--password cloudera \
--table customer_anonym \
--hcatalog-table customer_anonym
```

Links  
<!--- http://www.alpha-epsilon.de/cca175/2017/07/21/using-sqoop-to-move-data-between-hdfs-and-mysql--->
http://hadooped.blogspot.com/2013/06/apache-sqoop-part-3-data-transfer.html  
https://community.hortonworks.com/questions/22425/sqoop-export-from-hive-table-specifying-delimiters.html
