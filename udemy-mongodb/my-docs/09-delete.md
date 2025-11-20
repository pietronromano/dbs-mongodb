# 09:  Update Operations 
- Course Section: https://www.mongodb.com/docs/manual/tutorial/remove-documents/
- mongodb-slides.pdf: pg 76-

Documentation:
- https://www.mongodb.com/docs/manual/tutorial/update-documents/

# deleteOne() & deleteMany()
 - You can use deleteOne() and deleteMany() to delete one or more documents in a collection
 - You specify a filter (query selector) with the same operators you know from find()

```js
//Delete One Example
db.users.deleteOne({"_id": ObjectId('691f61b5894ac4042c908d36')});

//Delete Many Example with filter
db.users.deleteMany({"phone": "inactive"});

//Delete Many Example with no filter (delete all documents in collection)
db.users.deleteMany({});

```

---




