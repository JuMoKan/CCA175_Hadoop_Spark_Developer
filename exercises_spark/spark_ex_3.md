## Excercise: sqoop import into hive – spark rank by order by – export into hdfs

Import mysql customers table into hive table customers.  
Rank cities within each state by number of customers and order by state name ascending and rank descending.  
Output: state, city, no_customers and rank_no_customers.  
Save result as textfile into /user/cloudera/rank_cities_by_size_by_state.  

### Step 1: Get the data from mysql to hive

```
sqoop import --connect "jdbc:mysql://localhost/retail_db" --username root --password cloudera \
  --table customers \
  --hive-import  \
  --hive-table default.customers
```


### Step 2: Get the data from mysql to hive

```
from pyspark.sql import pyspark.sql.HiveContext
hive_context = HiveContext(sc)

customers_by_state_city = hive_context.sql("""
	select customer_state, customer_city, count(*) as no_cust,
	from customers
	group by customer_state, customer_city
	order by customer_state asc, count(*) desc 
""")

customers_by_state_city.registerTempTable("customers_by_state_city")

customers_by_state_city_rank = hive_context.sql("""
	select customer_state, customer_city, no_cust,
		rank() over (PARTITION BY customer_state ORDER BY no_cust desc) as rank
	from customers_by_state_city
	order by customer_state, rank asc 
""")
```



