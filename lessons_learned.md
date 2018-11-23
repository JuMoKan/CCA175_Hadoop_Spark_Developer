Ĺessons learned:
* sqoop import: no blank after \
* in sql-queries: use " or '
* enable hive-metastore in spark: sudo ln -s /usr/lib/hive/conf/hive-site.xml /usr/lib/spark/conf/hive-site.xml
* für sum/count/rank () over (partitioned by .. ): hiveContext nicht sqlContext nutzen


Ĺessons learned data I/O
* sc.textFile("/user.. ): /user not user, dont forget the / 
* set compression codec:  sqlContext.setConf(“spark.sql.parquet.compression.codec”,”gzip”), see: Spark SQL, DataFrames and Datasets Guide --> Parquet Files --> Configuration; Unter Upgrading from Spark SQL 1.3 to 1.4 Beispiel wie man sqlContext.setConf("spark.sql.retainGroupColumns", "false") settet
* read/write sequence files: rdd.read bzw. rdd.write

* textfiles
    * read textfile to df: Spark SQL, DataFrames and Datasets Guide --> Interoperating with RDDs --> Inferring the Schema Using Reflection
    * write df to textfile: df.repartition(1).rdd.map(lambda x: str(x[0])+ "," + x[1] + "," + x[2]) (kein Beispiel auf Spark Documentation gefunden)


* avro-files
    *import com.databricks.spark.avro._;
     sqlContext.read.avro(<path to location>);
    *sqlContext.setConf(“spark.sql.avro.compression.codec”,”snappy”) //use snappy, deflate, uncompressed;
     dataFrame.write.avro(<path to location>);

