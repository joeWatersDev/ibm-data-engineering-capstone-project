# MySQL OLTP Database

You are a data engineer at an e-commerce company. Your company needs you to design a data platform that uses MySQL as an OLTP database. You will be using MySQL to store the OLTP data.

**Objectives**
- design the schema for OLTP database
- load data into OLTP database
- automate admin tasks

**Tools / Software Used**
- MySQL 8.0.22
- phpMyAdmin 5.0.4

## Database Schema
![Example entries in database for schema design](https://github.com/joeWatersDev/ibm-data-engineering-capstone-project/blob/main/1%20-%20MySQL%20OLTP%20Database/schema.png)

## 1 - Create the database and esign a table named sales_data
Create a database named sales.
```
CREATE DATABASE sales;
```
Design a table named sales_data based on the sample data given. 
Create the sales_data table in sales database.
```
CREATE TABLE “sales_data” (
	“product_id” INT NOT NULL,
	“customer_id” INT NOT NULL,
	“price” INT NOT NULL,
	“quantity” INT NOT NULL,
	“timestamp” TIMESTAMP NOT NULL ON UPDATE CURRENT_TIMESTAMP
);
```

## 2 - Import and load sample data
Data imported via phpMyAdmin web interface
![Confirmation of data import](https://github.com/joeWatersDev/ibm-data-engineering-capstone-project/blob/main/1%20-%20MySQL%20OLTP%20Database/importdata.png)

We can confirm successful load and show a count of entries in the table.
```
USE sales;
SHOW tables;
```
```
+-----------------+
| Tables_in_sales |
+-----------------+
| sales_data      |
+-----------------+
1 row in set (0.01 sec)
```
```
SELECT COUNT(*) FROM sales_data;
```
```
+----------+
| COUNT(*) |
+----------+
|     2605 |
+----------+
1 row in set (0.01 sec)
```


## 3 - Create an index
Create an index named ts on the timestamp field.
```
CREATE INDEX ts ON sales_data (timestamp);
```
We can confirm successful creation of the index with the following command.
```
SHOW INDEX FROM sales_data;
```
```
+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table      | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| sales_data |          1 | ts       |            1 | timestamp   | A         |        2605 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+------------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
1 row in set (0.00 sec)
```

## 4 - Write a bash script to export data.

First create the bash script.
```
touch datadump.sh
```

Within the script, export the sales_data records to an sql file.
```
#!/bin/bash
mysqldump -u root -p sales sales_data > sales_data.sql
```

Modify the file permissions to make it executable.
```
chmod u+x datadump.sh
```

Finally, execute the script
```
./datadump.sh