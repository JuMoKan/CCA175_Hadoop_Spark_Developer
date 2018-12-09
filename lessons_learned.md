**Ĺessons learned:**
* sqoop import: no blank after \
* enable hive-metastore in spark: sudo ln -s /usr/lib/hive/conf/hive-site.xml /usr/lib/spark/conf/hive-site.xml
* für sum/count/rank () over (partitioned by .. ): hiveContext nicht sqlContext nutzen
* Path to compression codecs: Cloudera Product Documentation --> CDH --> All Catgories --> Compression --> Data Compression 
* Link zu compression codecs: https://www.cloudera.com/documentation/enterprise/6/6.0/topics/introduction_compression.html#concept_wlk_hgy_pv


**Ĺessons learned data I/O** 


* textfiles
    * sc.textFile("/user.. ): dont' forget the / before user
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

* convert to date
    * from pyspark.sql.types import * 
    * from pyspark.sql.functions import *
    * orders = orders.withColumn('order_date', to_date(orders.order_date))


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
