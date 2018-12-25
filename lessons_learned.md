**Ĺessons learned:**
* sqoop import: no blank after \
* enable hive-metastore in spark: sudo ln -s /usr/lib/hive/conf/hive-site.xml /usr/lib/spark/conf/hive-site.xml
* für sum/count/rank () over (partitioned by .. ): hiveContext nicht sqlContext nutzen
* Achtung: sqlContext nur einmal öffnen / definieren, sonst werden Tabllen nicht registriert
    df.registerTempTable("df")
    all_tables = sqlContext.tables()
    all_tables.show()

* Path to compression codecs: Cloudera Product Documentation --> CDH --> All Catgories --> Compression --> Data Compression 
* Link zu compression codecs: https://www.cloudera.com/documentation/enterprise/6/6.0/topics/introduction_compression.html#concept_wlk_hgy_pv


**Ĺessons learned data I/O** 

* textfiles
    * text_from_hdfs_lines = sc.textFile("/user/..") dont' forget the / before user
    * read: Spark SQL, DataFrames and Datasets Guide --> Interoperating with RDDs --> Inferring the Schema Using Reflection
    * after read / before write check order of columns str(x[i]) or not !
    * write: df.repartition(1).rdd.map(lambda x: str(x[0])+ "," + x[1] + "," + x[2])


* avro-files
    * sqlContext.setConf("spark.sql.avro.compression.codec","snappy");
    * df = sqlContext.read.format("com.databricks.spark.avro").load("/tmp/file.avro")
    * df.write.format("com.databricks.spark.avro").save("/tmp/output");


* set compression codec: 
    * sqlContext.setConf(“spark.sql.parquet.compression.codec”,”gzip”)
    * see: Spark SQL, DataFrames and Datasets Guide --> Parquet Files --> Configuration; 
    * Unter Upgrading from Spark SQL 1.3 to 1.4: Beispiel setConf sqlContext.setConf("spark.sql.retainGroupColumns", "false") 

* read/write sequence files: key-value pairs
    * rdd.read bzw. rdd.write  
    * df.repartition(1).rdd.map(lambda x: (str(x[0]), str(x[0])+ "," + str(x[1]) + "," + x[2])).saveAsSequenceFile("/user/cloudera/convert_ex3/sequence")



**Spark: convert to date**
    * from pyspark.sql.types import * 
    * from pyspark.sql.functions import *
    * orders = orders.withColumn('order_date', to_date(orders.order_date))
    * orders = orders.withColumn('order_date', F.from_unixtime(orders.order_date/1000))



**Ĺessons learned Spark Joins** 

```
from pyspark.sql import SQLContext, Row
sqlContext = SQLContext(sc)
name_age_rdd = sc.parallelize([("malte", 29), ("zoe", 3), ("marli", 1)])
name_age = sqlContext.createDataFrame(name_age_rdd, ['name', 'age'])

name_city_rdd = sc.parallelize([("malte", "rhein"), ("zoe", "ruhr"), ("marli", "ruhr")])
name_city = sqlContext.createDataFrame(name_city_rdd, ['name', 'city'])
```

#df_a.join(df_b, df_a.id == df_b.id): id column appears 2 times
```
join_v1 = name_city.join(name_age, name_city.name == name_age.name)   
join_v1.show()
+-----+-----+-----+---+                                                         
| name| city| name|age|
+-----+-----+-----+---+
|marli| ruhr|marli|  1|
|malte|rhein|malte| 29|
|  zoe| ruhr|  zoe|  3|
+-----+-----+-----+---+
```

#df_a.join(df_b, "id): id colum appears 1 time
```
join_v2 = name_city.join(name_age, "name")
join_v2.show()
+-----+-----+---+                                                               
| name| city|age|
+-----+-----+---+
|marli| ruhr|  1|
|malte|rhein| 29|
|  zoe| ruhr|  3|
+-----+-----+---+
```

#sql(select * from df_a join df_b on df_a.id=df_b.id: id colum appears 2 times

```
name_age.registerTempTable("name_age")
name_city.registerTempTable("name_city")

all_tables = sqlContext.tables()
all_tables.show()

join_sql = sqlContext.sql("""select * from name_age na join name_city nc on na.name=nc.name""")
join_sql.show()

```









* hive rank() over () function  
    did not work:
    ```
    top_5_products_by_category_prep = hiveContext.sql("""
    select product_category_id, product_id, product_name, product_price, rank() over (partition by product_category_id order by product_price desc) as product_rank
    from products
    order by product_category_id, rank() over (partition by product_category_id order by product_price desc)
    """)
    ```

    this worked:

    ```
    top_5_products_by_category_prep = hiveContext.sql("""
    select product_category_id, product_id, product_name, product_price, rank() over (partition by product_category_id order by product_price desc) as product_rank
    from products
    order by product_category_id, product_rank
    """)
    ```
