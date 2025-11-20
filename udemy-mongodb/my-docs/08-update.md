# 08:  Update Operations 
- Course Section: https://www.udemy.com/course/mongodb-the-complete-developers-guide/learn/lecture/11850842#overview
- mongodb-slides.pdf: pg 70-76

Documentation:
- https://www.mongodb.com/docs/manual/tutorial/update-documents/


# updateOne() & updateMany()
 - You can use updateOne() and updateMany() to update one or more documents in a collection
 - You specify a filter (query selector) with the same operators you know from find()
 - The second argument then describes the update (e.g. via $set or otherupdate operators)

---

# Sample Data
```js
//Create usersDB and sample data
use usersDB
db.users.insertMany(
[
  {
    "name": "Max",
    "hobbies": [
      {
        "title": "Sports",
        "frequency": 3
      },
      {
        "title": "Cooking",
        "frequency": 6
      }
    ],
    "phone": 131782734
  },
  {
    "name": "Manuel",
    "hobbies": [
      {
        "title": "Cooking",
        "frequency": 5
      },
      {
        "title": "Cars",
        "frequency": 2
      }
    ],
    "phone": "012177972",
    "age": 30
  },
  {
    "name": "Anna",
    "hobbies": [
      {
        "title": "Sports",
        "frequency": 2
      },
      {
        "title": "Yoga",
        "frequency": 3
      }
    ],
    "phone": "80811987291",
    "age": null
  },
  {
    "name": "Chris",
    "hobbies": ["Sports", "Cooking", "Hiking"]
  }
]
);

//Results
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId('691f61b5894ac4042c908d33'),
    '1': ObjectId('691f61b5894ac4042c908d34'),
    '2': ObjectId('691f61b5894ac4042c908d35'),
    '3': ObjectId('691f61b5894ac4042c908d36')
  }
}

db.users.find();
```

# Update Operators
 - You can update fields with a broad variety of field update operators like $set, $inc, $min etc
 - If you need to work on arrays, take advantage of the shortcuts ($, $[] and $[<identifier>] + arrayFilters)
 - Also use array update operators like `$push` or `$pop` to efficiently add or remove elements to or from arrays
 - **Upsert Option**: If updateOne(), updateMany(), or replaceOne() includes upsert : true and no documents match the specified filter, then the operation creates a new document and inserts it. If there are matching documents, then the operation modifies or replaces the matching document or documents.


```js
//Examples for update operators
db.users.updateOne(
    {_id: ObjectId('691f61b5894ac4042c908d33')},
    {$set: { 
    hobbies: [
      { title: 'Sports', frequency: 13 },
      { title: 'Cooking', frequency: 16 }
    ] } }
);

db.users.findOne(
    {"_id": ObjectId('691f61b5894ac4042c908d33')});

//Add field isSporty, Update many
db.users.updateMany(
    {"hobbies.title": "Sports"},
    { $set: { isSporty: true } }
);
db.users.find();

//Remove field isSporty, Update many
db.users.updateMany(
    { isSporty: true },
    { $unset: { isSporty: "" } }
);
db.users.find();

//Increase frequency of Sports hobby by 5 for all users having it
db.users.updateMany(
    { "hobbies.title": "Sports" },
    { $inc: { "hobbies.$.frequency": 5 } }
);
db.users.find();

//Set lastModified field to current date for a specific user
db.users.updateOne(
   { "_id": ObjectId('691f61b5894ac4042c908d36') },
   {
     $currentDate: { lastModified: true }
   }
)
db.users.findOne(
    {"_id": ObjectId('691f61b5894ac4042c908d36')});
//Result
lastModified: ISODate('2025-11-20T19:08:02.447Z')

```