---
title: SWARMDB API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: Node.js
  - go: Go
  - shell: HTTP

includes:
  - errors

search: true
---

# Introduction

SwarmDB is being developed by [Wolk Inc.](https://www.wolk.com/) to support decentralized database services.  

Prospective database users can use our NoSQL interfaces to store and retrieve data from a network of SwarmDB nodes that store chunks representing database records.  

This is a pre-alpha project under active development with Ethereum Swarm. To install the current version, please [README](https://github.com/wolktoken/swarm.wolk.com/blob/master/README.md). 

If you require further assistance, feel free to shoot us an email: [services@wolk.com](mailto:services@wolk.com)

# Configure

Upon installation and startup of your SWARMDB node, a default user and associated private key are generated and stored in the [SWARMDB configuration file](https://github.com/wolktoken/swarm.wolk.com/wiki/9.-SWARMDB-Server-Configuration,--Authentication-and-Voting#configuration-file).  

```javascript
//example: export PRIVATE_KEY=4b0d79af51456172dfcc064c1b4b8f45f363a80a434664366045165ba5217d53
$ export PRIVATE_KEY=PRIVATE_KEY_IN_YOUR_CONFIG_FILE
```

For Node.js API, you need to set the private key associated with this user as an environment variable in the terminal. 

# Install

```javascript
$ npm install swarmdb.js --save
// Note that since web3 module might have some issues to be installed at a latest version, you had better install it manually right now.
$ npm install web3@1.0.0-beta.26
```

```go
go get github.com/wolkdb/swarmdblib/
```

```shell
//Check that wolkdb is running and that you can do curl calls
$ ps aux | grep swarmdb
$ curl https://www.google.com | wc
```

To use SWARMDB API, you need to install the corresponding library.  
If you are using Node Shell in SwarmDB docker image, you can skip the install step.

```javascript
var swarmdbAPI = require("swarmdb.js");
```

```go
import "github.com/wolkdb/swarmdblib"
```

Import the library

# Connect
```javascript
// swarmdbAPI.createConnection(options)
var swarmdb = swarmdbAPI.createConnection({
    host: SWARMDB_HOST, //e.g. "localhost"
    port: SWARMDB_PORT  //e.g. 2001
});
```

```go
ip := "127.0.0.1"
port := int(2001)
owner := testowner.eth
conn, err := NewSWARMDBConnection(ip, port, owner)
```

```shell
For more information on this see https://github.com/wolkdb/swarm.wolk.com/wiki/5.-HTTP-Interface#authentication
```

Open a connection by specifying the host and port of the SWARMDB node.  Specific details may be found in the [SWARMDB configuration file](https://github.com/wolktoken/swarm.wolk.com/wiki/8.-SWARMDB-Server-Configuration,--Authentication-and-Voting#configuration-file).
Owner should be a valid ENS domain.

For Go API, if no IP, no port, or no owner address are defined (ip = 0, port = "", owner = "") then they will be pulled from the SwarmDB config file.  Using the config file assumes the node is running locally.


# Database

## Create Database

```javascript
// swarmdb.createDatabase(owner, databaseName, encrypted, callback)
swarmdb.createDatabase("test.eth", "testdb", 1, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});
// Output: {"affectedrowcount":1}
```

```go
name := "reallysmartdogs"
encrypted := int(1)
db, err := conn.CreateDatabase(name, encrypted)
```

```shell
// Create an encrypted database named testdb, which is owned by test.eth.
curl -X POST -d '{ "requesttype":"CreateDatabase", "encrypted":1 }'  \ 
     "http://localhost:8501/testdb.test.eth/" 
// Output: {"affectedrowcount":1}
```

Create a database by specifying database name and encrypted status.  

For encrypted status, 1 means true and 0 means false.

## Open Database 

```javascript
// swarmdb.openDatabase(owner, databaseName)
swarmdb.openDatabase("test.eth", "testdb");
```

```go
name := "smartdogs"
db, err  := conn.OpenDatabase(name)
```

```shell
// not required
```

Open an existing database by specifying owner and database name.

For Go API, owner will be the connection Object owner.

## List Databases

```javascript
// swarmdb.listDatabases(callback)
swarmdb.listDatabases(function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
// Output: {"data":[{"database":"testdb"}],"matchedrowcount":1}
```

```go
dblist, err := conn.ListDatabases()

//dblist contains:
[]swarmdblib.Row{
  swarmdblib.Row{"database": "smartdogs"},
  swarmdblib.Row{"database": "reallysmartdogs"},
}
```

```shell
// List the databases owned by test.eth.
curl -X POST -d '{ "requesttype":"ListDatabases" }'  \ 
     "http://localhost:8501/_.test.eth/"
// Output: {"data":[{"database":"testdb"}],"matchedrowcount":1}
```

Lists the Databases associated with a Connection.

# Table

For Node.js API, either createDatabase or openDatabase should be called prior to table method calls.

## Create Table

```javascript
// swarmdb.createTable(tableName, columns, callback)
var columns = [
  { "columnname": "email", "columntype": "STRING", "indextype": "BPLUS", "primary": 1 },
  { "columnname": "name", "columntype": "STRING", "indextype": "HASH", "primary": 0 },
  { "columnname": "age", "columntype": "INTEGER", "indextype": "BPLUS", "primary": 0 }
];
swarmdb.createTable("contacts", columns, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});
// Output: {"affectedrowcount":1}
```

```go
columns :=
  []swarmdblib.Column{
    swarmdblib.Column{
      ColumnName: "email",
      ColumnType: CT_STRING,
      IndexType: IT_BPLUSTREE,
      Primary: 1
    },
    swarmdblib.Column{
      ColumnName: "age",
      ColumnType: CT_INTEGER,
      IndexType: IT_BPLUSTREE,
      Primary: 0
    },
}
name := "softwareengineerdogs"
tbl, err := db.CreateTable(name, columns)
```

```shell
// Create a table named contacts, which belongs to the database named testdb, which is owned by test.eth.
// The contacts table has 3 columns
// 1) email is the primary key and is of type STRING and uses the BPLUS index
// 2) name is of type STRING and uses the HASH index
// 3) age is of type INTEGER and uses the BPLUS index

curl -X POST -d '{ "requesttype":"CreateTable", "table":"contacts", \
      "columns":[{ "columnName":"email", "ColumnType":"STRING", "Primary":1, "IndexType":BPLUS }, \ 
                 { "columnname":"name", "columntype":"STRING", "IndexType":"HASH" }, \
                 { "columnname":"age", "columntype":"INTEGER", "IndexType":"BPLUS" } ] }' \
      "http://localhost:8501/testdb.test.eth/"
// Output: {"affectedrowcount":1}
```

Create a table by specifying table name and column details.  

Columns consist of the following parameters:

* **indextype**: enter the desired indextype, defined [HERE](https://github.com/wolkdb/swarm.wolk.com/wiki/8.-SWARMDB-Types#table-index-types)
* **columnname**: enter the desired column name (no spaces allowed)
* **columntype**: enter the desired columntype, defined [HERE](https://github.com/wolkdb/swarm.wolk.com/wiki/8.-SWARMDB-Types#table-column-types)
* **primary**: enter 1 if the column is the primary key and 0 for any other column.

## Open Table 

```javascript
// swarmdb.openTable(tableName)
swarmdb.openTable("contacts");
```

```go
name := "accountantdogs"
tbl, err := db.OpenTable(name)
```

```shell
// not required
```

Open a table by specifying table name. 
In Go, a table object must be created using the database object.

## Describe Table

```javascript
// swarmdb.describeTable(tableName, callback)
swarmdb.describeTable("contacts", function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
// Output: {"data":[{"ColumnName":"email","ColumnType":"STRING","IndexType":"BPLUS","Primary":1},{"ColumnName":"name","ColumnType":"STRING","IndexType":"HASH","Primary":0},{"ColumnName":"age","ColumnType":"INTEGER","IndexType":"BPLUS","Primary":0}]}
```

```go
description, err := db.DescribeDatabase()

//description contains:
[]swarmdblib.Row{
  swarmdblib.Row{
    "ColumnName": "email",
    "ColumnType": CT_STRING,
    "IndexType":  IT_BPLUSTREE,
    "Primary":    float64(1)
  },
  swarmdblib.Row{
    "ColumnName": "age",
    "ColumnType": CT_INTEGER,
    "IndexType":  IT_BPLUSTREE,
    "Primary":    float64(0)
  },
}
```
Shows the Columns of an existing Table.

## List Tables

```javascript
// swarmdb.listTables(callback)
swarmdb.listTables(function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
// Output: {"data":[{"table":"contacts"}],"matchedrowcount":1}
```

```go

list, err := db.ListTables()

//list contains:
[]swarmdblib.Row{
  swarmdblib.Row{"table": "accountantdogs"},
  swarmdblib.Row{"table": "softwareengineerdogs"},
}

```

Lists the existing Tables in the specified Database.

# Write

Writing a row (or rows) to SWARMDB tables may be done via the PUT call or running a SQL INSERT/UPDATE query.

For Node.js API, either createTable or openTable should be called prior to Write calls.

## Put
```javascript
// swarmdb.put(rows, callback)
swarmdb.put( [ { "name": "Bertie Basset", "age": 7, "email": "bertie@gmail.com" } ],  function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
// Output: {"affectedrowcount":1}
```

```go
rowtoadd := Row{"email": "peanut@dogs.com", "age": 12}
rowstoadd :=
  []swarmdblib.Row{
    swarmdblib.Row{"email": "ralph@dogdog.com", "age": 7},
    swarmdblib.Row{"email": "nori@dogs.uk", "age": 1},
  }
err = tbl.Put(rowtoadd)
err = tbl.Put(rowstoadd)
```

```shell
// Insert into or Update the contacts table (in the testdb database, owned by test.eth) with the row {"email":"bertie@gmail.com", "name":"Bertie Basset", "age":7} 
curl -X POST -d '{ "requesttype":"Put", \
                   "row":{"email":"bertie@gmail.com", "name":"Bertie Basset", "age":7} }' \
     "http://localhost:8501/testdb.test.eth/contacts"
// Output: {"affectedrowcount":1}
```

Put calls allow for writing rows.

<aside class="notice">The "rows" input consists of an array rows, where a row is a defined as a json object containing column/value pairs.  Each row must at least include a column/value pair for the primary key.</aside>

## Insert
```javascript
// swarmdb.query(sqlQuery, callback)
swarmdb.query("INSERT INTO contacts(email, name, age) VALUES('bertie@gmail.com','Bertie',7);", function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
// Output: {"affectedrowcount":1}
```

```go
```

```shell
// Query (Insert) a row into the contacts table (in the testdb database, owned by test.eth)
curl -X POST -d '{ "requesttype":"Query", \
                   "query":"INSERT INTO contacts(email, name, age) VALUES(\"bertie@gmail.com\",\"Bertie Basset\",7) " }' \
     "http://localhost:8501/testdb.test.eth/"
// Output: {"affectedrowcount":1}
```

Insert Query calls allow for the insertion of rows by specifying an INSERT query using standard SQL. SWARMDB currently only supports one row insert per call.

## Update
```javascript
// swarmdb.query(sqlQuery, callback)
swarmdb.query("UPDATE contacts SET age=8 WHERE email='bertie@gmail.com';", function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
// Output: {"affectedrowcount":1}
```

```go
```

```shell
// Query (Update) a row in the contacts table (owned by test.eth owner ENS), setting the age to 8, where the email='bertie@gmail.com'
curl -X POST -d '{ "requesttype":"Query", \
                   "query":"UPDATE contacts SET age=8 WHERE email=\"bertie@gmail.com\" " }' \
     "http://localhost:8501/testdb.test.eth/"
// Output: {"affectedrowcount":1}
```
Update Query calls allow for the update on non-primary key using standard SQL.

# Read

Reading a row (or rows) from SWARMDB tables may be done via the GET call or running a Select query.

For Node.js API, either createTable or openTable should be called prior to Read calls.

## Get
```javascript
// swarmdb.get(key, callback)
swarmdb.get("bertie@gmail.com", function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
// Output: {"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```

```go
rowgotten = tbl.Get("peanut@dogs.com")

//rowgotten contains:
[]swarmdblib.Row{
  swarmdblib.Row{"age": 12, "email": "peanut@dogs.com"},
}

```

```shell
// Try to retrieve a row with the primary key value bertie@gmail.com from the contacts table (in the testdb database, owned by test.eth)
curl -X POST "http://localhost:8501/testdb.test.eth/contacts/bertie@gmail.com"
// Output: {"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```
Get allows for the retrieval of a single row by specifying the value of a row's primary key.

## Select
```javascript
// swarmdb.query(sqlQuery, callback)
swarmdb.query("SELECT email, name, age FROM contacts WHERE email = 'bertie@gmail.com'", function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
// Output: {"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```

```go
rowsgotten, err = tbl.Query("select email, age from testtable where age < 10")

//rowsgotten contains:
[]swarmdblib.Row{
  swarmdblib.Row{"email": "ralph@dogdog.com", "age": 7},
  swarmdblib.Row{"email": "nori@dogs.uk", "age": 1},
}
```

```shell
// Query the testdb database (owned by test.eth owner ENS)
curl -X POST -d '{ "requesttype":"Query", \
                   "query":"SELECT email, name, age FROM contacts WHERE email=\"bertie@gmail.com\" " }' \
     "http://localhost:8501/testdb.test.eth/"
// Output: {"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```

Select Query calls allow for the retrieval of rows by specifying a SELECT query using standard SQL.  The [supported query operands](https://github.com/wolktoken/swarm.wolk.com/wiki/8.-SWARMDB-Types#supported-query-operands) are allowed to be used on both primary and secondary keys.


