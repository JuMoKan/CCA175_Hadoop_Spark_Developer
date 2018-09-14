
## Excercise: sqoop import parquet -- select products by price -- save as json 

Import mysql orders table into hdfs (/user/cloudera/products) as parquet file.  
Select all products over 200$ and save as json file using snappy compression.   
Save to /user/cloudera/products_o200  

### Step 1: Import from mysql to hdfs

```
sqoop import --connect "jdbc:mysql://localhost/retail_db" --username root --password cloudera \
  --table products \
  --target-dir /user/cloudera/products \
  --as-parquetfile
```


### Step 2: Select products over 200$ 

```
products = sqlContext.read.parquet('/user/cloudera/products')

products.registerTempTable("products")

products_o200 = sqlContext.sql("""
	select *
	from products
	where product_price > 200
""")

```


### Step 3: save as json

```
codec = "org.apache.hadoop.io.compress.GzipCodec"
products_o200.write.json("/user/cloudera/products_o200")

To do: check again
products_o200.toJSON.saveAsTextFile("/user/cloudera/products_o200,codec )
```