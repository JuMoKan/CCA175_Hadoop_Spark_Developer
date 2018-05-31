```
sqoop list-databases \
  --connect "jdbc:mysql://quickstart.cloudera:3306" \
  --username retail_dba \
  --password cloudera
```
---------------------------------

```
sqoop list-tables \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera
```

```
sqoop eval \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username retail_dba \
  --password cloudera \
  --query "select count(1) from order_items"
```

```
sqoop import-all-tables \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username=retail_dba \
  --password=cloudera \
  --warehouse-dir=/user/cloudera/sqoop_import
```

```
sqoop import \
  --connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
  --username=retail_dba \
  --password=cloudera \
  --table departments \
  --target-dir /user/cloudera/sqoop_import/retail.db/departments
```

#If we want to change the character that seperates fields and the one seperating lines :
```
--fields-terminated-by '|' \
  --lines-terminated-by '\n' \
```
