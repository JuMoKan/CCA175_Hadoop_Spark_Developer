Import customers table from mysql to hdfs to /user/cloudera/io_ex3/import.    
Count customers by state.  
Create table in mysql co_cust_by_state(columns state and No_cust).  
Export result cústomers by state to mysql table.  


```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--password cloudera \
--username root \
--table customers \
--columns "customer_id,customer_state" \
--target-dir /user/cloudera/io_ex3/import \
--as-parquetfile	
```


```
sqlContext = SQLContext(sc)
customers_id_state = sqlContext.read.parquet("/user/cloudera/io_ex3/import")

customers_id_state.registerTempTable('customers_id_state')

no_customers_by_state = sqlContext.sql("""
    select customer_state, count(customer_id) as No_cust
    from customers_id_state
    group by customer_state
""")

no_customers_by_state.head(20)

no_customers_by_state.write.parquet("/user/cloudera/io_ex3/no_cust_by_state")

no_customers_by_state.repartition(1).rdd.map(lambda x: x[0]+"," +str(x[1])).saveAsTextFile("/user/cloudera/io_ex3/no_cust_by_state_text")
```


in mysql
```
create table no_cust_by_state(customer_state varchar(20), No_cust int);

sqoop export \
--connect "jdbc:mysql://localhost/retail_db" \
--password cloudera \
--username root \
--export-dir /user/cloudera/io_ex3/no_cust_by_state_text \
--table no_cust_by_state 
```

```
select count(*) from no_cust_by_state;
```