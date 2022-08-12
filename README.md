Starting the server
---
With the defaults or explicitly.

    mongod
    mongod --bind_ip localhost --port 27017 --dbpath C":\data\db\

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
Scripts can be specified when starting the shell.

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

    > db.books.insertOne({ "Title": "Book1" })
    {
      acknowledged": true,
      insertedId": ObjectId("62f29d8132b7ec34dae90a5c")
    }

New documents are automatically assigned an _id.

    > db.books.findOne()
    { _id": ObjectId("62f29d8132b7ec34dae90a5c"), "Title": "Book1" }

Adding multiple documents in bulk.

    > db.books.insertMany([ { "Title": 'Book2' }, { "Title": "Book3" }, { "Title": "Book4" } ])
    {
      "acknowledged": true,
      "insertedIds": {
        '0': ObjectId("62f29ed032b7ec34dae90a5e"),
        '1': ObjectId("62f29ed032b7ec34dae90a5f"),
        '2': ObjectId("62f29ed032b7ec34dae90a60")
      }
    }

Queries
---
Searching for specific values.  
All values must match.

    db.users.find({ "FirstName": "Bob" })
    db.users.find({ "FirstName": "Bob", "Occupation": "Pirate" })

Using comparaison operators.

    db.users.find({ "Salary": { "$gt": 10 } })
    db.users.find({ "Salary": { "$gte": 10 } })
    db.users.find({ "Salary": { "$lt": 50 } })
    db.users.find({ "Salary": { "$lte": 50 } })
    db.users.find({ "Salary": { "$eq": 70 } })
    db.users.find({ "Salary": { "$ne": 70 } })
    db.users.find({ "Salary": { "$in": [ 10, 50 ] }})

Using logical operators.

    db.users.find({ "$and": [{ "FirstName": "Bob" }, { "Salary": 10 }]})
    db.users.find({ "$or": [{ "FirstName": "Bob" }, { "Salary": 10 }]})
    db.users.find({ "$nor": [{ "FirstName": "Bob" }, { "Salary": 10 }]})

Negations.

    db.users.find({ "FirstName": { "$not": { "$eq": "Bob" }}})

Querying for null.  
Matches null and undefined.

    db.users.find({ "Occupation": null })
    db.users.find({ "Occupation": { "$eq": null, "$exists": true }})

Regular expressions.

    db.users.find({ "FirstName": { "$regex": /^Bob/ }})

Providing a condition as a javascript function.  
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

Querying the size of arrays.  
Cannot be combined with other operators.

    db.users.find({ "Numbers": { "$size": 3 }})

Queries on Embedded Documents
---
Searching for specific values in embedded documents.  
All values must be matched.

    db.invoices.find({ "Customer.Name": "Bob", "Customer.Address": "123 Sunny Street" })

Searching for exact embedded documents.  
Keys must be in the same order.

    db.invoices.find({ "Customer": { "Name": "Bob", "Address": "123 Sunny Street" }})

Returned keys
---
Return only a subset of keys.  
Always includes _id.

    db.users.find({}, { "FirstName": 1, "LastName": 1 })

Not returning a subset of keys.  
Includes the other ones.

    db.users.find({}, { "Password": 0 })

Returning an array subset.

    db.users.find({}, { "Languages": { "$slice": 2 }})
    db.users.find({}, { "Languages": { "$slice": -3 }})
    db.users.find({}, { "Languages": { "$slice": [1, 2] }})

Returning an the matching array item.

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