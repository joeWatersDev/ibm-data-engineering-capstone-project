# MongoDB NoSQL Databases

You are a data engineer at an e-commerce company. Your company needs you to design a data platform that uses MongoDB as a NoSQL database. You will be using MongoDB to store the e-commerce catalog data.

**Objectives**
- Import data into a MongoDB database
- Query data in a MongoDB database
- Export data from MongoDB

**Tools / Software Used**
- MongoDB Server
- MongoDB Command Line Backup Tools

## 1 - Install dependancies
Ensure 'mongoimport' and 'mongoexport' are installed.
```
wget https://fastdl.mongodb.org/tools/db/mongodb-database-tools-ubuntu1804-x86_64-100.3.1.tgz

tar -xf mongodb-database-tools-ubuntu1804-x86_64-100.3.1.tgz

export PATH=$PATH:/home/project/mongodb-database-tools-ubuntu1804-x86_64-100.3.1/bin
```

## 2 - Download and import catalog data
Download the catalog.json file from the provided source.
```
wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DB0321EN-SkillsNetwork/nosql/catalog.json
```

Import the downloaded ‘catalog.json’ to mongodb into a database named ‘catalog’ and a collection named ‘electronics’.

```
start_mongo

use catalog
db.createCollection("electronics")

mongoimport --host localhost --port 27017 --db catalog --collection electronics --authenticationDatabase admin --username root --password *password* catalog.json
```

Confirm that the database and collection were created.
```
show dbs
```
```
admin    0.000GB
catalog  0.000GB
config   0.000GB
local    0.000GB
```
```
show collections
```
```
electronics
```


## 3 - Create an index
Create an index so we can query our collection.
```
db.electronics.createIndex({"type":1})
```
```
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
```
It is now possible to query the data for information such as the total count of laptops
```
db.electronics.find( {"type":"laptop"} ).count()
```
```
389
```
or 6 inch smart phones
```
db.electronics.find( {"type":"smart phone", "screen size":6} ).count()
```
```
8
```
or the average screen size of all smart phones.
```
db.electronics.aggregate([{$match: {"type": "smart phone"}},{"$group":{"_id":"$type","average":{"$avg":"$screen size"}}}])
```
```
{ "_id" : "smart phone", "average" : 6 }
```

## 4 - Export data

We can use mongoexport to export our collection. We export the fields _id, “type”, and “model”, from the ‘electronics’ collection into a file named electronics.csv
```
mongoexport --host localhost --port 27017 --authenticationDatabase admin --username root --password *password* --db catalog --collection electronics --fields _id,type,model --out electronics.csv

```
```
2023-12-08T17:11:21.863-0500    connected to: mongodb://localhost:27017/
2023-12-08T17:11:21.875-0500    exported 438 records
```