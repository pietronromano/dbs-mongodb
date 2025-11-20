# 06: Diving Into Create Operations
- Course Section: https://www.udemy.com/course/mongodb-the-complete-developers-guide/learn/lecture/11850580#overview
- mongodb-slides.pdf: pg 52-58


# Inserting Documents
You can use the `insertOne()` and `insertMany()` methods to add documents to a collection in MongoDB.
References:
- https://www.mongodb.com/docs/manual/reference/method/db.collection.insertMany/
- https://www.mongodb.com/docs/manual/reference/method/db.collection.insertOne/
- https://www.mongodb.com/docs/manual/tutorial/insert-documents/
- https://www.mongodb.com/docs/manual/reference/write-concern/

## Atomicity
All write operations in MongoDB are atomic on the level of a single document. This means that if you perform an insert, update, or delete operation on a single document, that operation will either complete fully or not at all. 
However, operations that modify multiple documents are not atomic by default.


## Order of Execution of Operations
By default, documents are inserted in the order they are provided.
- If `ordered` is set to `true` and an insert fails, the server **does not continue** inserting records. `True` is the default value for `ordered`.
- If `ordered` is set to `false` and an insert fails, the server **continues** inserting records. Documents may be reordered by mongod to increase performance. Applications should not depend on ordering of inserts if using an unordered insertMany().

## Write Concerns
With write concerns, you can specify the level of acknowledgment requested from MongoDB for write operations to a standalone mongod instance, a replica set, or a sharded cluster.

Write concern can include the following fields:
{ w: <value>, j: <boolean>, wtimeout: <number> }

- the `w` option to request acknowledgment that the write operation has propagated to a specified number of mongod instances or to mongod instances with specified tags.
- the `j` option to request acknowledgment that the write operation has been written to the on-disk journal (if journaling is enabled). Journaling provides durability in the event of a power failure or other unexpected shutdown, and is quicker than waiting for the data to be written to data files.
- the `wtimeout` option to specify a time limit [in milliseconds] to prevent write operations from blocking indefinitely.

## Syntax
```javascript
use publications
db.authors.insertMany(
   [ {"author": "Author 11",} , {"author": "Author 12",}, {"author": "Author 13",} ],
   {
      writeConcern: { w: 1, j: true, wtimeout: 1000 },
      ordered: false
   }
)

//Result:
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId('691cea6329e579438f28d661'),
    '1': ObjectId('691cea6329e579438f28d662'),
    '2': ObjectId('691cea6329e579438f28d663')
  }
}
```

---

# Importing Data
- mongoimport is a command-line utility that comes bundled with MongoDB. It is used to import content from a JSON, CSV, or TSV file into a MongoDB database.
- https://www.mongodb.com/docs/database-tools/mongoimport/
cd udemy-mongodb

# Example Command: will create the 'tv' database and 'tvShows' collection if they do not exist
```javascript
//From terminal:
mongoimport --db tv --collection shows --file downloads/tv-shows.json --jsonArray

//From mongo shell:
use tv
db.shows.find()
```