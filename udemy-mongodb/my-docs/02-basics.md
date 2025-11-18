# 02: Basics and CRUD Operations
- Course Section: https://www.udemy.com/course/mongodb-the-complete-developers-guide/learn/lecture/11739168#overview
- Shell CRUD: https://www.mongodb.com/docs/mongodb-shell/crud/

## Notes:
- Databases and Collections are created laizly - only when data is first stored, but they can be created explicitly
- Insert with data from 01-flights.json:
  - NOTE: _id can be supplied or will be generated if not present
  - _id values are ordered according to when they're inserted
  ```javascript
   insertedIds: {
    '0': ObjectId('691a3765191c9d54d944819e'),
    '1': ObjectId('691a3765191c9d54d944819f')
  }
  ```

# Flight Data collection
```javascript
use flights
db.flightData.insertMany ([
  {
    "departureAirport": "MUC",
    "arrivalAirport": "SFO",
    "aircraft": "Airbus A380",
    "distance": 12000,
    "intercontinental": true
  },
  {
    "departureAirport": "LHR",
    "arrivalAirport": "TXL",
    "aircraft": "Airbus A320",
    "distance": 950,
    "intercontinental": false
  }
])
```

## Finding documents
- Find All: NOTE: find() returns a CURSOR, not just an array
`db.flightData.find()`

- FindOne ({filter})
`db.flightData.findOne({"departureAirport":"LHR"})`


## Update data ({filter}, {set})
`db.flightData.updateOne({departureAirport: "LHR"}, {$set: {"Aircraft": "Boeing 777"}})`

- Add a field, filer with _id:
`db.flightData.updateOne({_id: ObjectId('691a3765191c9d54d944819f')},{$set: {"delayed" : true}})`


## Delete data
- Delete One:
`db.flightData.deleteOne({departureAirport: "MUC"})`

- See 02-passengers.json: insert many:
```javascript
db.passengers.insertMany(
    [
  {
    "name": "Max Schwarzmueller",
    "age": 29
  },
 
]
)
```

- Find All: NOTE: find() returns a CURSOR, not just an array
```javascript
db.passengers.find()

//Result:
...
    name: 'Albert Twostone',
    age: 68
  }
]
Type "it" for more
```

- Force an array 
```javascript
db.passengers.find().toArray()
```
---

## Projection
```javascript
// Just return name, although _id gets included by default:
db.passengers.find({},{name: 1})

// exclude id: exclude with 0:
db.passengers.find({},{name: 1, _id: 0})
```
---

## Arrays
- New  Hobbies collection: see array-data.json
```javascript
db.hobbies.insertMany(
    [
  {
    "name": "Max",
    "hobbies": ["Sports", "Cooking"],
    "age": 29,
    "examScores": [
      { "difficulty": 4, "score": 57.9 },
      { "difficulty": 6, "score": 62.1 },
      { "difficulty": 3, "score": 88.5 }
    ]
    ...
)
```

- Find:
```javascript
db.hobbies.find({"name": "Max"})
db.hobbies.find({"hobbies": "Cooking"})
```

- Query an array: https://www.mongodb.com/docs/manual/tutorial/query-array-of-documents/
- NOTE: we concatenate properties with "." in a string:
`db.hobbies.find({"examScores.difficulty": {$eq: 3}})`


