Enable hive-metastore  
sudo ln -s /usr/lib/hive/conf/hive-site.xml /usr/lib/spark/conf/hive-site.xml


http://community.cloudera.com/t5/Advanced-Analytics-Apache-Spark/how-to-access-the-hive-tables-from-spark-shell/m-p/36653  
https://stackoverflow.com/questions/36051091/query-hive-table-in-pyspark  
<!--- http://discuss.itversity.com/search?q=hive%20pyspark%20metastore
http://discuss.itversity.com/t/not-able-to-access-default-database-in-pyspark/7157
Removed the linked file using sudo rm -R /etc/spar/conf/hive.xml
Again linked the file using sudo ln -s /etc/hive/conf/hive-site.xml /etc/spark/conf/hive-site.xml
--->

sqlContext.sql("CREATE TABLE IF NOT EXISTS mytable AS SELECT * FROM customers")



