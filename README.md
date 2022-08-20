Starting the server
---
With the defaults or explicitly.

    mongod
    mongod --bind_ip localhost --port 27017 --dbpath C:\data\db\

Starting the shell
---
With the defaults or explicitly.

    mongosh
    mongosh --host localhost --port 27017

The shell is a complete javascript interpreter.

    > let add = (a, b) => a + b;
    > add(1, 2);
    3

Databases
---
The prompt shows the current database.  
Switching to a different database.

    test> use books
    switched to db books
    books>

Data types
---
Supported data types.  
Numbers are float64 by default, be careful.  
Dates are in seconds since the unix epoch, so UTC.

    let document = {
      "Null": null,
      "Boolean": true,
      "Float64": 1,
      "Integer": Int32(1),
      "Long": Long("1"),
      "String": "A",
      "Date": ISODate(),
      "ObjectId": ObjectId(),
      "Array": [ 1, 2, 3 ],
      "EmbeddedDocument": {
      }
    }

Insertions
---
Adding documents to a collection.  
New documents are automatically assigned an _id.

    db.books.insertOne({ "title": "Book1" })

    {
      acknowledged: true,
      insertedId: ObjectId("62f29d8132b7ec34dae90a5c")
    }

Adding multiple documents in bulk.

    db.books.insertMany([ { "title": "Book2" }, { "title": "Book3" }, { "title": "Book4" } ])

    {
      acknowledged: true,
      insertedIds: {
        '0': ObjectId("62f29ed032b7ec34dae90a5e"),
        '1': ObjectId("62f29ed032b7ec34dae90a5f"),
        '2': ObjectId("62f29ed032b7ec34dae90a60")
      }
    }

Queries
---
Searching for specific values.  
All values must match.

    db.users.find({ "firstName": "Alice" })
    db.users.find({ "firstName": "Alice", "occupation": "Pirate" })

Using comparaison operators.

    db.users.find({ "salary": { "$gt": 10 } })
    db.users.find({ "salary": { "$gte": 10 } })
    db.users.find({ "salary": { "$lt": 50 } })
    db.users.find({ "salary": { "$lte": 50 } })
    db.users.find({ "salary": { "$eq": 70 } })
    db.users.find({ "salary": { "$ne": 70 } })
    db.users.find({ "salary": { "$in": [ 10, 50 ] }})

Using logical operators.

    db.users.find({ "$and": [{ "firstName": "Alice" }, { "salary": 10 }]})
    db.users.find({ "$or": [{ "firstName": "Alice" }, { "salary": 10 }]})
    db.users.find({ "$nor": [{ "firstName": "Alice" }, { "salary": 10 }]})

Negations.

    db.users.find({ "firstName": { "$not": { "$eq": "Alice" }}})

Querying for null.  
Matches null and undefined.

    db.users.find({ "occupation": null })
    db.users.find({ "occupation": { "$eq": null, "$exists": true }})

Regular expressions.

    db.users.find({ "firstName": { "$regex": /^Alice/ }})

Matching documents by code.  
Expressive but causes a collection scan.

    db.users.find({ "$where": function () {
        return this.firstName != this.occupation;
    }})

Queries on arrays
---
Searching for specific values in arrays.  
All values must be matched.

    db.users.find({ "numbers": 5 })
    db.users.find({ "numbers": { "$all": [ 5, 10 ]}})

Searching for exact arrays.  
Items must be in the same order.

    db.users.find({ "numbers": [ 5, 10 ]})

Conditions on arrays.  
All conditions are matched by one item.

    db.users.find({ "numbers": { "$gt": 10, "$lt": 20 }})

All conditions are matched by the same item.

    db.users.find({ "numbers": { "$elemMatch": { "$gt": 10, "$lt": 20 }}})

Querying by the size of an array.

    db.users.find({ "numbers": { "$size": 3 }})

Queries on embedded documents
---
Searching for specific values in embedded documents.  
All values must be matched.

    db.invoices.find({ "customer.name": "Alice", "customer.address": "123 Sunny Street" })

Searching for exact embedded documents.  
Keys must be in the same order.

    db.invoices.find({ "customer": { "name": "Alice", "address": "123 Sunny Street" }})

Projections
---
Returning only a subset of keys.  
Always includes _id.

    db.users.find({}, { "firstName": 1, "lastName": 1 })

Not returning a subset of keys.  
Includes the other ones.

    db.users.find({}, { "password": 0 })

Returning the subset of an array.

    db.users.find({}, { "languages": { "$slice": 2 }})
    db.users.find({}, { "languages": { "$slice": -3 }})
    db.users.find({}, { "languages": { "$slice": [1, 2] }})

Returning the matching item of an array.

    db.users.find({ "languages" : "French" }, { "languages.$": 1 })

Cursors
---
The find method returns a cursor.  
No query is executed before it gets iterated.

    let users = db.users.find({});

    while (users.hasNext()) {
        let user = users.next();
        print(user); 
    }

Sorting.

    db.users.find({}).sort({ "firstName": 1, "lastName": -1 })

Paginating.

    db.users.find({}).skip(1).limit(3)

Updates
---
The first parameter is a query.  
It identifies the updated documents.

Setting values.  

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$set": { "occupation": "Ninja", "invisible": true }})

Incrementing values.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$inc": { "salary": 10 }})

Removing values.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$unset": { "invisible": 1 }})

Updating multiple documents in bulk.

    db.users.updateMany(
        { "occupation": "Programmer" },
        { "$inc": { "salary": 10 }})

Updating documents by code.

    let user = db.users.findOne({ "firstName": "Alice" })
    user.firstName = user.firstName.toUpperCase()
    db.users.replaceOne({ "_id": user._id }, user)

Updates on arrays
---
Adding a single value.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$push": { "pets": "Bird" }})

Adding multiple value.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$push": { "pets": { "$each": [ "Cat", "Dog" ]}}})

Limiting the array length.  
Keeps the last three items pushed.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$push": { "pets": { "$each": [ "Elephant" ], "$slice": -3 }}})

Using arrays as sets.  
Does not add existing values.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$addToSet": { "pets": "Fish" }})

Updating an array item by index.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$set": { "pets.0": "Alligator" }})

Updating an array item by query match.

    db.users.updateOne(
        { "pets": "Dog" },
        { "$set": { "pets.$": "Wolf" }})

Updating multiple array items.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$set": { "pets.$[elem]": "Eagle" }},
        {
          "arrayFilters": [{ "elem": "Bird" }]
        })

Upserts
---
Does an update when a document is matched.

    db.users.updateOne(
        { "firstName": "Alice" },
        { "$set": { "occupation": "Cook" }},
        { "upsert": true })

Otherwise does an insert.  
Includes the values from the query.

    db.users.updateOne(
        { "firstName": "Bob" },
        { "$set": { "occupation": "Pilot" }},
        { "upsert": true })

Setting a value only on insertion.

    db.users.updateOne(
        { "firstName": "Carl" },
        { "$setOnInsert": { "createdOn": ISODate() }},
        { "upsert": true })

Find and update
---
Makes the operations atomic.

    db.processes.findOneAndUpdate(
        { "status": "READY" },
        { "$set": { "status": "RUNNING" }},
        { "sort": { "priority": 1 }, "returnNewDocument": true })

    {
      _id: ObjectId("62f7ed7be49ee735d296c1e6"),
      priority: 1,
      status: 'RUNNING'
    }

Deletions
---
Deleting a document.  

    db.users.deleteOne({ "firstName": "Alice" })

Deleting multiple documents in bulk.

    db.users.deleteMany({ "occupation": "Programmer" })

Dropping a collection.

    db.users.drop();

Indexes
---
Creating indexes.

    db.users.createIndex({ "username": 1 })
    db.users.createIndex({ "username": 1, "age": 1 })

Creating unique indexes.  
Partial when null is allowed.

    db.users.createIndex({ "email": 1 }, { "unique": true })
    
    db.users.createIndex({ "email": 1 }, {
      "unique": true,
      "partialFilterExpression": {
        "email": { "$exists": true }
      }})

Creating indexes on arrays.  
Each document gets one index entry per array item.  
Careful with index size.

    db.users.createIndex({ "addresses.city": 1 })

Time to live indexes.  
Expired documents automatically get deleted.

    db.logs.createIndex({ "dateTime": 1 }, { "expireAfterSeconds": 24 * 60 * 60 })

Listing indexes.

    db.users.getIndexes()

Explaining query plans.

    db.users.find({ "username": "user100" }).explain("executionStats")

Understanding execution stages.

- COLLSCAN: Consider an index
- IXSCAN: Good job
- FETCH: Consider a convering index
- PROJECTION_COVERED: Good job
- SORT: Consider adding the sort key to the index

Significant indicators.

- nReturned
- docsExamined
- executionTimeMillis

Geospatial queries
---
Type of values.  

    { "type": "Point", "coordinates": [1, 2] }
    { "type": "LineString", "coordinates": [[1, 2], [3, 4], [4, 5]] }
    { "type": "Polygon", "coordinates": [[1, 2], [3, 4], [4, 5]] }

Creating an index.

    db.neighborhoods.createIndex({ "location": "2dsphere" })

Querying for areas containing a point.

    db.neighborhoods.find({
      "geometry": {
        "$geoIntersects": {
          "$geometry": {
            "type": "Point",
            "coordinates": [-73.93414657, 40.82302903]
          }
        }
      }})

Querying for locations near a point.

    db.restaurants.find({
      "location": {
        "$nearSphere": {
          "$geometry": {
            "type": "Point",
            "coordinates": [-73.93414657, 40.82302903]
          },
          "$maxDistance": 8046.7
        }
      }})

Full text queries
---
Creating an index.

    db.recipes.createIndex({ "name": "text" })

    db.recipes.createIndex(
      { "name": "text", "description": "text" },
      { "weights": { "name": 3, "description": 2 }})

Querying for multiple keywords.

    db.recipes.find({ "$text": { "$search": "spiced" }})
    db.recipes.find({ "$text": { "$search": "spiced espresso" }});
    db.recipes.find({ "$text": { "$search": "espresso -milk" }});

Querying for exact phrases.

    db.recipes.find({ "$text": { "$search": "\"ice cream\"" }});

Querying relevance.

    db.recipes.find(
      { "$text": { "$search": "spiced espresso" }},
      { "score": { "$meta": "textScore" }})

Aggregation framework
---
Building a pipeline.

    db.users.aggregate([
      { "$match": { "age": { "$gte": 20, "$lte": 30 }}},
      { "$sort": { "age": 1 }},
      { "$skip": 10 },
      { "$limit": 5 },
      { "$project": { "_id": 0, "username": 1}}
    ])

Joining.

    db.users.aggregate([
      { "$lookup":
        {
           "from": "profiles",
           "localField": "_id",
           "foreignField": "userId",
           "as": "profile"
        }
      }
    ])

Grouping.

    db.users.aggregate([
      { "$group": {
        "_id": { "occupation": "$occupation" },
        "averageAge": { "$avg": "$age" }
      }}
    ])

Unwinding arrays.  
Duplicates the document for each array item.

    db.users.aggregate([
      { "$match": { "firstName": "Alice" }},
      { "$unwind": "$numbers" }
    ])

    [
      { _id: ObjectId("62faac6e2d6219e5d234a42c"), firstName: 'Alice', numbers: 1 },
      { _id: ObjectId("62faac6e2d6219e5d234a42c"), firstName: 'Alice', numbers: 2 },
      { _id: ObjectId("62faac6e2d6219e5d234a42c"), firstName: 'Alice', numbers: 3 }
    ]

Saving the results.

    db.users.aggregate([
      { "$merge": { "into": "results", "on": "_id" }}
    ])

Security
---
Creating a user.

    mongosh

    use admin
    db.createUser({
      user: "mathieu",
      pwd: passwordPrompt(),
      roles: [ "root" ]
    })

Shutdown mongod and mongosh.  
Connecting as a user.

    mongod --auth
    mongosh --authenticationDatabase "admin" -u "mathieu" -p

Replica sets
---
Provides failover servers synchronized by operation log replay.  
Starting the servers.

    mongod --bind_ip localhost --port 27017 --replSet replSet1 --dbpath C:\data\rs1\
    mongod --bind_ip localhost --port 27018 --replSet replSet1 --dbpath C:\data\rs2\
    mongod --bind_ip localhost --port 27019 --replSet replSet1 --dbpath C:\data\rs3\

Configuring the replica set.

    mongosh --host localhost --port 27017

    let config = {
      "_id": "replSet1",
      "members": [
        { "_id": 0, "host": "localhost:27017", "priority": 2 },
        { "_id": 1, "host": "localhost:27018" },
        { "_id": 2, "host": "localhost:27019" }
      ]
    }

    rs.initiate(config)
    rs.status()

Connecting to the replica set.

    mongosh --host replSet1/localhost:27017,localhost:27018,localhost:27019

    db.cars.insertOne({ "color": "red" })

Kill the primary server.  
A new primary gets elected.

    rs.status()

The replica set is still available.  
No reconnection is required from the application.

    db.cars.find()

Sharding
---
Spreads data accross multiple servers.  
Starting the configuration server.

    mongod --bind_ip localhost --port 27017 --replSet config --configsvr --dbpath C:\data\config\
    mongosh localhost:27017
    
    rs.initiate()

Starting the router.

    mongos --bind_ip localhost --port 27018 --configdb config/localhost:27017

Starting the shard servers.

    mongod --bind_ip localhost --port 27019 --replSet shard1 --shardsvr --dbpath C:\data\shard1\
    mongosh localhost:27019
    
    rs.initiate()

    mongod --bind_ip localhost --port 27020 --replSet shard2 --shardsvr --dbpath C:\data\shard2\
    mongosh localhost:27020

    rs.initiate()

Adding the shard servers.

    mongosh localhost:27018

    sh.addShard("shard1/localhost:27019")
    sh.addShard("shard2/localhost:27020")
    sh.status()

Sharding a collection.

    mongosh localhost:27018

    sh.shardCollection("test.cities", { "index": "hashed" })

    for (var i = 0; i < 100; i++) {
        db.cities.insertOne({ "index": i, "name": "city" + i })
    }

    db.cities.getShardDistribution()

Transactions
---
Requires a replica set.

    mongod --replSet replSet1 --dbpath C:\data\tx\
    mongosh

    rs.initiate()

Running a transaction.

    let session = db.getMongo().startSession()

    session.startTransaction({
      "readConcern": { "level": "snapshot" },
      "writeConcern": { "w": "majority" }
    })

    // Not visible outside the transaction.
    session.getDatabase("test").getCollection("widgets").insertOne({
      "color": "red"
    })

    // Outside changes are not visible after first read.
    session.getDatabase("test").getCollection("widgets").find()

    // Outside changes on read documents cause concurrency errors.
    session.getDatabase("test").getCollection("widgets").updateMany({}, {
      "$set": { "color": "blue" }
    })

    session.commitTransaction()

Backups
---
Making a backup.

    mongodump
    mongodump -d test
    mongodump -d test -c users

Restoring a backup.

    mongorestore dump/

Profiling
---
Querying the current operations.

    db.currentOp()

Using the profiler.

    db.setProfilingLevel(2)

    db.keyboards.insert({ "color": "black", "noisy": true })
    db.keyboards.findOne({ "noisy": true })
    db.keyboards.deleteOne({})

    db.system.profile.find()

    db.setProfilingLevel(0)

Per collection and global statistics.

    mongotop
    mongostat
