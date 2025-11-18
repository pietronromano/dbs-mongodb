# 03: Schemas and Relations
- Course Section: https://www.udemy.com/course/mongodb-the-complete-developers-guide/learn/lecture/11758254#overview
- mongodb-slides.pdf: pg 29-49

- MongoDB Limits: https://docs.mongodb.com/manual/reference/limits/
- MongoDB Data Types: https://docs.mongodb.com/manual/reference/bson-types/
- Schema Validation: https://docs.mongodb.com/manual/core/schema-validation/
- Relations Lookup: https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/
- Schema Validation: https://www.mongodb.com/docs/drivers/rust/v3.2/fundamentals/schema-validation/

---

# Data Modelling & Structuring – Things to Consider
- In which Format will you fetch your Data?
- How often will you fetch and change your Data?
- How much data will you save (and how big is it)?
- How is your Data related?
- Will Duplicates hurt you (=> many Updates)?
- Will you hit Data/ Storage Limits?

## Modelling Schemas
- Schemas should be modelled based onyour application needs
- Important factors are: Read and writefrequency, relations, amount (and size) of data
---

## Modelling Relations
- Two options: Embedded documents or references
- Use embedded documents if you got one-to-one or one-to-many relationships and no app or data size reason to split
- Use references if data amount/ size or application needs require it or for many-to-many relations
- Exceptions are always possible => Keep your app requirements in mind!
  
## Schema Validation 
- You can define rules to validate inserts and update before writing to the database
- Choose your validation level and action based on your application requirements

---

# BSON Data Types

| Type | Number | Alias | Example Value |
|------|--------|-------|---------------|
| Double | 1 | "double" | `12.34` |
| String | 2 | "string" | `"Hello MongoDB"` |
| Object | 3 | "object" | `{ name: "John", age: 30 }` |
| Array | 4 | "array" | `[1, 2, 3, "text"]` |
| Binary data | 5 | "binData" | Binary data/files |
| Undefined | 6 | "undefined" | `undefined` (deprecated) |
| ObjectId | 7 | "objectId" | `ObjectId("507f1f77bcf86cd799439011")` |
| Boolean | 8 | "bool" | `true` or `false` |
| Date | 9 | "date" | `new Date()` or `ISODate("2025-11-17")` |
| Null | 10 | "null" | `null` |
| Regular Expression | 11 | "regex" | `/pattern/i` |
| JavaScript | 13 | "javascript" | `function() { return true; }` |
| Symbol | 14 | "symbol" | (deprecated) |
| JavaScript with scope | 15 | "javascriptWithScope" | Function with scope |
| 32-bit integer | 16 | "int" | `123` |
| Timestamp | 17 | "timestamp" | `Timestamp(1634567890, 1)` |
| 64-bit integer | 18 | "long" | `NumberLong("9223372036854775807")` |
| Decimal128 | 19 | "decimal" | `NumberDecimal("123.45")` |
| Min key | -1 | "minKey" | `MinKey` |
| Max key | 127 | "maxKey" | `MaxKey` |

---

# Examples
- Connect with mongosh
`mongosh --port 27017`

- DBs
`show dbs`

- Find a document:
db.flightData.findOne({"_id": ObjectId("691a3765191c9d54d944819f")})

- Find a value:
db.flightData.findOne({"_id": ObjectId("691a3765191c9d54d944819f")}).distance

## Using $type to check data types

- Find documents where `distance` is a number (double):
```javascript
db.flightData.find({ distance: { $type: "double" } })
// or using type number
db.flightData.find({ distance: { $type: 1 } })
```

- Find documents where `distance` is an integer:
```javascript
db.flightData.find({ distance: { $type: "int" } })
```

- Find documents where `delayed` is a boolean:
```javascript
db.flightData.find({ delayed: { $type: "bool" } })
```

- Find documents where a field is a string:
```javascript
db.flightData.find({ aircraft: { $type: "string" } })
```

- Check for multiple types:
```javascript
db.flightData.find({ distance: { $type: ["int", "long", "double"] } })
```

- Using typeof in JavaScript (for returned values):
```javascript
var doc = db.flightData.findOne({"_id": ObjectId("691a3765191c9d54d944819f")})
typeof doc.distance  // returns "number"
typeof doc.aircraft  // returns "string"
```

---

# Relations

- Create a publications db:
```javascript
use publications
```

- Insert several Authors and Books:
```javascript
db.authors.insertMany(
    [
    {"author":"Author 1"},
    {"author":"Author 2"},
    {"author":"Author 3"}
    ]
)
    >insertedIds: {
        '0': ObjectId('691b944eef4dd0b2c955d54c'),
        '1': ObjectId('691b944eef4dd0b2c955d54d'),
        '2': ObjectId('691b944eef4dd0b2c955d54e')
    }

db.books.insertMany(
    [
    {"book":"Book 1"},
    {"book":"Book 2"},
    {"book":"Book 3"}
    ]
)

//Results
insertedIds: {
    '0': ObjectId('691b94e1ef4dd0b2c955d54f'),
    '1': ObjectId('691b94e1ef4dd0b2c955d550'),
    '2': ObjectId('691b94e1ef4dd0b2c955d551')
}
```

- Relate the books to the Authors
var author = db.authors.findOne({"_id":  ObjectId('691b944eef4dd0b2c955d54c')})
```javascript
//Relate author 1 to books 1,2
db.authors.updateOne({_id: ObjectId('691b944eef4dd0b2c955d54c')},
        {$set: {"books" : 
        [ObjectId('691b94e1ef4dd0b2c955d54f'),ObjectId('691b94e1ef4dd0b2c955d550') ]
        }})

//Relate author 3 to book 3
db.authors.updateOne({_id: ObjectId('691b944eef4dd0b2c955d54e')},
        {$set: {"books" : 
        [ObjectId('691b94e1ef4dd0b2c955d551') ]
        }})

//Show an author with book ids
db.authors.findOne({_id: ObjectId('691b944eef4dd0b2c955d54c')})

// Lookup authors with related books
db.authors.aggregate([
    {$lookup:{from: "books", localField: "books",foreignField: "_id", as: "books_details"}}
])
```

---

# Schema Validation
- Create a collection explicitly, adding a schema (see validation.js)
- validationAction: 'error' | warn'
```javascript
db.createCollection("posts", {  
    validationAction: 'error',
    validator: {
        $jsonSchema: {
        bsonType: 'object',
        required: ['title', 'text', 'creator', 'comments'],
        properties: {
            title: {
            bsonType: 'string',
            description: 'must be a string and is required'
            },
            text: {
            bsonType: 'string',
            description: 'must be a string and is required'
            },
            creator: {
            bsonType: 'objectId',
            description: 'must be an objectid and is required'
            },
            comments: {
            bsonType: 'array',
            description: 'must be an array and is required',
            items: {
                bsonType: 'object',
                required: ['text', 'author'],
                properties: {
                text: {
                    bsonType: 'string',
                    description: 'must be a string and is required'
                },
                author: {
                    bsonType: 'objectId',
                    description: 'must be an objectid and is required'
                }
                }
            }
            }
        }
        }
    }
})
```

- Try adding a document that doesn't conform to the schema
```javascript
db.posts.insertOne(
    {
        "title": "title1"
    }
) 

//Results:
details: {
    operatorName: '$jsonSchema',
    schemaRulesNotSatisfied: [
      {
        operatorName: 'required',
        specifiedAs: { required: [ 'title', 'text', 'creator', 'comments' ] },
        missingProperties: [ 'comments', 'creator', 'text' ]
      }
    ]
  }
```

- Delete a Collection
db.posts.drop()