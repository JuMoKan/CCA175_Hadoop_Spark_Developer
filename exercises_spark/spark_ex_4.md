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
products = parts.map(lambda p: Row(product_id=float(p[0]), product_category_id=int(p[1]), product_price=float(p[2]))).toDF()
products.show(10)
products.registerTempTable('products')
```

```
products_category = sqlContext.sql("""
select product_category_id,
MIN(product_price) as min_price_in_category,
MAX(product_price) as max_price_in_category,
AVG(product_price) as avg_price_in_category,
count(distinct product_id) as n_products_in_category
from products
group by product_category_id
""")

products_category.show(10)
```

