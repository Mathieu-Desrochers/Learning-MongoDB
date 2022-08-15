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

Javascript
---
The shell is a complete javascript interpreter.

    > let add = (a, b) => a + b;
    > add(1, 2);
    3

Running Scripts
---
Scripts can be run when starting the shell.

    mongosh migration001.js

Using an External Editor
---
Setting the editor.

    > config.set("editor", "notepad")

Editing a variable.

    > let document = {}
    > edit document
    > document = { "A": 1 }

Editing a statement.

    > edit
    > db.insertOne()

Databases
---
The prompt shows the current database.  
Switching to a different database.

    test> use books
    switched to db books
    books>

Data Types
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

    db.books.insertOne({ "Title": "Book1" })

    {
      acknowledged: true,
      insertedId: ObjectId("62f29d8132b7ec34dae90a5c")
    }

Adding multiple documents in bulk.

    db.books.insertMany([ { "Title": "Book2" }, { "Title": "Book3" }, { "Title": "Book4" } ])

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

    db.users.find({ "FirstName": "Alice" })
    db.users.find({ "FirstName": "Alice", "Occupation": "Pirate" })

Using comparaison operators.

    db.users.find({ "Salary": { "$gt": 10 } })
    db.users.find({ "Salary": { "$gte": 10 } })
    db.users.find({ "Salary": { "$lt": 50 } })
    db.users.find({ "Salary": { "$lte": 50 } })
    db.users.find({ "Salary": { "$eq": 70 } })
    db.users.find({ "Salary": { "$ne": 70 } })
    db.users.find({ "Salary": { "$in": [ 10, 50 ] }})

Using logical operators.

    db.users.find({ "$and": [{ "FirstName": "Alice" }, { "Salary": 10 }]})
    db.users.find({ "$or": [{ "FirstName": "Alice" }, { "Salary": 10 }]})
    db.users.find({ "$nor": [{ "FirstName": "Alice" }, { "Salary": 10 }]})

Negations.

    db.users.find({ "FirstName": { "$not": { "$eq": "Alice" }}})

Querying for null.  
Matches null and undefined.

    db.users.find({ "Occupation": null })
    db.users.find({ "Occupation": { "$eq": null, "$exists": true }})

Regular expressions.

    db.users.find({ "FirstName": { "$regex": /^Alice/ }})

Matching documents by code.  
Expressive but causes a collection scan.

    db.users.find({ "$where": function () {
        return this.Name != this.Occupation;
    }})

Queries on Arrays
---
Searching for specific values in arrays.  
All values must be matched.

    db.users.find({ "Numbers": 5 })
    db.users.find({ "Numbers": { "$all": [ 5, 10 ]}})

Searching for exact arrays.  
Items must be in the same order.

    db.users.find({ "Numbers": [ 5, 10 ]})

Conditions on arrays.  
All conditions are matched by one item.

    db.users.find({ "Numbers": { "$gt": 10, "$lt": 20 }})

All conditions are matched by the same item.

    db.users.find({ "Numbers": { "$elemMatch": { "$gt": 10, "$lt": 20 }}})

Querying by the size of an array.

    db.users.find({ "Numbers": { "$size": 3 }})

Queries on Embedded Documents
---
Searching for specific values in embedded documents.  
All values must be matched.

    db.invoices.find({ "Customer.Name": "Alice", "Customer.Address": "123 Sunny Street" })

Searching for exact embedded documents.  
Keys must be in the same order.

    db.invoices.find({ "Customer": { "Name": "Alice", "Address": "123 Sunny Street" }})

Projections
---
Returning only a subset of keys.  
Always includes _id.

    db.users.find({}, { "FirstName": 1, "LastName": 1 })

Not returning a subset of keys.  
Includes the other ones.

    db.users.find({}, { "Password": 0 })

Returning the subset of an array.

    db.users.find({}, { "Languages": { "$slice": 2 }})
    db.users.find({}, { "Languages": { "$slice": -3 }})
    db.users.find({}, { "Languages": { "$slice": [1, 2] }})

Returning the matching item of an array.

    db.users.find({ "Languages" : "French" }, { "Languages.$": 1 })

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

    db.users.find({}).sort({ "FirstName": 1, "LastName": -1 })

Paginating.

    db.users.find({}).skip(1).limit(3)

Updates
---
The first parameter is a query.  
It identifies the updated documents.

Setting values.  

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$set": { "Occupation": "Ninja", "Invisible": true }})

Incrementing values.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$inc": { "Salary": 10 }})

Removing values.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$unset": { "Invisible": 1 }})

Updating multiple documents in bulk.

    db.users.updateMany(
        { "Occupation": "Programmer" },
        { "$inc": { "Salary": 10 }})

Updating documents by code.

    let user = db.users.findOne({ "FirstName": "Alice" })
    user.FirstName = user.FirstName.toUpperCase()
    db.users.replaceOne({ "_id": user._id }, user)

Updates on Arrays
---
Adding a single value.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$push": { "Pets": "Bird" }})

Adding multiple value.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$push": { "Pets": { "$each": [ "Cat", "Dog" ]}}})

Limiting the array length.  
Keeps the last three items pushed.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$push": { "Pets": { "$each": [ "Elephant" ], "$slice": -3 }}})

Using arrays as sets.  
Does not add existing values.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$addToSet": { "Pets": "Fish" }})

Updating an array item by index.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$set": { "Pets.0": "Alligator" }})

Updating an array item by query match.

    db.users.updateOne(
        { "Pets": "Dog" },
        { "$set": { "Pets.$": "Wolf" }})

Updating multiple array items.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$set": { "Pets.$[elem]": "Eagle" }},
        {
          "arrayFilters": [{ "elem": "Bird" }]
        })

Upserts
---
Does an update when a document is matched.

    db.users.updateOne(
        { "FirstName": "Alice" },
        { "$set": { "Occupation": "Cook" }},
        { "upsert": true })

Otherwise does an insert.  
Includes the values from the query.

    db.users.updateOne(
        { "FirstName": "Bob" },
        { "$set": { "Occupation": "Pilot" }},
        { "upsert": true })

Setting a value only on insertion.

    db.users.updateOne(
        { "FirstName": "Carl" },
        { "$setOnInsert": { "CreatedOn": ISODate() }},
        { "upsert": true })

Find and Update
---
Makes the operations atomic.

    db.processes.findOneAndUpdate(
        { "Status": "READY" },
        { "$set": { "Status": "RUNNING" }},
        { "sort": { "Priority": 1 }, "returnNewDocument": true })

    {
      _id: ObjectId("62f7ed7be49ee735d296c1e6"),
      Priority: 1,
      Status: 'RUNNING'
    }

Deletions
---
Deleting a document.  

    db.users.deleteOne({ "FirstName": "Alice" })

Deleting multiple documents in bulk.

    db.users.deleteMany({ "Occupation": "Programmer" })

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

Geospatial Queries
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

Full Text Queries
---
Creating an index.

    db.recipes.createIndex({ "name": "text" })

    db.recipes.createIndex(
      { "name": "text", "description": "text" },
      { "weights": { "name": 3, "description": 2 }})

Querying for multiple keywords.

    db.recipes.find({ $text: { $search: "spiced" } })
    db.recipes.find({ $text: { $search: "spiced espresso" }});
    db.recipes.find({ $text: { $search: "espresso -milk" } });

Querying for exact phrases.

    db.recipes.find({ $text: { $search: "\"ice cream\"" }});

Querying relevance.

    db.recipes.find(
      { $text: { $search: "spiced espresso" }},
      { score: { $meta: "textScore" }})

Aggregation Framework
---
