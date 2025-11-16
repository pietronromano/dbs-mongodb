# 01: Introduction and Installation
- Course Section: https://www.udemy.com/course/mongodb-the-complete-developers-guide/learn/lecture/13914002#overview

## Install MongoDB Community with Docker
- Pull the MongoDB Docker Image
`docker pull mongodb/mongodb-community-server:latest`

- Run the Image as a Container
`docker run --name mongodb -p 27017:27017 -d mongodb/mongodb-community-server:latest`


## Install VS Code extension
- https://www.mongodb.com/docs/mongodb-vscode/
- Open VS Code Extensions: MongoDB for VS Code v1.14.2 
- In the Extension: create a new connection, use this connection string:
mongodb://127.0.0.1:27017
  

## Install Mongo tools  on mac
- Mongosh: https://www.mongodb.com/docs/mongodb-shell/install/?operating-system=macos&macos-installation-method=homebrew
`brew install mongosh`

- Database Tools: https://www.mongodb.com/docs/database-tools/installation/?operating-system=macos&package-type=homebrew
- Check if there:
`brew list | grep mongodb-database-tools`
- Install:
`brew tap mongodb/brew`
brew install mongodb-database-tools

---

# Test the Deployment
- Connect with mongosh
mongosh --port 27017

- Validate Your Deployment: To confirm your MongoDB instance is running, run the Hello command (NOTE: mongosh allows multiline commands):

```javascript
db.runCommand(
    {
        hello: 1
    }
)
```

- The result of this command returns a document describing your mongod deployment:
```javascript
{
   isWritablePrimary: true,
   topologyVersion: {
      processId: ObjectId("63c00e27195285e827d48908"),
      counter: Long("0")
},
   maxBsonObjectSize: 16777216,
   maxMessageSizeBytes: 48000000,
   maxWriteBatchSize: 100000,
   localTime: ISODate("2023-01-12T16:51:10.132Z"),
   logicalSessionTimeoutMinutes: 30,
   connectionId: 18,
   minWireVersion: 0,
   maxWireVersion: 20,
   readOnly: false,
   ok: 1
}
```

- List the dbs:
```javascript
show dbs

Results: Shows 3 metadata dbs 
    admin   40.00 KiB
    config  60.00 KiB
    local   40.00 KiB
```

---

# Mongodb basic commands

- Create a new DB automatically:
```javascript
use Shop

Results:
switched to db Shop
Shop>
```

- Create a collection, "products", and insert a document:
```javascript
db.products.insertOne(
    {
        "name": "Product 1",
        "price": 12.99 
    }
)

Results:
    {
        acknowledged: true,
        insertedId: ObjectId('6919fded191c9d54d944819b')
    }
```

- Show contents of the collection:
- NOTE: the data returned is NOT in JSON format (no quotations around fields, single quotations around string values)
```javascript
db.products.find()

Results:
[
  {
    _id: ObjectId('6919fded191c9d54d944819b'),
    name: 'Product 1',
    price: 12.99
  }
]
```

- Insert a nested object

db.products.insertOne(
    {
        "name": "PC 1",
        "price": 1299.99, 
        "specification" : {
            "RAM": 32,
            "CPU": "Intell i7"
        }
    }
)

