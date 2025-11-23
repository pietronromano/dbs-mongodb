# 12:  Aggregation Pipeline 
- Course Section: https://www.udemy.com/course/mongodb-the-complete-developers-guide/learn/lecture/11859808#overview
- mongodb-slides.pdf: pg 96-104 

**Documentation:**
- Aggregation Pipeline: https://www.mongodb.com/docs/manual/core/aggregation-pipeline/
- Tutorials: https://www.mongodb.com/docs/manual/tutorial/aggregation-complete-examples/

# Import Sample Data (if not done already: see 10-indexes.md)
```sh
mongoimport --db contactData --collection contacts --file ./downloads/persons.json --jsonArray
```

```js
//Connect to the DB
mongosh --port 27017
show dbs
// Switch to contactData DB
use contactData

show collections
// count documents in contacts collection
db.contacts.countDocuments()

//Show first document
db.contacts.findOne()
```

---

# Pipeline Stages
The aggregation pipeline consists of multiple stages that process documents. Each stage transforms the documents as they pass through the pipeline. Common stages include:
- `$match`: Filters the documents to pass only the ones that match the specified condition(s) to the next pipeline stage.
- `$group`: Groups input documents by the specified _id expression and applies the accumulator expressions to each group.
- `$project`: Reshapes each document in the stream, such as by adding new fields or removing existing fields.
- `$sort`: Sorts all input documents and returns them to the next stage in the pipeline in sorted order.
- `$limit`: Limits the number of documents passed to the next stage in the pipeline.
- `$skip`: Skips over the specified number of documents and passes the remaining documents to the next stage in the pipeline.
- `$unwind`: Deconstructs an array field from the input documents to output a document for each element.


# Matching and Grouping
```js
//First stage: $match to filter female contacts
db.contacts.aggregate([
  { $match: { gender: "female"}}
])
//Results: returns a cursor with all female contacts

//Second stage: $group to group by state and count number of documents per state
// - _id: defines the group by field: "$" signifies field
// - totalPersons: creates a new field that counts the number of documents per group
db.contacts.aggregate([
    { $match: { gender: 'female' } },
    { $group: { _id: { state: "$location.state" }, totalPersons: { $sum: 1 } } },
    { $sort: { totalPersons: -1 } } //Sort by totalPersons descending
])
```

# Projecting Fields
```js
//Full Name with Capitalized First Letter
db.contacts.aggregate([
    {
      $project: {
        _id: 0,
        gender: 1,
        fullName: {
          $concat: [
            { $toUpper: { $substrCP: ['$name.first', 0, 1] } },
            {
              $substrCP: [
                '$name.first',
                1,
                { $subtract: [{ $strLenCP: '$name.first' }, 1] }
              ]
            },
            ' ',
            { $toUpper: { $substrCP: ['$name.last', 0, 1] } },
            {
              $substrCP: [
                '$name.last',
                1,
                { $subtract: [{ $strLenCP: '$name.last' }, 1] }
              ]
            }
          ]
        }
      }
    }
  ])

//Create GeoJSON Location Field, Convert birthdate Field to Date Type
db.contacts.aggregate([
    {
      $project: {
        _id: 0,
        name: 1,
        email: 1,
        birthdate: { $convert: { input: '$dob.date', to: 'date' } },
        age: "$dob.age",
        location: {
          type: 'Point',
          coordinates: [
            {
              $convert: {
                input: '$location.coordinates.longitude',
                to: 'double',
                onError: 0.0,
                onNull: 0.0
              }
            },
            {
              $convert: {
                input: '$location.coordinates.latitude',
                to: 'double',
                onError: 0.0,
                onNull: 0.0
              }
            }
          ]
        }
      }
    },
    {
      $project: {
        gender: 1,
        email: 1,
        location: 1,
        birthdate: 1,
        age: 1,
        fullName: {
          $concat: [
            { $toUpper: { $substrCP: ['$name.first', 0, 1] } },
            {
              $substrCP: [
                '$name.first',
                1,
                { $subtract: [{ $strLenCP: '$name.first' }, 1] }
              ]
            },
            ' ',
            { $toUpper: { $substrCP: ['$name.last', 0, 1] } },
            {
              $substrCP: [
                '$name.last',
                1,
                { $subtract: [{ $strLenCP: '$name.last' }, 1] }
              ]
            }
          ]
        }
      }
    }
  ]);

```

---

# Filtering Array Elements
```js
//Example of $filter within $project to filter array elements
db.contacts.aggregate([
    {
      $project: {
        _id: 0,
        name: 1,
        email: 1,
        hobbies: {
          $filter: {
            input: '$hobbies',
            as: 'hobby',
            cond: { $ne: ['$$hobby', 'reading'] } //Exclude 'reading' from hobbies
          }
        }
      }
    }
  ]);

```

---

# Using Bucket Stages
```js
//Example of $bucket to categorize ages into groups
db.contacts.aggregate([
    {
      $bucket: {
        groupBy: '$dob.age',
        boundaries: [18, 30, 40, 50, 60, 120],
        output: {
          numPersons: { $sum: 1 },
          averageAge: { $avg: '$dob.age' }
        }
      }
    }
  ]);

//Example of $bucketAuto to automatically create age buckets
db.contacts.aggregate([
    {
      $bucketAuto: {
        groupBy: '$dob.age',
        buckets: 5,
        output: {
          numPersons: { $sum: 1 },
          averageAge: { $avg: '$dob.age' }
        }
      }
    }
  ])
```

---

# Writing Aggregation Results to a new Collection
```js
//Example of $out to write aggregation results to a new collection
db.contacts.aggregate([
    {
      $project: {
        _id: 0,
        name: 1,
        email: 1,
        birthdate: { $toDate: '$dob.date' },
        age: "$dob.age",
        location: {
          type: 'Point',
          coordinates: [
            {
              $convert: {
                input: '$location.coordinates.longitude',
                to: 'double',
                onError: 0.0,
                onNull: 0.0
              }
            },
            {
              $convert: {
                input: '$location.coordinates.latitude',
                to: 'double',
                onError: 0.0,
                onNull: 0.0
              }
            }
          ]
        }
      }
    },
    {
      $project: {
        gender: 1,
        email: 1,
        location: 1,
        birthdate: 1,
        age: 1,
        fullName: {
          $concat: [
            { $toUpper: { $substrCP: ['$name.first', 0, 1] } },
            {
              $substrCP: [
                '$name.first',
                1,
                { $subtract: [{ $strLenCP: '$name.first' }, 1] }
              ]
            },
            ' ',
            { $toUpper: { $substrCP: ['$name.last', 0, 1] } },
            {
              $substrCP: [
                '$name.last',
                1,
                { $subtract: [{ $strLenCP: '$name.last' }, 1] }
              ]
            }
          ]
        }
      }
    },
    { $out: "transformedPersons" }
  ]);
//Check new collection
show collections
db.transformedPersons.findOne()
````

---
