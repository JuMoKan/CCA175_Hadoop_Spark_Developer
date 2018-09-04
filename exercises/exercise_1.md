# Excercise:
Import orders and customers table from mysql into hdfs  location `/user/cloudera/ex_1/orders/`  `/customers/`.   
Join data to find out customers, whose orders status is "pending".  
Output should have customer_id, customer_fname, order_id and order_status.                                            
Save result as textfile in /user/cloudera/exc_1/output  



## Step 1: Get the data from mysql to hdfs  
```
sqoop import --connect "jdbc:mysql://localhost/retail_db" --username root --password cloudera \
  --table orders  \
  --target-dir /user/cloudera/ex_1/orders/
```

```
sqoop import --connect "jdbc:mysql://localhost/retail_db" --username root --password cloudera  \
  --table customers \
  --target-dir /user/cloudera/ex_1/customers/
```

Check in hadoop: `hadoop fs -ls`


## Step 2: Read data in pyspark.dataframe

```
from pyspark.sql import Row 

customers = sc.textFile("/user/cloudera/ex_1/customers/") \
    .map(lambda l: l.split(",")) \
    .map(lambda l: Row(customer_id=int(l[0]) 
    				  ,customer_fname = l[1] 
					  ,customer_lname  = l[2] 
					  ,customer_email = l[3] 
					  ,customer_password = l[4] 
					  ,customer_street = l[5] 
					  ,customer_city = l[6]
					  ,customer_state = l[7]
					  ,customer_zipcode = l[8]))\
	.toDF()


orders = sc.textFile("/user/cloudera/ex_1/orders/") \
    .map(lambda l: l.split(",")) \
    .map(lambda l: Row(  order_id=int(l[0]) 
    				    ,order_date = l[1]
						,order_customer_id= int(l[2]) 
						,order_status =l[3]))\
	.toDF()

from pyspark.sql.types import * 
from pyspark.sql.functions import *
orders = orders.withColumn('order_date', to_date(orders.order_date))


```

## Step 3: Join as SQL Statement  
```
customers.registerTempTable("customers")
orders.registerTempTable("orders")


order_status = sqlContext.sql("""
	SELECT  Distinct order_status 
	From orders
""")

cust_orders_pend = sqlContext.sql("""
	SELECT  Distinct c.customer_id, c.customer_fname, o.order_id, o.order_status 
	From customers c
	INNER JOIN orders o
	ON c.customer_id = o.order_customer_id
	WHERE o.order_status = "PENDING"
""")

```


## Step 4: Save  

