# Question 5:
PreRequiste:
Run below sqoop command to import customer table from mysql  into hdfs to the destination /user/cloudera/problem3/customer/parquet  as parquet file. Only import customer_id,customer_fname,customer_city.
sqoop import --connect "jdbc:mysql://localhost/retail_db" --password cloudera --username root --table customers --columns "customer_id,customer_fname,customer_city"  --target-dir /user/cloudera/problem3/customer/parquet --as-parquetfile

# Instructions:
Count number of customers grouped by customer city,customer first name where customer_fname is like "Mary" and order the results  by customer first name and save the result as text file with fields separated by pipe character
Input folder is  /user/cloudera/problem3/customer/parquet.

Output Requirement:
Result should have customer_city,customer_fname and count of customers and output should be saved in /user/cloudera/problem3/customer_grouped as text file with fields separated by pipe character

