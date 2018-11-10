Copy mysql products table to hdfs /user/cloudera/products  as text file.  
Columns should be delimited by pipe ‘|’.  

Return for the products with a price < 200 Dollar  
•	the average, lowest and highest price within each category  
•	number of total products within each category.  

Save file as text in /user/cloudera/products_category description.  

```
sqoop import \
--connect "jdbc:mysql://localhost/retail_db" \
--username retail_dba \
--password cloudera \
--table products \
--target-dir products \
--columns "product_id, product_category_id, product_price" \
--fields-terminated-by "|"
```

```
from pyspark.sql import SQLContext, Row
sqlContext = SQLContext(sc)

lines = sc.textFile("/user/cloudera/products")
parts = lines.map(lambda l: l.split("|"))
products = parts.map(lambda r: Row(product_id=float(r[0]), product_category_id=int(r[1]), product_price=float(r[2]))).toDF()
products.show(10)
products.registerTempTable('products')
# products.createTempView("products") ## Spark > 1.6
# sqlContext.registerDataFrameAsTable(products, "products")

```

```
products_category = sqlContext.sql("""
select product_category_id,
MIN(product_price) as min_price_in_category,
MAX(product_price) as max_price_in_category,
cast(avg(product_price) as decimal(10,2)) as avg_price_in_category,
count(distinct product_id) as n_products_in_category
from products
group by product_category_id
order by product_category_id desc
""")

products_category.show(10)
```


Dataframes API

```
# products.agg({'productPrice': 'min','productPrice': 'max'}).show() 
# das gibt nur max aus

from pyspark.sql import functions as F

products\
    .filter(products['productPrice']<100)\
    .groupby('productCatID')\
    .agg(F.count(products.productID).alias('N_Products'),\
	     F.min(products.productPrice).alias('Min_Price'),\
	     F.max(products.productPrice).alias('Max_Price'))\
	.sort('productCatID', asending=True).show()

```

