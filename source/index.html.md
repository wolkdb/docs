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

If you require further assistance, feel free to shoot us an email: services@wolk.com

# Install

```javascript
$ npm install swarmdb.js --save
// Note that since web3 module has some issues to be installed at a latest version, you need to install it manually right now.
$ npm install web3@1.0.0-beta.26
```

```go
go get github.com/wolkdb/swarmdb
```

```shell
// Check that wolkdb is running and that you can do curl calls
$ ps aux | grep swarmdb
$ curl https://www.google.com | wc
```

To use SWARMDB API, you need to install the corresponding library.

```javascript
var swarmdbAPI = require("swarmdb.js");
```

```go
import "github.com/wolkdb/swarmdb"
```

Import the library in your project.

# Configure

Upon installation and startup of your SWARMDB node, a default user and associated private key are generated and stored in the [SWARMDB configuration file](https://github.com/wolktoken/swarm.wolk.com/wiki/9.-SWARMDB-Server-Configuration,--Authentication-and-Voting#configuration-file).  

```javascript
$ export PRIVATE_KEY=PRIVATE_KEY_IN_YOUR_CONFIG_FILE
```

Set the private key associated with this user as an environment variable. 

```javascript
$ export PROVIDER=YOUR_PROVIDER_LINK
```

(Optional) You can also specify an Ethereum provider instead of using the default http://localhost:8545

# Connect
```javascript
// swarmdbAPI.createConnection(options)
var swarmdb = swarmdbAPI.createConnection({
    host: SWARMDB_HOST, //e.g. "localhost"
    port: SWARMDB_PORT  //e.g. 2001
});
```

```go
// (db *SWARMDBDatabase) OpenConnection(options) 
options := SWARMDBConnectionOptions{"ip": SWARMDB_HOST, "port": SWARMDB_PORT}
conn, err := swarmdb.OpenConnection(options)
if err != nil {
   panic("could not open connection")
}
```

```shell
For more information on this see https://github.com/wolkdb/swarm.wolk.com/wiki/5.-HTTP-Interface#authentication
```

Open a connection by specifying the host and port of the SWARMDB node.  Specific details may be found in the [SWARMDB configuration file](https://github.com/wolktoken/swarm.wolk.com/wiki/8.-SWARMDB-Server-Configuration,--Authentication-and-Voting#configuration-file).

# Create Database

```javascript
// swarmdb.createDatabase(db, owner, encrypted, callback)
swarmdb.createDatabase("testdb", "test.eth", 1, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});
```

```go
// (db *SWARMDBConnection) CreateDatabase(owner string, databaseName string)
owner := "test.eth"
databaseName := "testdb"
db, err := conn.CreateDatabase(owner, databaseName)
if err != nil {
   panic("could not create database")
}
```

```shell
curl -X POST -d '{ "requesttype":"CreateDatabase", "encrypted":0 }'  \ 
     "http://localhost:8501/testdb.test.eth/" 
```

Create a database by specifying database name, owner and encrypted status.  

Owner should be a valid ENS domain.

For encrypted status, 1 means true and 0 means false.

# Create Table

```javascript
// swarmdb.createTable(table, db, owner, columns, callback)
var columns = [
  { "columnname": "email", "columntype": "string", "indextype": "bplus", "primary": 1 },
  { "columnname": "name", "columntype": "string", "indextype": "hash" },
  { "columnname": "age", "columntype": "integer", "indextype": "bplus" }
];
swarmdb.createTable("contacts", "testdb", "test.eth" columns, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});
```

```go
// (db *SWARMDBDatabase) CreateTable(tableName string, []Column)
// Create Table
tableName := "contacts"
cols := make([]swarmdb.Column, 3)
cols[0].ColumnName = "email"
cols[0].Primary = 1
cols[0].IndexType = "bplus"
cols[0].ColumnType = "string"

cols[1].ColumnName = "name"
cols[1].IndexType = "hash"
cols[1].ColumnType = "string"

cols[2].ColumnName = "age"
cols[2].IndexType = "bplus"
cols[2].ColumnType = "integer"

tbl, err := db.CreateTable(tableName, cols)
if err != nil {
   panic("could not create table")
}
```

```shell
curl -X POST -d '{ "requesttype":"CreateTable", "table":"contacts", \
      "columns":[{ "columnName":"email", "ColumnType":"string", "Primary":1, "IndexType":”bplus” }, \ 
                 { "columnname":"name", "columntype":"string", "IndexType":"hash" }, \
                 { "columnname":"age", "columntype":"integer", "IndexType":"bplus" } ] }' \
      "http://localhost:8501/testdb.test.eth/"
```

Create a table by specifying table name, database name, owner and column details.  

Columns consist of the following parameters:  

* **indextype**: enter the integer value corresponding with the desired indextype ([more details](https://github.com/wolktoken/swarm.wolk.com/wiki/8.-SWARMDB-Types#table-column-types))  
* **columnname**: enter the desired column name (no spaces allowed)  
* **columntype**: enter the integer value corresponding with the desired columntype ([more details](https://github.com/wolktoken/swarm.wolk.com/wiki/8.-SWARMDB-Types#table-index-types))  
* **primary**: enter 1 if the column is the primary key and 0 for any other column  

# Opening a Database 

In Go, a database object should be created using the connection.

```javascript
// not required
```

```go
// (db *SWARMDBConnection) Open(owner string, databaseName string)
owner := "test.eth"
databaseName := "testdb"
db, err := conn.OpenDatabase(owner, databaseName)
if err != nil {
   panic("could not create database")
}
```

# Opening a Table 

To open a table, a table object must be created using the database object.

```javascript
// TBD
```


```go
// (tbl *SWARMDBTable) Open(tableName string)
tbl, err := db.Open(tableName)
if err != nil {
   panic("could not open table")
}
```

```shell
// not required
```


# Read

Reading a row (or rows) from SWARMDB tables may be done via the GET call or running a SQL Select query.

Table owner, which is an Ethereum address without "0x", is required to read a row.

## Get
```javascript
// swarmdb.get(table, tableowner, key, callback)
var tableowner = "ADDRESS_IN_YOUR_CONFIG_FILE";
swarmdb.get("contacts", tableowner, "bertie@gmail.com", function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
```

```go
// (tbl *SWARMDBTable) Get(key string)
row, err := tbl.Get("bertie@gmail.com")
if err != nil {
   panic("could not Get row")
}
fmt.Printf("Row: %v\n")
```

```shell
curl -X POST "http://localhost:8501/testdb.test.eth/contacts/bertie@gmail.com"
```
Get calls allow for the retrieval of a single row by specifying the value of a rows primary key.

## Select
```javascript
// swarmdb.query(sqlQuery, tableowner, callback)
var tableowner = "ADDRESS_IN_YOUR_CONFIG_FILE";
swarmdb.query("SELECT email, name, age FROM contacts WHERE email = 'bertie@gmail.com'", tableowner, function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
```

```go
// (db *SWARMDBDatabase) Query(query string) (rows []Row, err error)
query := "SELECT email, name, age FROM contacts WHERE email=\"bertie@gmail.com\""
rows, err := db.Query(query)
if err != nil {
   panic("could not Get row")
}
for i, row := range rows {
    fmt.Printf("Row %d: %v\n", i, row)
}
```

```shell
curl -X POST -d '{ "requesttype":"Query", \
                   "query":"SELECT email, name, age FROM contacts WHERE email=\"bertie@gmail.com\" " }' \
     "http://localhost:8501/testdb.test.eth/"
```

Select Query calls allow for the retrieval of rows by specifying a SELECT query using standard SQL.  The [supported query operands](https://github.com/wolktoken/swarm.wolk.com/wiki/8.-SWARMDB-Types#supported-query-operands) are allowed to be used on both primary and secondary keys.

# Write
Writing a row (or rows) to SWARMDB tables may be done via the PUT call or running a SQL INSERT query.

Table owner, which is an Ethereum address without "0x", is required to read a row.

## Put
```javascript
// swarmdb.put(table, tableowner, rows, callback)
var tableowner = "ADDRESS_IN_YOUR_CONFIG_FILE";
swarmdb.put("contacts", tableowner, [ { "name": "Bertie Basset", "age": 7, "email": "bertie@gmail.com" } ],  function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
```

```go
// (table *SWARMDBTable) Put(row Row) (error err)
var r swarmdb.Row
r.Cells = make(map[string]interface{})
r.Cells["email"] = "bertie@gmail.com"
r.Cells["name"] = "Bertie Basset"
r.Cells["age"] = 7
err := tbl.Put(row)
if err != nil {
   panic("could not Put row")
}
```

```shell
curl -X POST -d '{ "requesttype":"Put", \
                   "row":{"email":"bertie@gmail.com", "name":"Bertie Basset", "age":7} }' \
     "http://localhost:8501/testdb.test.eth/contacts"
```

Put calls allow for writing rows.

<aside class="notice">The "rows" input consists of an array rows, where a row is a defined as a json object containing column/value pairs.  Each row must at least include a column/value pair for the primary key.</aside>

## Insert
```javascript
// swarmdb.query(sqlQuery, tableowner, callback)
var tableowner = "ADDRESS_IN_YOUR_CONFIG_FILE";
swarmdb.query("INSERT INTO contacts(email, name, age) VALUES('bertie@gmail.com','Bertie',7);", tableowner, function (err, result) {
    if (err) {
      throw err;
    }
    console.log(result);
});
```

```go
// (db *SWARMDBConnection) Query(query string) (rows []Row, error)
query := "INSERT INTO contacts(email, name, age) VALUES(‘bertie@gmail.com’,’Bertie Basset’,7)"
rows, err := db.Query(query)
if err != nil {
   panic("could not Get row")
}
```

```shell
curl -X POST -d '{ "requesttype":"Query", \
                   "query":"INSERT INTO contacts(email, name, age) VALUES(‘bertie@gmail.com’,’Bertie Basset’,7) " }' \
     "http://localhost:8501/testdb.test.eth/"
```

Insert Query calls allow for the insertion of rows by specifying an INSERT query using standard SQL. SWARMDB currently only supports one row insert per call.
