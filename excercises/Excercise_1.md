# Question 4:
Import orders table from mysql into hdfs location /user/cloudera/practice4/question3/orders/.
sqoop import --connect "jdbc:mysql://localhost/retail_db" --username root --password cloudera --table orders --target-dir /user/cloudera/practice4/question3/orders/

Import customers from mysql into hdfs location /user/cloudera/practice4/question3/customers/. 
sqoop import --connect "jdbc:mysql://localhost/retail_db" --username root --password cloudera --table customers --target-dir /user/cloudera/practice4/question3/customers/

# Instructions:
Join data at hdfs  /user/cloudera/practice4/question3/orders/ & /user/cloudera/practice4/ question3/customers/ to find out customers whose orders status is "pending"
I translate this as: For every customer, who has a minimum one order pending, list all pending orders.
Output Requirement:
Output should have customer_id, customer_fname, order_id and order_status.                                            Result should be saved as textfile in /user/cloudera/p1/q7/output

