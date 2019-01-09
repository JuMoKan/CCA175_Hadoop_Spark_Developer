
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
products.filter(products.product_price > 200).toJSON()\
.saveAsTextFile("/user/cloudera/products_o200", compressionCodecClass="org.apache.hadoop.io.compress.GzipCodec")

Snappy-Compress did not work
products.filter(products.product_price > 200).toJSON()\
.saveAsTextFile("/user/cloudera/products_o200_snappy", \
compressionCodecClass="org.apache.hadoop.io.compress.SnappyCodec")

Did not work either (wrote json, no gzip compression)
sqlContext.setConf("spark.sql.json.compression.codec", "gzip") 
products.filter(products.product_price > 200).write.json("/user/cloudera/products_test_gzip")
```