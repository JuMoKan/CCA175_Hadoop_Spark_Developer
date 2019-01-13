**Ĺessons learned:**


* Hive Operators and User-Defined Functions: select concat(customer_fname,'\t',customer_lname,':',customer_city) from ... where ..like '%Text%'

* enable hive-metastore in spark: sudo ln -s /usr/lib/hive/conf/hive-site.xml /usr/lib/spark/conf/hive-site.xml
    * http://community.cloudera.com/t5/Advanced-Analytics-Apache-Spark/how-to-access-the-hive-tables-from-spark-shell/m-p/36653
    * https://stackoverflow.com/questions/36051091/query-hive-table-in-pyspark


* für sum/count/rank () over (partitioned by .. ): hiveContext nicht sqlContext nutzen

* sqoop import: no blank after \
* sqoop export
    * make sure mysql table has same data-type as export-table
    * --input-fields-terminated-by "\t" (falls spezielle field delimiter)

* Achtung: sqlContext nur einmal öffnen / definieren, sonst werden Tabllen nicht registriert  
    df.registerTempTable("df")  
    all_tables = sqlContext.tables()  
    all_tables.show()

* Compression codecs: 
    * Cloudera Product Documentation --> CDH --> All Catgories --> Compression --> Data Compression 
    * Link: https://www.cloudera.com/documentation/enterprise/6/6.0/topics/introduction_compression.html#concept_wlk_hgy_pv

* chmod: 1: execute only, 2: write only, 3: write and execute, 4  read only chmod 765 /user/../*
* parts.map(lambda p: Row(product_id = int(p[0]), --> make sure to start with 0 p[0]

* mysql
    * mysql -uroot -pcloudera  
    * mysql -h localhost -u retail_db -p
    * select User, Host from mysql.user;
    * GRANT ALL PRIVILEGES ON DataBaseName.* to ''@'localhost';
    * create table MYsqlDATABASE.Table(
      11.2.2 Fixed-Point Types (Exact Value) - DECIMAL, NUMERIC
      11.2.3 Floating-Point Types (Approximate Value) - FLOAT, DOUBLE)



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

* get avro schema
    * hdfs dfs -get /user/hive/warehouse/retail_db.db/orders/part-m-00000.avro
    * avro-tools getschema part-m-00000.avro > orders.avsc
    * hdfs dfs -mkdir /user/hive/schemas
    * hdfs dfs –put orders.avsc /user/hive/schemas

```
create external table table_from_avro
stored as avro
location '/user/hive/warehouse/folder_with_avro'
TBLPROPERTIES ('avro.schema.url'='user/hive/schemas/orders.avsc');
```

* json files: set compression codec
    * orders_avro_snappy.toJSON().saveAsTextFile("/user/cloudera/problem5/json-gzip2", compressionCodecClass="org.apache.hadoop.io.compress.GzipCodec")
    * did not work: sqlContext.setConf("spark.sql.json.compression.codec", "gzip") orders_avro_snappy.write.json("/user/cloudera/problem5/json-gzip")

* set compression codec: 
    * sqlContext.setConf(“spark.sql.parquet.compression.codec”,”gzip”)
    * see: Spark SQL, DataFrames and Datasets Guide --> Parquet Files --> Configuration; 
    * Unter Upgrading from Spark SQL 1.3 to 1.4: Beispiel setConf sqlContext.setConf("spark.sql.retainGroupColumns", "false") 

* read/write sequence files: key-value pairs
    * rdd.read bzw. rdd.write  
    * df.repartition(1).rdd.map(lambda x: (str(x[0]), str(x[0])+ "," + str(x[1]) + "," + x[2])).saveAsSequenceFile("/user/cloudera/convert_ex3/sequence")

* csv: df.write.format("com.databricks.spark.csv").save(path)


**Spark: convert to date**

    * from pyspark.sql.types import * 
    * from pyspark.sql.functions import * 
    * orders = orders.withColumn('order_date', to_date(orders.order_date)) 
    * orders = orders.withColumn('order_date', from_unixtime(orders.order_date/1000)) 


**hive rank() over () function**  
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








