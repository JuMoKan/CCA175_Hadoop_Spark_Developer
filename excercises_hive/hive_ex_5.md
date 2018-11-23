Create a hive meta store database named retail_db and import all tables from mysql retail_db database.  


Rank products within department by price and order by department ascending and rank descending.  


Find top 10 customers with most unique product purchases.  
If more than one customer has the same number of product purchases then the customer with the lowest customer_id will take precedence.  
Then create table with details of the products purchased by top 10 customers, which are priced at less than 100 USD per unit.   

Store the results in new meta store tables within hive. 



This only creates data in hdfs:
```
sqoop-import-all-tables \
--connect "jdbc:mysql://localhost/retail_db" \
--username retail_dba \
--password cloudera \
--warehouse-dir user/hive/warehouse/retail_db
```

```
create database retail_db;
#--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \

sqoop-import-all-tables \
--connect "jdbc:mysql://localhost/retail_db" \
--username retail_dba \
--password cloudera \
--warehouse-dir /user/hive/warehouse/retail_db \
--hive-import \
--hive-database retail_db

hadoop fs -ls hdfs://quickstart.cloudera:8020/user/cloudera/user/hive/warehouse/retail_db
```



Rank products within department by price and order by department ascending and rank descending.

```
select d.*, p.product_id, p.product_category_id, p.product_price, c.*, 
    rank() OVER (PARTITION BY department_id ORDER BY product_price desc) as rank_by_dep_by_price
from products p
    join categories c
    on p.product_category_id = c.category_id
    join departments d
    on d.department_id = c.category_department_id
order by d.department_id asc, rank_by_dep_by_price desc
;
```


Find top 10 customers with most unique product purchases. 
If more than one customer has the same number of product purchases then the customer with the lowest customer_id will take precedence.

```
CREATE TABLE  retail_db.top_cust AS
select o.order_customer_id as top_cust_customer_id, count(distinct oi.order_item_product_id) as N_dis_prod
from orders o
    join order_items oi
    on o.order_id = oi.order_item_order_id
group by o.order_customer_id
order by N_dis_prod desc, o.order_customer_id asc
limit 10
;
```

Then extract details of products purchased by top 10 customers which are priced at less than 100 USD per unit. 

```
select distinct p.*
from orders o
    join order_items oi
    on o.order_id = oi.order_item_order_id
    join products p
    on p.product_id = oi.order_item_product_id
where p.product_price < 100 and o.order_customer_id in (select top_cust_customer_id from top_cust)
;
```

Store the results in new meta store tables within hive. 

```
CREATE TABLE  retail_db.... AS
```
