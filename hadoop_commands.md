### Help
hdfs dfs -help get


### Move from hdfs to local
hdfs dfs -get /user/cloudera/ex_2/city_no_cust /home/cloudera/hdfs_output  
hdfs dfs -copyToLocal /user/cloudera/ex_2/city_no_cust /home/cloudera/hdfs_output

### Move from local to hdfs
hdfs dfs -put /home/cloudera/hdfs_input /user/cloudera/ex_2/city_no_cust  
hdfs dfs -put -f /home/cloudera/hdfs_input /user/cloudera/ex_2/city_no_cust  (to overwrite existing files in hdfs)  
hdfs dfs -copyFromLocal /home/cloudera/hdfs_input /user/cloudera/ex_2/city_no_cust 

### Delete directory
hdfs  dfs -rm -r /user/cloudera/ex_2/city_no_cust

### Compression codecs
https://hadoop.apache.org/docs/r2.7.2/api/org/apache/hadoop/io/compress/CompressionCodec.html

### Overview File System (FS) shell: shell-like commands appendtoFile, cat, chmod, get, put, usw.
https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html
