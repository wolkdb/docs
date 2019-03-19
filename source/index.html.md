---
title: WOLK API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: javascript

includes:
  - errors

search: true
---

# Wolk: Trustless Web Browsing and Trustless Appstores

The Wolk blockchain enables trustless, serverless web browsing model making lock-in between users and developers impossible:
* user data is kept in user buckets eg wolk://owner/bucket
* developer code is kept in application buckets owned by the developer eg wolk://developer/app  
* there is no server held by developers holding user data hostage,
* there is no platform holding developers hostage, only a protocol
* storage nodes of the blockchain cannot hold user data or developer code hostage

The Wolk API is a Javascript interface, enables developers of web content to store and retrieve data from user buckets

Storers earn rewards from running Wolk nodes, and developers earn rewards when applications.

To use the WOLK API and develop Wolk applications, obtain a browser or Chrome extension from:

* Wolk Browser: https://github.com/wolkdb/browser
* Wolk Extension: https://github.com/wolkdb/extension

Both expose the WOLK API with a Wolk class with methods that interact with the HTTP Blockchain.

# NoSQL Operations

## createCollection(owner, collection[, callback])

* `owner <string>`
* `collection <string>`
* `callback <Function>``
  * `err <Error>`
  * `txhash <string>`

Asynchronously creata new collection, returning a transaction hash.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.createCollection(owner, collection)
  .then(function(txhash) {
    console.log(txhash);
  })
  .catch(function(err) {
    console.error(err);
  })
```

```javascript
// callback
wolk.createCollection(owner, collection, function(err, txhash) {
    if ( err ) {
      throw(err);
    }
    console.log(txhash);
  })
```

## listCollections(owner[, callback]

* `owner <string>`
* `callback <Function>``
  * `err <Error>`
  * `result <string>`

Asynchronously list an owner's collections, returning a list of collections.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.listCollections(owner, collection)
  .then(function(result) {
    console.log(result);
  })
  .catch(function(err) {
    console.error(err);
  })
```

```javascript
// callback
wolk.listCollections(owner, collection, function(err, result) {
    if ( err ) {
      throw(err);
    }
    console.log(result);
  })
```

## deleteCollection(owner, collection)

* `owner <string>`
* `collection <string>`
* `callback <Function>``
  * `err <Error>`
  * `txhash <string>`

Asynchronously delete an owner's collection, returning transaction hash.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.deleteCollection(owner, collection)
  .then(function(txhash) {
    console.log(txhash);
  })
  .catch(function(err) {
    console.error(err);
  })
```

```javascript
// callback
wolk.deleteCollection(owner, collection, function(err, result) {
    if ( err ) {
      throw(err);
    }
    console.log(result);
  })
```

## setKey(owner, collection, key, val[, callback])

* `owner <string>`
* `collection <string>`
* `key <string>``
* `callback <Function>``
  * `err <Error>`
  * `result <string>`

Asynchronously set a key-value pair in an owner's collection.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.setKey("bruce", "planets", "pluto", "small")
  .then(function(txhash) {
    console.log(txhash)
  })
  .catch(function(err) {
    console.error(err)
  })

// callback
wolk.setKey("bruce", "planets", "pluto", "small", function(err, txhash) {
  if ( err ) {
    throw(err);
  }
  console.log(txhash);
})
```

## getKey(owner, collection, key[, callback])

* `owner <string>`
* `collection <string>`
* `key <string>``
* `callback <Function>``
  * `err <Error>`
  * `result <string>`

Asynchronously gets the value corresponding to a given key in the owner's collection.  If `callback` is not provided, returns a Promise.

```javascript
// promise
wolk.getKey("bruce", "planets", "pluto")
  .then(function(result) {
    console.log(result);
  })

// callback
wolk.getKey("bruce", "planets", "pluto", function (err, result) {
  if ( err ) {
    throw(err);
  }
  console.log(result);
})
```

## scanCollection(owner, collection[, callback])  

* `owner <string>`
* `collection <string>`
* `callback <Function>``
  * `err <Error>`
  * `result <string>`

Asynchronously returns _all_ key-value pairs in a collection.   If `callback` is not provided, returns a Promise.

```javascript
// promise
wolk.scanCollection("bruce", "planets")
  .then(function(result) {
    console.log(result);
  })
  .catch(function(err) {
    console.error(err);
  })

// callback
wolk.scanCollection("bruce", "planets", function (err, result) {
  if ( err ) {
    throw(err);
  }
  console.log(result);
})  
```

## deleteKey(owner, collection, key)

* `owner <string>`
* `collection <string>`
* `key <string>``
* `callback <Function>``
  * `err <Error>`
  * `result <string>`

Asynchronously delete specific key-value pair in an owner's collection.   If `callback` is not provided, returns a Promise.

```javascript
// promise
wolk.deleteKey("bruce", "planets", "pluto")
  .then(function(txhash) {
    console.log(txhash)
  })
  .catch(function(err) {
    console.error(err)
  })

// promise
wolk.deleteKey("bruce", "planets", "pluto"), function(err, txhash) {
  if ( err ) {
    throw(err);
  }
  console.log(txhash);
})
```

# SQL Operations

## createDatabase(owner, database, options [, callback])

* `owner <string>`
* `database <string>`
* `options <JSONObject>`
* `callback <Function>``
  * `err <Error>`
  * `result <string>`

Creates a database owned by owner.

```javascript
// promise
wolk.createDatabase("alina", "db03", {})
  .then(function(txhash) {
    console.log(txhash)
  })
  .catch(function(err) {
    console.error(err)
  })

// callback
wolk.createDatabase("alina", "db03", {}, function(err, txhash) {
  if ( err ) {
    throw(err)
  }
  console.log(txhash);
}
```

## listDatabases(owner, options)

* `owner <string>`
* `options <JSONObject>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

List databases owned by a user.  

```javascript
// promise
wolk.listDatabases("alina", {})
  .then(function(result) {
    console.log(result);
  })
  .catch(function(err) {
    console.error(err);
  })

// callback
wolk.listDatabases("alina", {}, function(err, result) {
  if ( err ) {
    throw(err)
  }
  console.log(result);
});

{
  result: {
    data: [{
        database: "testdb"
    }],
    matchedrowcount: 1
  }
}
```

## deleteDatabase(owner, database)

* `owner <string>`
* `database <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Delete a database by specifying owner and database name.

```javascript
// promise
wolk.deleteDatabase("alina", "db03")
  .then(function(txhash) {
    console.log(txhash)
  })
  .catch(function(err) {
    console.error(err)
  })

// callback
wolk.deleteDatabase("alina", "db03", function(err, txhash) {
  if ( err ) {
    throw(err);
  }
  console.log(txhash)
})
```

## createTable(owner, database, table, columns, options[, callback])

* `owner <string>`
* `database <string>`
* `table <string>`
* `column <column[]>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`


Create a table with specific column definitions.  Note: current supported types for columns are `STRING`, `INTEGER`, `FLOAT` and a table must have a primary key.

```javascript
let columns = [{
    "columnname": "person_id",
    "indextype": "BPLUS",
    "columntype": "INTEGER",
    "primary": 1
  }, {
    "columnname": "name",
    "indextype": "BPLUS",
    "columntype": "STRING"
  }]

// promise
wolk.createTable("alina", "db03", "person", columns, {})
  .then( function(txhash) {
    console.log(txhash)
  })
  .catch( function(err) {
    console.error(err)
  })
{"txhash": "98827bbd28f1cf65f91dbc0abdb7e50e6bb4a9d4fb69283b1ca5ff828c686a9b"}

// callback
wolk.createTable("alina", "db03", "person", columns, {}, function(err, txhash) {
  if ( err ) {
    throw(err);
  }
  console.log(txhash)
})
{"txhash": "98827bbd28f1cf65f91dbc0abdb7e50e6bb4a9d4fb69283b1ca5ff828c686a9b"}
```

## describeTable(owner, database, table, options[, callback])

* `owner <string>`
* `database <string>`
* `table <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Shows the definition of a table in an owner's database.

```javascript
// promise
wolk.describeTable("alina", "db03", "person", {})
   .then( function(result) {
     let res = JSON.parse(result);
     console.log(res)
   })
   .catch( function(err) {
     console.error(err);
   })

// callback
wolk.describeTable("alina", "db03", "person", {}, function(err, result) {
     let res = JSON.parse(result);
     console.log(res)
   })

// result:
{"data":[{"ColumnName":"name","ColumnType":"STRING","IndexType":"BPLUS","Primary":0},{"ColumnName":"person_id","ColumnType":"INTEGER","IndexType":"BPLUS","Primary":1}]}
```

## listTables(owner, database, options)

* `owner <string>`
* `database <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Lists the tables in an owner's database.

```javascript
// promise
wolk.listTables("alina", "db03", {})
  .then(  function(result) {
     console.log("list tables:", result)
     let res = JSON.parse(result);
   })
  .catch( function(err) {
    console.error(err)
  })

// callback
wolk.listTables("alina", "db03", {}, function(err, result) {
     console.log("list tables:", result)
     let res = JSON.parse(result);
   })

// result:
{
  result: {
    data: [{
        table: "demoitems"
    }],
    matchedrowcount: 1
  }
}
```

## executeSQL(owner, database, sql, options[, callback])

* `owner <string>`
* `database <string>`
* `sql <string>`
* `options <JSONObject>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Executes transactional SQL query in an owner's database.

### INSERT Query

```javascript
var sql_insert = "INSERT INTO demoitems(ID, Name, Description, Price, Currency, Hash) VALUES('ID1','Item1','Hot Item #1',0.11,'ethereum', 'Qmd286K6pohQcTKYqnS1YhWrCiS4gz7Xi34sdwMe9USZ7u')";
var queryCommand = { "requesttype":"Query", "owner":"wolkdb.eth", "database":"testdb", "query": sql1 };

// promise
wolk.executeSQL("alina", "db03", sql_insert, {})
  .then(function(txhash) {
    console.error(txhash)
  })
  .catch(function(err) {
    console.error(err)
  })

// callback
wolk.executeSQL("alina", "db03", sql_insert, {}, function(err, txhash) {
  if ( err ) {
      throw(error);
  }
  console.log(txhash)
})
```

### UPDATE Query

```javascript
var sql_update = "UPDATE demoitems SET Price = 0.13 WHERE ID='ID2'"
// promise
wolk.executeSQL("alina", "db03", sql_update, {})
  .then(function(txhash) {
    console.error(txhash)
  })
  .catch(function(err) {
    console.error(err)
  })

// callback
wolk.executeSQL("alina", "db03", sql_update, {}, function(err, txhash) {
  if ( err ) {
      throw(error);
  }
  console.log(txhash)
})
```


### DELETE Query

```javascript
var sql_delete = "DELETE FROM demoitems WHERE ID='ID1'"
// promise
wolk.executeSQL("alina", "db03", sql_delete, {})
  .then(function(txhash) {
    console.error(txhash)
  })
  .catch(function(err) {
    console.error(err)
  })

// callback
wolk.executeSQL("alina", "db03", sql_delete, {}, function(err, txhash) {
  if ( err ) {
      throw(error);
  }
  console.log(txhash)
})
```

## readSQL(owner, database, sql, options)

* `owner <string>`
* `database <string>`
* `sql <string>`
* `options <JSONObject>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Executes read-only SQL query in an owner's database.

```javascript
var sql_select = "SELECT * FROM demoitems WHERE price > 0.10"

// promise
wolk.readSQL("alina", "db03", sql_select, {})
  .then(function(result) {
    let res = JSON.parse(result);
    console.log(res.data);
  })
  .catch(function(err) {
    console.error(err)
  })

// callback
wolk.readSQL("alina", "db03", sql_select, {}, function(err, result) {
  if ( err ) {
    throw(err);
  }
  let res = JSON.parse(result);
  console.log(res.data);
});

// result:
{
  result: {
    data: [{
        ID: "ID1",
        Name: "Item1",
        Description: "Hot Item #1",
        Price:0.11,
        Currency:'ethereum',
        Hash:'Qmd286K6pohQcTKYqnS1YhWrCiS4gz7Xi34sdwMe9USZ7u'
    },{
        ID: "ID2",
        Name: "Item2",
        Description: "Hot Item #2",
        Price:0.09,
        Currency:'ethereum',
        Hash:'Qme1gTwdtFaUMoqwTNWX5SdYmNfiAj4N88qujbxmsX5z6u'
    }],
    matchedrowcount: 2
  }
}
```

## dropTable(owner, database, table, options[, callback])

* `owner <string>`
* `database <string>`
* `table <string>`
* `options <JSONObject>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Drop a table by specifying table name.

```javascript
// promise
wolk.dropTable("alina", "db03", "person", {})
  .then( function(txhash) {
    console.log(txhash)
  })
  .catch( function(err) {
    console.error(err)
  })

// callback
wolk.dropTable("alina", "db03", "person", {}, function(err, txhash) {
  if ( err ) {
    throw(err);
  }
  console.log(txhash)
})
```
