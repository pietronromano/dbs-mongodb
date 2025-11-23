# 10:  Indexes
- Course Section: https://www.udemy.com/course/mongodb-the-complete-developers-guide/learn/lecture/11850910#overview
- mongodb-slides.pdf: pg 79-91: (NOTE: VERY USEFUL SLIDES!)

Documentation:
- Indexes: https://www.mongodb.com/docs/manual/indexes/
- Partial Indexes: https://docs.mongodb.com/manual/core/index-partial/
- Supported default_languages: https://docs.mongodb.com/manual/reference/text-search-languages/#text-search-languages
- How to use different languages in the same index: https://docs.mongodb.com/manual/tutorial/specify-language-for-text-index/#create-a-text-index-for-a-collection-in-multiple-languages
- TTL Indexes: https://www.mongodb.com/docs/manual/core/index-ttl/

# Key Concepts
- Indexes improve query performance by creating data structures that allow for faster data retrieval.
- Without indexes, MongoDB must perform a collection scan (COLLSCAN) to find documents that match a query, which can be slow for large datasets.
- Indexes can be created on one or more fields in a collection, and can be ascending (1) or descending (-1).
- MongoDB supports various types of indexes, including single-field indexes, compound indexes, multikey indexes (for arrays), text indexes (for full-text search), and geospatial indexes.
- Indexes can be created using the `createIndex()` method, and their usage can be analyzed using the `explain()` method to understand query performance.
- Indexes consume additional disk space and can impact write performance, so it's important to carefully consider which indexes to create based on query patterns.
- Limitations include:
  - Indexes are not beneficial if the query returns a large portion of the collection, as the overhead of using the index may outweigh the benefits.
  - Indexes cannot be created on fields that contain arrays of arrays.
  - There is a limit to the number of indexes per collection (64).
  - Indexes must fit within the available RAM for optimal performance.  
  - Indexes on large collections can take time to build and may require additional resources during the build process.
- Regularly monitor and analyze index usage to ensure optimal performance and adjust indexes as needed based on changing query patterns.
- Consider using partial indexes or **TTL (Time-To-Live)** indexes for specific use cases to optimize performance and storage. 
- Understand the trade-offs between read and write performance when designing indexes for your MongoDB collections.
- Use **covered queries** (queries that can be satisfied entirely by the index) to further improve performance.
- Keep in mind that indexes need to be maintained during write operations, which can introduce overhead.


# Import Data and Test Queries before Indexes

```js
//Import sample data: 5000 contacts
mongoimport --db contactData --collection contacts --file ./downloads/persons.json --jsonArray

//Connect to the DB
mongosh --port 27017
show dbs
use contactData
db.contacts.countDocuments()

db.contacts.findOne()

//Find all contact older than 60
db.contacts.countDocuments({ "dob.age": { $gt: 60 } })

db.contacts.find({ "dob.age": { $gt: 60 } }).limit(5)

//Use explain() to see query plan
db.contacts.explain().find({ "dob.age": { $gt: 60 } })
//Results in COLLSCAN (collection scan) since no index exists on dob.age 
winningPlan: {
      isCached: false,
      stage: 'COLLSCAN'

//More detailed execution stats
db.contacts.explain("executionStats").find({ "dob.age": { $gt: 60 } })
//Results in COLLSCAN (collection scan) since no index exists on dob.age
...
executionStats:
    executionTimeMillis: 5,
    totalDocsExamined: 5000,
    totalKeysExamined: 0

//Show allPlansExecution to see all stages
db.contacts.explain("allPlansExecution").find({ "dob.age": { $gt: 60 } })

```
---

# Create Indexes

```js
//Create index on dob.age, ascending order (1)
db.contacts.createIndex({ "dob.age": 1 })
//Result: 
dob.age_1

//Use explain() again to see query plan, will now use the index
db.contacts.explain("executionStats").find({ "dob.age": { $gt: 60 } })

//Results in IXSCAN (index scan), down from 5000 to 1222 documents examined, 5ms to 3ms
...
winningPlan: {
      isCached: false,
      stage: 'FETCH',
      inputStage: {
        stage: 'IXSCAN',
        keyPattern: { 'dob.age': 1 },
        indexName: 'dob.age_1',
...
executionStats: {
    executionSuccess: true,
    nReturned: 1222,
    executionTimeMillis: 3,
    totalKeysExamined: 1222,
    totalDocsExamined: 1222,
```

---

# Compound Indexes

```js
//Create compound index on age (asc) and gender (asc)
db.contacts.createIndex({ "dob.age": 1, "gender": 1 })
//Result:
dob.age_1_gender_1

//Test query that uses both fields
db.contacts.explain("executionStats").find({ "dob.age": { $gt: 60 }, gender: "male" })

//Show all the indexes on the collection
db.contacts.getIndexes()

//Create a unique index on email
db.contacts.createIndex({ email: 1 }, { unique: true })
//Finds and returns duplicate key error if duplicate emails exist

//Create a partial index on gender field 
db.contacts.createIndex(
  { gender: 1 },
  { partialFilterExpression: { gender: "male" } }
)
```

---

# Text Indexes

```js
use publications
db.books.countDocuments()

db.books.updateOne({ "_id": ObjectId('691b94e1ef4dd0b2c955d54f') },
  {
    $set: {
      description: "A fantasy novel about a young wizard named Harry Potter who discovers his magical heritage and attends Hogwarts School of Witchcraft and Wizardry."
    }
  }
)

db.books.updateOne(
  { _id: ObjectId('691b94e1ef4dd0b2c955d550')},
  {
    $set: {
      description: "A science fiction novel that explores themes of identity, consciousness, and the nature of reality through the story of a man who wakes up in a simulated world."
    }
  }
);

db.books.updateOne(
 { _id: ObjectId('691b94e1ef4dd0b2c955d551') },
 {
   $set: {
     description: "A dystopian novel set in a totalitarian society where the government exercises extreme control over its citizens, exploring themes of surveillance, propaganda, and individuality."
   }
 }
);

db.books.find()

//Create text index on description field
db.books.createIndex({ description: "text" })
//Result:
description_text

//Search using text index: (don't need to use the field name)
db.books.find({ $text: { $search: "novel" } })


//Search for multiple words, will return documents containing either word, showing relevance score 
db.books.find(
  { $text: { $search: "novel totalitarian" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } }) 

//Search for documents containing both "novel" AND "totalitarian"
db.books.find({ $text: { $search: "\"novel\" \"totalitarian\"" } })

```

---

# Building Indexes in the Background

```js
//Shell: Execute credit-ratings.js script to insert 1 million documents
mongosh downloads/credit-rating.js

//mongdb shell
use credit
show collections
db.ratings.countDocuments()

//Try searching before creating index
db.ratings.explain("executionStats").find({ age: { $gt: 60 } })
//Results in COLLSCAN, 1 million documents examined, 221ms

//Create index in background (default in MongoDB 4.2+)
db.ratings.createIndex({ age: 1 }, { background: true })
//Result: age_1, takes about a second to build

//Try searching after creating index (leave it a few minutes to ensure index is built)
db.ratings.getIndexes()

db.ratings.explain("executionStats").find({ age: { $gt: 60 } })
//Results in IXSCAN, 385609 documents examined, 77ms
```
