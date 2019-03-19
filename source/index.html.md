---
title: WOLK API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: Wolk API

includes:
  - errors

search: true
---

# Wolk: Trustless Web Apps

The Wolk blockchain embodies a next generation HTTP protocol (the Wolk protocl)
where web browsing is  _trustless_ and _serverless_, with web browsers interacting not with web servers but with a web blockchain.  

The Wolk protocol makes lock-in between users and developers impossible, enabling true digital freedom:

* user data is kept in user buckets eg wolk://owner/bucket
* developer code is kept in application buckets owned by the developer eg wolk://developer/app  
* there is no server held by developers holding user data hostage
* there is no platform holding developers hostage, only the Wolk protocol
* storage nodes of the blockchain cannot hold user data or developer code hostage
* storers earn rewards from running Wolk nodes, and developers earn rewards when applications are used.

## Wolk API

The Wolk API is a Javascript interface that enables developers of web content to store and retrieve data from the Wolk blockchain:

Wolk API Method  | HTTP Request Method / Path   | HTTP Request Body            | HTTP Response Body (200 OK) | HTTP Proof Response Header
---------------- | ---------------------------- | ---------------------------- | --------------------------- | ------
createAccount    | PUT wolk/account/owner       | JSON Account                 | txhash                      | N/A
getAccount       | GET wolk/account/owner       | -                            | JSON Account                | JSON SMTProof
getBlock         | GET wolk/block/blocknumber   | -                            | JSON Block                  | verified against blockchain
getLatestBlock   | GET wolk/block/latest        | -                            | JSON Block                  | verified against blockchain
getTransaction   | GET wolk/tx/txhash           | -                            | JSON Transaction            | verified against blockchain
createCollection | PUT owner/collection         | JSON Collection definition   | txhash                      | N/A
listCollections  | GET owner                    | -                            | JSON Collection             | JSON ScanProof
deleteCollection | DELETE owner/collection      | -                            | txhash                      | N/A
setKey           | PUT owner/collection/key     | ArrayBuffer or JSON Schema   | txhash                      | N/A
getKey           | GET owner/collection/key     | -                            | value                       | JSON NoSQLProof
scanCollection   | GET owner/collection         | -                            | []value                     | JSON ScanProof
deleteKey        | DELETE owner/collection      | -                            | txhash                      | N/A
createDatabase   | PUT owner/database/SQL       | JSON SQLRequest              | txhash                      | N/A
listDatabases    | GET owner/database/SQL       | JSON SQLRequest              | JSON Database               | N/A
deleteDatabase   | DELETE owner/database/SQL    | JSON SQLRequest              | txhash                      | N/A
createTable      | PUT owner/database/table     | JSON SQLRequest              | txhash                      | N/A
describeTable    | GET owner/database/table     | JSON SQLRequest              | JSON Columns                | JSON NoSQLProof
listTables       | GET owner/database           | JSON SQLRequest              | []JSON Table                | JSON ScanProof
dropTable        | DELETE owner/database/table  | JSON SQLRequest              | txhash                      | N/A
executeSQL       | POST owner/database/table    | JSON SQLRequest              | txhash                      | N/A
readSQL          | PATCH owner/database/table   | JSON SQLRequest              | JSON SQLResponse            | JSON STARKProof (Not implemented)

The Wolk Class abstracts all HTTP operations in the Wolk API methods that each interact with the Wolk Blockchain, described below.  
Unless otherwise noted, all Wolk API methods are asynchronous, supporting both a callback and promise pattern.  

## Request/Response Headers

Each interaction with a Wolk Blockchain involves submitting _signed_ HTTP requests with `Requester` being the users public JSON Web Key and `Sig` holding the users signature (64 hex).
A Wolk Browser/Extension will register a user private key with `createAccount` and is responsible for protecting the users keys and ensuring that application code does not
engage in signing actions without the user's permission.   
Current implementations have Wolk nodes support for ECDSA-256 with SHA-256 hashes, but in principle any number of curves and hashing functions could be used (e.g. Edwards with Blake).   

Request Method           | HTTP Request Headers         | HTTP Response Headers        
------------------------ | ---------------------------- | ----------------------------
GET                      | Sig, Requester, Proof        | Proof, Proof-Type                        
POST, PUT, DELETE, PATCH | Sig, Requester               | -                           

PUT, POST, DELETE, PATCH requests result in a transaction being submitted to the Wolk Blockchain with a txhash being returned immediately.  When consensus is achieved, the transaction is included in a minted block and GET requests can return proofs.     Wolk Browsers should send their `Proof` header in GET requests and Wolk nodes must send `Proof` and `Proof-Type` headers in addition to the both.  
Each Proof-type is described separately.

Wolk nodes run data sharing and consensus protocols that Wolk API users do not need to concern themselves with but are critical for decentralized storage.

## Request Options

GET requests uniformly support an JSON RequestOptions structure to control request behavior, with the following attributes:
 * blockNumber <int>
 * maxContentLength <int>
 * history <int>
 * range <string>



# Wolk Clients

To use the WOLK API and develop Wolk applications, obtain a browser or Chrome extension from:

 * [Wolk Browser](https://github.com/wolkdb/browser)
 * [Wolk Extension](https://github.com/wolkdb/extension)

Anyone can fork these browsers and extensions to develop additional Wolk Clients that interacts with a Wolk blockchain.

# Blockchain Methods

The API uses the following core Blockchain types:

 * JSON Account (indexed by `owner <string>`)
 * JSON Block (indexed by `blocknumber <int>`)
 * JSON Transaction (indexed by `txhash <string>`)

## createAccount

`createAccount(owner, account[, callback])`

* `owner <string>`
* `account <JSON Account>`
* `callback <Function>`
  * `err <Error>`
  * `txhash <string>`

Asynchronously creates account using owner name and JSON Account object.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.createAccount("jill", {"rsaPublicKey": "..."})
  .then(function(txhash) {
    console.log(txhash);
  })
  .catch(function(err) {
    console.error(err);
  })

// callback
wolk.createAccount("jack", {"rsaPublicKey": "..."}, function(err, txhash) {
    if ( err ) {
      throw(err);
    }
    console.log(txhash);
  })
```


## getAccount

`getAccount(owner[, callback])`

* `owner <string>`
* `callback <Function>`
  * `err <Error>`
  * `txhash <string>`

Asynchronously gets specific account by the account owner, originally created by `createAccount`.  If `callback` is not provided, returns a promise.


## getBlock

`getBlock(blockNumber[, callback])`

* `blockNumber <int>`
* `callback <Function>`
  * `err <Error>`
  * `block <JSON Block>`

Asynchronously gets specific block number.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.getBlock(42)
  .then(function(block) {
    console.log(block);
  })
  .catch(function(err) {
    console.error(err);
  })
// callback
wolk.getBlock(42, function(err, block) {
    if ( err ) {
      throw(err);
    }
    console.log(block);
  })
```

## getLatestBlock

`getLatestBlock([callback])`

* `blockNumber <int>`
* `callback <Function>`
  * `err <Error>`
  * `block <JSON Block>`

Asynchronously gets latest block.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.getLatestBlock()
  .then(function(block) {
    console.log(block);
  })
  .catch(function(err) {
    console.error(err);
  })
// callback
wolk.getLatestBlock(blockNumber, function(err, block) {
    if ( err ) {
      throw(err);
    }
    console.log(block);
  })
```

## getTransaction

`getTransaction(txhash[, callback])`

* `blockNumber <int>`
* `callback <Function>`
  * `err <Error>`
  * `tx <JSON Transaction>`

Asynchronously gets transaction using a transaction hash.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.getTransaction("98827bbd28f1cf65f91dbc0abdb7e50e6bb4a9d4fb69283b1ca5ff828c686a9b")
  .then(function(tx) {
    console.log(tx);
  })
  .catch(function(err) {
    console.error(err);
  })
// callback
wolk.getTransaction("98827bbd28f1cf65f91dbc0abdb7e50e6bb4a9d4fb69283b1ca5ff828c686a9b", function(err, tx) {
    if ( err ) {
      throw(err);
    }
    console.log(tx);
  })
```

# File/NoSQL Methods

The model for file storage and NoSQL is that keys and values are kept in an owner's collections.
The difference between File collections and NoSQL collections is that File collections are just blobs of bytes
and NoSQL collections are JSON records which may be indexed.

## createCollection

`createCollection(owner, collection[, callback])`

* `owner <string>`
* `collection <string>`
* `callback <Function>`
  * `err <Error>`
  * `txhash <string>`

Asynchronously creates new collection, returning a transaction hash.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.createCollection("bruce", "planets")
  .then(function(txhash) {
    console.log(txhash);
  })
  .catch(function(err) {
    console.error(err);
  })
// callback
wolk.createCollection("bruce", "planets", function(err, txhash) {
    if ( err ) {
      throw(err);
    }
    console.log(txhash);
  })
```

In the next release, we will have index specifiable in `CreateCollection`, with Range Proofs being returned in `scanCollection` responses and verified by Wolk clients.

## listCollections

`listCollections(owner[, callback])`

* `owner <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Asynchronously list an owner's collections, returning a list of collections.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.listCollections("bruce")
  .then(function(result) {
    console.log(result);
  })
  .catch(function(err) {
    console.error(err);
  })

// callback
wolk.listCollections("bruce", function(err, result) {
    if ( err ) {
      throw(err);
    }
    console.log(result);
  })
```

## deleteCollection

`deleteCollection(owner, collection)`

* `owner <string>`
* `collection <string>`
* `callback <Function>`
  * `err <Error>`
  * `txhash <string>`

Asynchronously delete an owner's collection, returning transaction hash.  If `callback` is not provided, returns a promise.

```javascript
// promise
wolk.deleteCollection("bruce", "planets",)
  .then(function(txhash) {
    console.log(txhash);
  })
  .catch(function(err) {
    console.error(err);
  })
// callback
wolk.deleteCollection("bruce", "planets", function(err, result) {
    if ( err ) {
      throw(err);
    }
    console.log(result);
  })
```

## setKey

`setKey(owner, collection, key, val[, callback])`

* `owner <string>`
* `collection <string>`
* `key <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Asynchronously sets a key-value pair in an owner's collection.  The collection must have already been created with `createCollection` and to retreive the value the caller must use `getKey`.  If `callback` is not provided, returns a promise.

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

## getKey

`getKey(owner, collection, key[, callback])`

* `owner <string>`
* `collection <string>`
* `key <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Asynchronously gets the value corresponding to a given key in the owner's collection.  Previously, the key-value pair must have been set with `setKey`.  If `callback` is not provided, returns a Promise.

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

## scanCollection

`scanCollection(owner, collection[, callback])`

* `owner <string>`
* `collection <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Asynchronously returns _all_ key-value pairs in a collection.  All key-value pairs have been set with `setKey` transactions.   If `callback` is not provided, returns a Promise.

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

## deleteKey

`deleteKey(owner, collection, key)`

* `owner <string>`
* `collection <string>`
* `key <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Asynchronously delete specific key-value pair in an owner's collection.  The key-value pair must have been set with `setKey` operations.  If `callback` is not provided, returns a Promise.

```javascript
// promise
wolk.deleteKey("bruce", "planets", "pluto")
  .then(function(txhash) {
    console.log(txhash)
  })
  .catch(function(err) {
    console.error(err)
  })

// callback
wolk.deleteKey("bruce", "planets", "pluto"), function(err, txhash) {
  if ( err ) {
    throw(err);
  }
  console.log(txhash);
})
```

# SQL Operations

The Wolk API uses the following data types in SQL Operations:
* JSON SQLRequest
* JSON SQLResponse

## createDatabase

`createDatabase(owner, database, options [, callback])`

* `owner <string>`
* `database <string>`
* `options <JSON RequestOptions>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Creates a database owned by owner.

# Qme1gTwdtFaUMoqwTNWX5SdYmNfiAj4N88qujbxmsX5z6u

```javascript
// promise
wolk.createDatabase("alina", "db03", {})
  .then(function(txhash) {
    console.log(txhash);
  })
  .catch(function(err) {
    console.error(err);
  })

// callback
wolk.createDatabase("alina", "db03", {}, function(err, txhash) {
  if ( err ) {
    throw(err)
  }
  console.log(txhash);
})
```

## listDatabases

`listDatabases(owner, options)`

* `owner <string>`
* `options <JSON RequestOptions>`
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

## deleteDatabase

`deleteDatabase(owner, database)`

* `owner <string>`
* `database <string>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Aysnchronously deleted a database by specifying owner and database name.

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

## createTable

`createTable(owner, database, table, columns, options[, callback])`

* `owner <string>`
* `database <string>`
* `table <string>`
* `column <JSON Array of Column>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Asynchronously creates a table with specific column definitions.  Note: current supported types for columns are `STRING`, `INTEGER`, `FLOAT` and a table must have a primary key.
If `callback` is not provided, returns a Promise.

```javascript
// promise
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

## describeTable

`describeTable(owner, database, table, options[, callback])`

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
{"data":[{"ColumnName":"name","ColumnType":"STRING","IndexType":"BPLUS","Primary":0},{"ColumnName":"person_id","ColumnType":"INTEGER","IndexType":"BPLUS","Primary":1}]}
```

## listTables

`listTables(owner, database, options)`

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
{
  result: {
    data: [{
        table: "demoitems"
    }],
    matchedrowcount: 1
  }
}
```

## dropTable

`dropTable(owner, database, table, options[, callback])`

* `owner <string>`
* `database <string>`
* `table <string>`
* `options <JSONObject>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Asynchronously drop a table by specifying table name in an owner's database.

```javascript
// promise
wolk.dropTable("alina", "db03", "person", {})
  .then( function(txhash) {
    console.log(txhash)
  })
  .catch( function(err) {
    console.error(err)
  })
```

```javascript
// callback
wolk.dropTable("alina", "db03", "person", {}, function(err, txhash) {
  if ( err ) {
    throw(err);
  }
  console.log(txhash)
})
```

## executeSQL

`executeSQL(owner, database, sql, options[, callback])`

* `owner <string>`
* `database <string>`
* `sql <string>`
* `options <JSONObject>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Asynchronously executes  SQL query in an owner's database.

### INSERT Query

```javascript
// promise
var sql_insert = "INSERT INTO demoitems(ID, Name, Description, Price, Currency, Hash) VALUES('ID1','Item1','Hot Item #1',0.11,'ethereum', 'Qmd286K6pohQcTKYqnS1YhWrCiS4gz7Xi34sdwMe9USZ7u')";
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
// promise
var sql_update = "UPDATE demoitems SET Price = 0.13 WHERE ID='ID2'"
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
// promise
var sql_delete = "DELETE FROM demoitems WHERE ID='ID1'"
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

## readSQL

`readSQL(owner, database, sql, options)`

* `owner <string>`
* `database <string>`
* `sql <string>`
* `options <JSONObject>`
* `callback <Function>`
  * `err <Error>`
  * `result <string>`

Executes read-only SQL query in an owner's database.

```javascript
// promise
var sql_select = "SELECT * FROM demoitems WHERE price > 0.10"
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
