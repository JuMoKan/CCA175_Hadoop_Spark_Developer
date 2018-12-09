Ĺessons learned:
* sqoop import: no blank after \
* in sql-queries: use " or '
* enable hive-metastore in spark: sudo ln -s /usr/lib/hive/conf/hive-site.xml /usr/lib/spark/conf/hive-site.xml
* für sum/count/rank () over (partitioned by .. ): hiveContext nicht sqlContext nutzen
* Path to compression codecs: Cloudera Product Documentation --> CDH --> All Catgories --> Compression --> Data Compression 
* Link zu compression codecs: https://www.cloudera.com/documentation/enterprise/6/6.0/topics/introduction_compression.html#concept_wlk_hgy_pv


Ĺessons learned data I/O


* textfiles
    * sc.textFile("/user.. ): /user not user, dont forget the / 
    * read textfile to df: Spark SQL, DataFrames and Datasets Guide --> Interoperating with RDDs --> Inferring the Schema Using Reflection
    * write df to textfile: df.repartition(1).rdd.map(lambda x: str(x[0])+ "," + x[1] + "," + x[2]) (kein Beispiel auf Spark Documentation gefunden)
    * check order of columns str(x[i]) or not !


* avro-files
    * sqlContext.setConf("spark.sql.avro.compression.codec","snappy");
    * df = sqlContext.read.format("com.databricks.spark.avro").load("/tmp/file.avro")
    * df.write.format("com.databricks.spark.avro").save("/tmp/output");


* set compression codec:  sqlContext.setConf(“spark.sql.parquet.compression.codec”,”gzip”), see: Spark SQL, DataFrames and Datasets Guide --> Parquet Files --> Configuration; Unter Upgrading from Spark SQL 1.3 to 1.4 Beispiel wie man sqlContext.setConf("spark.sql.retainGroupColumns", "false") settet

* read/write sequence files: rdd.read bzw. rdd.write  products.repartition(1).rdd.map(lambda x:  (str(x[0]), str(x[0])+","+str(x[1])+","+x[2]+","+x[3]+","+str(x[4])+str(x[5]))).saveAsSequenceFile("/user/cloudera/convert_ex3/sequence")

* convert to date
    *from pyspark.sql.types import * 
    *from pyspark.sql.functions import *
    *orders = orders.withColumn('order_date', to_date(orders.order_date))
