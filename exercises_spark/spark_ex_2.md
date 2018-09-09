# Excercise:

Import customers table from mysql into hdfs `/user/cloudera/ex_2/customers/` as parquet file.  
Import only customer_id and customer_city.

Count number of customers grouped by customer city, order the results by size (descending).  
Save the result as text file with fields separated by $ in `/user/cloudera/ex_2/city_no_cust`.



### Step 1: Get the data from mysql to hdfs  
```
sqoop import --connect "jdbc:mysql://localhost/retail_db" --password cloudera --username root \
  --table customers  \
  --columns "customer_id, customer_city" \
  --target-dir /user/cloudera/ex_2/customers/ \
  --as-parquetfile
```


### Step 2: Read in data and registerTempTable

```
#from pyspark import SparkConf, SparkContext
#conf = SparkConf().setMaster("local").setAppName("Hier_der_AppName")
#sc = SparkContext(conf = conf)
#Use sqlContext.read.parquet

customers=sqlContext.read.parquet("/user/cloudera/ex_2/customers/")
customers.registerTempTable("customers")
```

### Step 3: Create df with number of customers grouped by city
```
city_no_cust = sqlContext.sql("""
	SELECT customer_city,  count(*) as No
	FROM customers 
	GROUP BY customer_city
	ORDER BY No desc
""")
```


### Step 4: Save as text file
```
#Options: saveAsTextFile("hdfs:///user/cloudera/.../city_no_cust.txt")
city_no_cust.repartition(1).rdd\
	.map(lambda x: str(x[0]) + "$" + str(x[1]))\
	.saveAsTextFile("/user/cloudera/ex_2/city_no_cust")
	
hdfs dfs -get /user/cloudera/ex_2/city_no_cust /home/cloudera/hdfs_output
```

   



 
