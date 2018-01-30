---
title: SWARMDB API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: Node.js
  - go: Go
  - shell: CLI
  - http: HTTP

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
GO
```

```shell
CLI
```

```http
HTTP
```

To use SWARMDB API, you need to install the corresponding library.

```javascript
var swarmdbAPI = require("swarmdb.js");
```

```go
GO
```

```shell
CLI
```

```http
HTTP
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
GO
```

```shell
CLI
```

```http
HTTP
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
GO
```

```shell
CLI
```

```http
HTTP
```

Create a database by specifying database name, owner and encrypted status.  

Owner should be a valid ENS domain.

For encrypted status, 1 means true and 0 means false.

# Create Table

```javascript
// swarmdb.createTable(table, db, owner, columns, callback)
var columns = [
  { "indextype": 2, "columnname": "email", "columntype": 2, "primary": 1 },
  { "indextype": 2, "columnname": "name", "columntype": 2, "primary": 0 },
  { "indextype": 2, "columnname": "age", "columntype": 1, "primary": 0 }
];
swarmdb.createTable("contacts", "testdb", "test.eth" columns, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});
```

```go
GO
```

```shell
CLI
```

```http
HTTP
```

Create a table by specifying table name, database name, owner and column details.  

Columns consist of the following parameters:  

* **indextype**: enter the integer value corresponding with the desired indextype ([more details](https://github.com/wolktoken/swarm.wolk.com/wiki/8.-SWARMDB-Types#table-column-types))  
* **columnname**: enter the desired column name (no spaces allowed)  
* **columntype**: enter the integer value corresponding with the desired columntype ([more details](https://github.com/wolktoken/swarm.wolk.com/wiki/8.-SWARMDB-Types#table-index-types))  
* **primary**: enter 1 if the column is the primary key and 0 for any other column  

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
GO
```

```shell
CLI
```

```http
HTTP
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
GO
```

```shell
CLI
```

```http
HTTP
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
GO
```

```shell
CLI
```

```http
HTTP
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
GO
```

```shell
CLI
```

```http
HTTP
```

Insert Query calls allow for the insertion of rows by specifying an INSERT query using standard SQL. SWARMDB currently only supports one row insert per call.
