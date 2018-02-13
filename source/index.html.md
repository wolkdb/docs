---
title: SWARMDB API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: Node.js
  - go: Go
  - plaintext: HTTP

includes:
  - errors

search: true
---

# Introduction

SwarmDB is being developed by [Wolk Inc.](https://www.wolk.com/) to support decentralized database services.  

Prospective database users can use our NoSQL interfaces to store and retrieve data from a network of SwarmDB nodes that store chunks representing database records.  

This is a pre-alpha project under active development with Ethereum Swarm. To install the current version of SWARMDB, please see [README](https://github.com/wolkdb/swarm.wolk.com/blob/master/README.md). 

If you require further assistance, feel free to shoot us an email: [services@wolk.com](mailto:services@wolk.com)

# Install

To use SWARMDB API, you need to install the corresponding library.
If you are using Node Shell in SwarmDB docker image, the packages are already installed. You can skip this step and move onto next section.

```javascript
//Getting swarmdb and web3 package
$ npm install swarmdb.js --save
$ npm install web3@1.0.0-beta.26

//Using the package
var swarmdbAPI = require("swarmdb.js");

/* Note:
Some users have experienced problems with auto-installing the latest web3 package.
In case you have similar issue, we recommed you to do a clean web3 install manually. */


```
```go
//package installation
go get github.com/wolkdb/swarmdblib
```

```plaintext
//Verifying that wolkdb is running
$ ps aux | grep swarmdb
$ curl https://www.google.com | wc
```

# Configure

Upon installation and startup of your SWARMDB node, a default user and associated private key are generated and stored in the [SWARMDB configuration file](https://github.com/wolkdb/swarm.wolk.com/wiki/9.-SWARMDB-Server-Configuration,--Authentication-and-Voting#configuration-file). 
By default, the config file can be found at: `/usr/local/swarmdb/etc/swarmdb.conf`

```plaintext
Note: for sending HTTP requests, please follow the URL path construction conventions detailed below:

http://[HOST]:[PORT]/[DATABASENAME].[OWNERENS]/[TABLE]
- [HOST]: For singleton mode, this is typically "localhost", however, the actual IP is acceptable as well
- [PORT]: 8501 is the default PORT
- [DATABASENAME]: The name of the Database being manipulated or accessed.  When creating a Database, The [DATABASENAME] portion, as well as the trailing "." should be left of- [OWNERENS]: The "Owner" of the Database being manipulated or accessed.  This will always follow the format of x.y (i.e. wolk.eth, owner.name, etc ...)
- [OWNER]: The "Owner ENS" of the Database being manipulated or accessed.  This will always follow the format of x.y (i.e. wolk.eth, owner.name, etc ...)
- [TABLE]: The table, belonging to the database noted, which is owned by the Owner noted that is to be manipulated or accessed
```


```go
//import this library
import "github.com/wolkdb/swarmdblib"
```

```javascript
//Setting private key as environment variable
$ export PRIVATE_KEY=PRIVATE_KEY_IN_YOUR_CONFIG_FILE
//example: export PRIVATE_KEY=4b0d79af51456172dfcc064c1b4b8f45f363a80a434664366045165ba5217d53
```

For Node.js API, you need to set the private key associated with this user as an environment variable in the terminal. 


# Connect
```javascript
//Connecting to SWARMDB node
var SWARMDB_HOST = "localhost"
var SWARMDB_PORT = 2001
var conn = swarmdbAPI.createConnection({
    host: SWARMDB_HOST,
    port: SWARMDB_PORT
});
```

```go
//func NewSWARMDBConnection(ip string, port int, owner string, privateKey string) (dbc SWARMDBConnection, err error) 

ip := "127.0.0.1" //your SWARMDB node IP
port := int(2001) //your SWARMDB node Port number
owner := "db4db066584dea75f4838c08ddfadc195225dd80" //your SWARMDB node owner address
privateKey := "98b5321e784dde6357896fd20f13ac6731e9b1ea0058c8529d55dde276e45624" //your SWARMDB node private key

conn, err := NewSWARMDBConnection(ip, port, owner, privateKey)
```

```plaintext
For more information on this see https://github.com/wolkdb/swarm.wolk.com/wiki/9.-SWARMDB-Server-Configuration,--Authentication-and-Voting#http-authentication
```

Open a connection by specifying the host and port of the SWARMDB node.  Specific details may be found in the [SWARMDB configuration file](https://github.com/wolkdb/swarm.wolk.com/wiki/8.-SWARMDB-Server-Configuration,--Authentication-and-Voting#configuration-file).
Owner should be a valid ENS domain.


# Database

## Create Database

```javascript
//Creating Database
var owner = "test.eth"
var databaseName= "testdb"
var encrypted = 1
conn.createDatabase(owner, databaseName, encrypted, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});
// Output: {"affectedrowcount":1}
```

```go
//Creating Database
//func (dbc *SWARMDBConnection) CreateDatabase(name string, encrypted int) (db *SWARMDBDatabase, err error)

databaseName := "testdb"
encrypted := int(1)
db, err := conn.CreateDatabase(databaseName, encrypted)
if err != nil {
    fmt.Printf(err.Error())
}
```

```plaintext
// Create an encrypted database named testdb, which is owned by test.eth.
curl -X POST -d '{ "requesttype":"CreateDatabase", "encrypted":1 }'   "http://localhost:8501/testdb.test.eth/" 

// Output: {"affectedrowcount":1}
```

Create a database by specifying database name and encrypted status.  

For encrypted status, 1 means true and 0 means false.

## Open Database 

```javascript
//Open Database
var owner = "test.eth"
var databaseName= "testdb"
conn.openDatabase(owner, databaseName);

//Output: No return values currently
```

```go
//Open Database
databaseName := "testdb"
db, err  := conn.OpenDatabase(databaseName)
if err != nil {
    fmt.Printf(err.Error())
}

//func (dbc *SWARMDBConnection) OpenDatabase(name string) (db *SWARMDBDatabase, err error)
db, err  := conn.OpenDatabase(databaseName)
if err != nil {
    fmt.Printf( err.Error() )
}
```

```plaintext
// not required
```

Open an existing database by specifying owner and database name.

For Go API, owner will be the connection Object owner.

## List Databases

```javascript
//List Databases
conn.listDatabases(function (err, dblist) {
    if (err) {
      throw err;
    }
    console.log(dblist);
});

// Output: {"data":[{"database":"testdb"}],"matchedrowcount":1}
```

```go
//List Databases
//func (dbc *SWARMDBConnection) ListDatabases() (databases []Row, err error)

dblist, err := conn.ListDatabases()
if err != nil {
    fmt.Printf(err.Error())
}

//output:
type Row map[string]interface{}
dblist = []Row{
  swarmdblib.Row{"database": "testdb"},
}
```

```plaintext
// List the databases owned by test.eth.
curl -X POST -d '{ "requesttype":"ListDatabases" }'  "http://localhost:8501/test.eth/"

// Output: {"data":[{"database":"testdb"}],"matchedrowcount":1}
```

Lists the Databases associated with a Connection.

# Table

For Node.js API, either createDatabase or openDatabase should be called prior to table method calls.

## Create Table

```javascript
//Create Table
var tableName = "contacts"
var columns = [
  { "columnname": "email", "columntype": "STRING", "indextype": "BPLUS", "primary": 1 },
  { "columnname": "name", "columntype": "STRING", "indextype": "HASH", "primary": 0 },
  { "columnname": "age", "columntype": "INTEGER", "indextype": "BPLUS", "primary": 0 }
];
conn.createTable(tableName, columns, function (err, tbl) {
  if (err) {
    throw err;
  }
  console.log(tbl);
});

// Output: {"affectedrowcount":1}
```

```go
//Create Table
//func (db *SWARMDBDatabase) CreateTable(name string, columns []Column) (tbl *SWARMDBTable, err error)

columns :=
  []Column{
    Column{
      ColumnName: "email",
      ColumnType: CT_STRING,
      IndexType: IT_BPLUSTREE,
      Primary: 1
    },
    Column{
      ColumnName: "age",
      ColumnType: CT_INTEGER,
      IndexType: IT_BPLUSTREE,
      Primary: 0
    },
}
tableName := "contacts"
tbl, err := db.CreateTable(tableName, columns)
if err != nil {
    fmt.Printf( err.Error() )
}
```

```plaintext
// Create a table named contacts, which belongs to the database named testdb, which is owned by test.eth.
// The contacts table has 3 columns
// 1) email is the primary key and is of type STRING and uses the BPLUS index
// 2) name is of type STRING and uses the HASH index
// 3) age is of type INTEGER and uses the BPLUS index

curl -X POST -d '{ "requesttype":"CreateTable", "table":"contacts", "columns":[{ "columnName":"email", "ColumnType":"STRING", "Primary":1, "IndexType":"BPLUS" }, { "columnname":"name", "columntype":"STRING", "IndexType":"HASH" }, { "columnname":"age", "columntype":"INTEGER", "IndexType":"BPLUS" } ] }' "http://localhost:8501/testdb.test.eth/"
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
//Open Table
var tableName = "contacts";
conn.openTable(tableName);

//Output: No return values currently
```

```go
//Open Table for manipulating
tableName := "contacts"
//func (db *SWARMDBDatabase) OpenTable(name string) (tbl *SWARMDBTable, err error)
tbl, err := db.OpenTable(tableName)
if err != nil {
    fmt.Printf( err.Error() )
}
```

```plaintext
// not required
```

Open a table by specifying table name. 
In Go, a table object must be created using the database object.

## Describe Table

```javascript
//Describe Table
var tableName = "contacts"
conn.describeTable(tableName, function (err, description) {
    if (err) {
      throw err;
    }
    console.log(description);
});

// Output: {"data":[{"ColumnName":"email","ColumnType":"STRING","IndexType":"BPLUS","Primary":1},{"ColumnName":"name","ColumnType":"STRING","IndexType":"HASH","Primary":0},{"ColumnName":"age","ColumnType":"INTEGER","IndexType":"BPLUS","Primary":0}]}
```

```go
//Describe Table
//func (tbl *SWARMDBTable) DescribeTable() (description []Row, err error) 
description, err := db.DescribeTable()
if err != nil {
    fmt.Printf(err.Error())
}

//output:
type Row map[string]interface{}
description = 
  []Row{
    Row{
      "ColumnName": "email",
      "ColumnType": CT_STRING,
      "IndexType":  IT_BPLUSTREE,
      "Primary":    int(1)
    },
    Row{
      "ColumnName": "age",
      "ColumnType": CT_INTEGER,
      "IndexType":  IT_BPLUSTREE,
      "Primary":    int(0)
    },
  }
```

```plaintext
// List the column info for the contacts table (in the testdb database owned by test.eth).
curl -X POST -d '{ "requesttype":"DescribeTable" }'  "http://localhost:8501/testdb.test.eth/contacts"

// Output: {"data":[{"ColumnName":"email","ColumnType":"STRING","IndexType":"BPLUS","Primary":1},{"ColumnName":"name","ColumnType":"STRING","IndexType":"HASH","Primary":0},{"ColumnName":"age","ColumnType":"INTEGER","IndexType":"BPLUS","Primary":0}]}
```
Shows the Columns of an existing Table.

## List Tables

```javascript
//List Tables
conn.listTables(function (err, tblList) {
    if (err) {
      throw err;
    }
    console.log(tblList);
});

// Output: {"data":[{"table":"contacts"}],"matchedrowcount":1}
```

```go
//List Tables
//func (db *SWARMDBDatabase) ListTables() (tables []Row, err error)

tblList, err := db.ListTables()
if err != nil {
    fmt.Printf(err.Error())
}

//output:
type Row map[string]interface{}
list = []Row{
  Row{"table": "contacts"},
}
```

```plaintext
// List the tables belonging to the testdb database (owned by test.eth)
curl -X POST -d '{ "requesttype":"ListTables" }' "http://localhost:8501/testdb.test.eth/"

// Output: {"data":[{"table":"contacts"}],"matchedrowcount":1}
```

# Write

Writing a row (or rows) to SWARMDB tables may be done via the PUT call or running a SQL INSERT/UPDATE query.

Note: The PUT statement overwrites the existing row with the exact value specified if the primary key value already exists

Either CreateTable or OpenTable should be called prior to Write calls.

## Put
```javascript
//Add / Update a single row 
var rowToAdd = '[ { "name": "Bertie Basset", "age": 7, "email": "bertie@gmail.com" } ]';
conn.put( rowToAdd,  function (err, rowAdded) {
    if (err) {
      throw err;
    }
    console.log(rowAdded);
});
// Output: {"affectedrowcount":1}

//Add / Update multiple rows 
var rowsToAdd = '[ { "name": "Bertie Basset", "age": 7, "email": "bertie@gmail.com" }, {"email": "keisha@gmail.com", "age": 3, "name": "Keisha Shepherd"} ]'
conn.put( rowsToAdd,  function (err, rowsAdded) {
    if (err) {
      throw err;
    }
    console.log(rowsAdded);
});
// Output: {"affectedrowcount":2}
```

```go

//Add a single Row
//func (tbl *SWARMDBTable) Put(row interface{}) error
rowToAdd := Row{"email": "bertie@gmail.com", "age": 7, "name": "Bertie Basset"}
err = tbl.Put(rowToAdd)
if err != nil {
    fmt.Printf(err.Error())
}

rowsToAdd :=
  []swarmdblib.Row{
    swarmdblib.Row{"email": "bertie@gmail.com", "age": 7, "name": "Bertie Basset"},
    swarmdblib.Row{"email": "keisha@gmail.com", "age": 3, "name": "Keisha Shepherd"},
  }

//Add multiple Rows
err = tbl.Put(rowsToAdd)
if err != nil {
    fmt.Printf(err.Error())
}
```

```plaintext
// Insert into or Update the contacts table (in the testdb database, owned by test.eth) with the row {"email":"bertie@gmail.com", "name":"Bertie Basset", "age":7} 
curl -X POST -d '{ "requesttype":"Put",  "row":{"email":"bertie@gmail.com", "name":"Bertie Basset", "age":7} }' "http://localhost:8501/testdb.test.eth/contacts"

// Output: {"affectedrowcount":1}
```

Put calls allow for writing rows.

<aside class="notice">The "rows" input consists of an array rows, where a row is a defined as a json object containing column/value pairs.  Each row must at least include a column/value pair for the primary key.</aside>

## Insert
```javascript
//Insert a row via SQL
var sqlInsert = "INSERT INTO contacts(email, name, age) VALUES('warren@gmail.com','Warren',6);";
conn.query(sqlInsert, function (err, rowsInserted) {
    if (err) {
      throw err;
    }
    console.log(rowsInserted);
});

// Output: {"affectedrowcount":1}
```

```go
//Insert a row
//func (tbl *SWARMDBTable) Query(query string) (response []Row, err error)

sqlInsert := "INSERT INTO contacts(email, name, age) VALUES('warren@gmail.com','Warren',6);";
_, err = tbl.Query(sqlInsert)
if err != nil {
    fmt.Printf(err.Error())
}
```

```plaintext
// Query (Insert) a row into the contacts table (in the testdb database, owned by test.eth)
curl -X POST -d '{ "requesttype":"Query", "query":"INSERT INTO contacts(email, name, age) VALUES(\"warren@gmail.com\",\"Warren\",6) " }'  "http://localhost:8501/testdb.test.eth/"

// Output: {"affectedrowcount":1}
```

Insert Query calls allow for the insertion of rows by specifying an INSERT query using standard SQL. SWARMDB currently only supports one row insert per call.

## Update
```javascript
//Update a row using SQL
va sqlUpdate = "UPDATE contacts SET age=8 WHERE email='bertie@gmail.com';";
conn.query(sqlUpdate, function (err, rowsUpdated) {
    if (err) {
      throw err;
    }
    console.log(rowsUpdated);
});
// Output: {"affectedrowcount":1}
```

```go
//Update a row using SQL
//func (tbl *SWARMDBTable) Query(query string) (response []Row, err error)

sqlUpdate := "UPDATE contacts SET age=8 WHERE email='bertie@gmail.com';";
_, err = tbl.Query(sqlUpdate)
if err != nil {
    fmt.Printf(err.Error())
}
```

```plaintext
// Query (Update) a row in the contacts table (owned by test.eth owner ENS), setting the age to 8, where the email='bertie@gmail.com'
curl -X POST -d '{ "requesttype":"Query", "query":"UPDATE contacts SET age=8 WHERE email=\"bertie@gmail.com\" " }' "http://localhost:8501/testdb.test.eth/"

// Output: {"affectedrowcount":1}
```
Update Query calls allow for the update on non-primary key using standard SQL.


# Read

Reading a row (or rows) from SWARMDB tables may be done via the GET call or running a SQL Select query.

In Node.js, either createTable or openTable should be called prior to Read calls.

## Get
```javascript
//Read a row with a given primary key

var primaryKey = "bertie@gmail.com";
conn.get(primaryKey, function (err, retrievedRow) {
    if (err) {
      throw err;
    }
    console.log(retrievedRow);
});
// Output: {"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```

```go
//Retrieve a row with a given primary key
//func (tbl *SWARMDBTable) Get(key string) (rows []Row, err error)
primaryKey = "bertie@gmail.com";
retrievedRow, err := tbl.Get(primaryKey)
if err != nil {
   fmt.Printf(err.Error())
}
fmt.Printf("Output: %v\n")

//output:
type Row map[string]interface{}
retrievedRow = 
  []Row{
    Row{"name":"Bertie Basset", "age": 7, "email": "bertie@gmail.com"},
  }
```

```plaintext
// Try to retrieve a row with the primary key value bertie@gmail.com from the contacts table (in the testdb database, owned by test.eth)
curl "http://localhost:8501/testdb.test.eth/contacts/bertie@gmail.com"

// Output: {"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```
Get calls allow for the retrieval of a single row by specifying the value of a rows primary key.

## Select
```javascript
//Retrieve a Row using SQL
var sqlSelect = "SELECT email, name, age FROM contacts WHERE email = 'bertie@gmail.com'";
conn.query(sqlSelect, function (err, retrievedRows) {
    if (err) {
      throw err;
    }
    console.log(retrievedRows);
});
// Output: {"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```

```go
//Retrieve a Row using SQL
//func (tbl *SWARMDBTable) Query(query string) (response []Row, err error)

sqlSelect := "SELECT email, name, age FROM contacts WHERE email = 'bertie@gmail.com'";
retrievedRows, err = tbl.Query(sqlSelect)

//output:
type Row map[string]interface{}
retrievedRows = []Row {
  Row{"email": "bertie@gmail.com", "name":"Bertie Basset", "age": 7},
}
```

```plaintext
// Query the testdb database (owned by test.eth owner ENS)
curl -X POST -d '{ "requesttype":"Query", "query":"SELECT email, name, age FROM contacts WHERE email=\"bertie@gmail.com\" " }'  "http://localhost:8501/testdb.test.eth/"

// Output: {"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```

Lists the existing Tables in the specified Database.

# Remove

Removing a Row, Table or Database is not recoverable. If a Database is removed, all the Tables associated with it are removed. Likewise, if a Table is removed, all rows in the Table are also removed. 

## Delete

<!--
```javascript
//Delete Row from a Table
```

```go
//Delete Row from a Table
//func (tbl *SWARMDBTable) Delete(key string) error

primaryKey := "bertie@gmail.com"
err := tbl.Delete(primaryKey)
if err != nil {
    fmt.Printf(err.Error())
}

```

```shell
//Delete Row from a Table
curl -X POST -d '{ "requesttype":"Delete" }'  "http://localhost:8501/testdb.test.eth/"

// Output: {"data":[{"database":"testdb"}],"matchedrowcount":1}
```
-->

NOT SUPPORTED IN PROOF OF CONCEPT RELEASE: Removes a row in a Table by primary key value.
## DropTable

```javascript
//Drop Table
```

```go
//func (db *SWARMDBDatabase) DropTable(name string) (err error)

err := db.DropTable("contacts")
if err != nil {
    fmt.Printf(err.Error())
}

```

```shell
//Drop contacts table from testdb (which is owned by test.eth) 
curl -X POST -d '{ "requesttype":"DropTable"  }'  "http://localhost:8501/testdb.test.eth/contacts"

// Output: {"affectedrowcount":1}
```

Removes a Table.

## DropDatabase

```javascript
//Drop Database
```

```go
//Drop Database
//func (dbc *SWARMDBConnection) DropDatabase(name string) (err error) 

err := dbc.DropDatabase("contacts")
if err != nil {
    fmt.Printf(err.Error())
}

```

```shell
// Drop testdb database owned by test.eth.
curl -X POST -d '{ "requesttype":"DropDatabase" }'  "http://localhost:8501/testdb.test.eth/"

// Output: {"affectedrowcount":1}
```

Removes a Database.
