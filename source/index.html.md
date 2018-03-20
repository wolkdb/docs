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

**Since this is just a proof-of-concept, going from one version to the next is currently not backwards compatible. Thus, when updating from one version of SwarmDB to the next, you will need to clear out the old database from the previous version.**  

This is a pre-alpha project under active development with Ethereum Swarm. To install the current version of SWARMDB, please see [README](https://github.com/wolkdb/swarm.wolk.com/blob/master/README.md). 

If you require further assistance, feel free to shoot us an email: [services@wolk.com](mailto:services@wolk.com)

# Install

To use SWARMDB API, you need to install the corresponding library.

**If you are using Node Shell in SwarmDB docker image, swarmdb.js package is already installed at `/root/swarmdb.js`.
You can skip this step and move onto next section.**

```javascript
//Getting swarmdb package
$ npm install swarmdb.js --save
```

```go
//Getting swarmdblib package
go get github.com/wolkdb/swarmdblib
```

```plaintext
//Verifying that wolkdb is running
$ ps aux | grep wolkdb | grep -vE 'wolkdb-start|grep'
$ curl https://www.google.com | wc

//Restart wolkdb (if it's not running)
$ sudo kill -9 $(ps aux | grep wolkdb | grep -v grep | awk '{print$2}')
$ /usr/local/swarmdb/bin/wolkdb &
```

# Configuration

Upon successfully installation of your SWARMDB node, an address and its associated private key are randomly generated and stored in the [SWARMDB configuration file](https://github.com/wolkdb/swarm.wolk.com/wiki/9.-SWARMDB-Server-Configuration,--Authentication-and-Voting#configuration-file). 
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


```javascript
//Setting private key as environment variable (in terminal)
$ export PRIVATE_KEY=PRIVATE_KEY_IN_YOUR_CONFIG_FILE
//example: export PRIVATE_KEY=1111222233334444555566667777888899990000aaaabbbbccccddddeeeeffff

//Note: private key and other settings can be found at '/usr/local/swarmdb/etc/swarmdb.conf'
```

For Node.js API, you need to set the private key associated with this user as an environment variable in the terminal. 


# Connect
```javascript
//starting nodejs console
$ node 

//If you are manually installing swarmdb.js
var swarmdbJSPath = "swarmdb.js";
//Else if you are using Node Shell in SwarmDB docker image
var swarmdbJSPath = "/root/swarmdb.js";

var swarmdbAPI = require(swarmdbJSPath);

//Connecting to SWARMDB node
var SWARMDB_HOST = "localhost";
var SWARMDB_PORT = 2001;
var conn = swarmdbAPI.createConnection({
  host: SWARMDB_HOST,
  port: SWARMDB_PORT
});

// Output: No return value

```

```go

//Using swarmdblib package
import "github.com/wolkdb/swarmdblib"

//Connecting to SWARMDB node
//func NewSWARMDBConnection(ip string, port int, owner string, privateKey string) (dbc SWARMDBConnection, err error)

host := "localhost"             //your SWARMDB node IP
port := int(2001)               //your SWARMDB node Port number
owner := "test.ens"             //your SWARMDB node owner address
privateKey := "YOURPRIVATEKEY"  //your SWARMDB node private key
conn, err := swarmdblib.NewSWARMDBConnection(host, port, owner, privateKey)
if err != nil {
    fmt.Printf(err.Error())
}

//Note: private key and other settings can be found at '/usr/local/swarmdb/etc/swarmdb.conf'

```

```plaintext
For more information on this see https://github.com/wolkdb/swarm.wolk.com/wiki/9.-SWARMDB-Server-Configuration,--Authentication-and-Voting#http-authentication
```

Open a connection by specifying the `host` and `port` of the SWARMDB node.  Specific details may be found in the [SWARMDB configuration file](https://github.com/wolkdb/swarm.wolk.com/wiki/8.-SWARMDB-Server-Configuration,--Authentication-and-Voting#configuration-file).
Owner should be a valid ENS domain.


# Database

## Create Database

```javascript
//Creating Database
var owner = "test.eth";
var databaseName= "testdb";
var encrypted = 1;
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

For encryption status, `1` means "encrypted" while `0` means "not-encrypted".

## Open Database 

```javascript
//Open Database
var owner = "test.eth";
var databaseName= "testdb";
conn.openDatabase(owner, databaseName);

// Output: No return value
```

```go
//Open Database
//func (dbc *SWARMDBConnection) OpenDatabase(name string) (db *SWARMDBDatabase, err error)

databaseName := "testdb"
db, err  := conn.OpenDatabase(databaseName)
if err != nil {
    fmt.Printf(err.Error())
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
conn.listDatabases(function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output:
{"data":[{"database":"testdb"}],"matchedrowcount":1}
```

```go
//List Databases
//func (dbc *SWARMDBConnection) ListDatabases() (databases []Row, err error)

dblist, err := conn.ListDatabases()
if err != nil {
    fmt.Printf(err.Error())
}

// Output:
type swarmdblib.Row map[string]interface{}
dblist = []swarmdblib.Row{
  swarmdblib.Row{"database": "testdb"},
}
```

```plaintext
// List the databases owned by test.eth.
curl -X POST -d '{ "requesttype":"ListDatabases" }'  "http://localhost:8501/test.eth/"

// Output:
{"data":[{"database":"testdb"}],"matchedrowcount":1}
```

Lists the Databases associated with a Connection.

# Table

For Node.js API, either createDatabase or openDatabase should be called prior to table method calls.

## Create Table

```javascript
//Create Table
var tableName = "contacts";
var columns = [
  { "columnname": "email", "columntype": "STRING", "indextype": "BPLUS", "primary": 1 },
  { "columnname": "name", "columntype": "STRING", "indextype": "HASH", "primary": 0 },
  { "columnname": "age", "columntype": "INTEGER", "indextype": "BPLUS", "primary": 0 }
];
conn.createTable(tableName, columns, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output: {"affectedrowcount":1}
```

```go
//Create Table
//func (db *SWARMDBDatabase) CreateTable(name string, columns []Column) (tbl *SWARMDBTable, err error)

columns :=
  []swarmdblib.Column{
    swarmdblib.Column{
      ColumnName: "email",
      ColumnType: swarmdblib.CT_STRING,
      IndexType: swarmdblib.IT_BPLUSTREE,
      Primary: 1,
    },
    swarmdblib.Column{
      ColumnName: "age",
      ColumnType: swarmdblib.CT_INTEGER,
      IndexType: swarmdblib.IT_BPLUSTREE,
      Primary: 0,
    },
    swarmdblib.Column{
      ColumnName: "name",
      ColumnType: swarmdblib.CT_STRING,
      IndexType: swarmdblib.IT_BPLUSTREE,
      Primary: 0,
    },
}
tableName := "contacts"
tbl, err := db.CreateTable(tableName, columns)
if err != nil {
    fmt.Printf(err.Error())
}
```

```plaintext
/* Create a table named contacts, which belongs to the database named testdb, which is owned by test.eth.

For example, creating a 'contacts' table which has 3 columns:
 1) 'email' is of type  STRING and uses the BPLUS index (Primary key)
 2) 'name'  is of type  STRING and uses the  HASH index (Non-primary key)
 3)  'age'  is of type INTEGER and uses the BPLUS index (Non-primary key) */

curl -X POST -d '{ "requesttype":"CreateTable", "table":"contacts", "columns":[{ "columnName":"email", "ColumnType":"STRING", "Primary":1, "IndexType":"BPLUS" }, { "columnname":"name", "columntype":"STRING", "IndexType":"HASH" }, { "columnname":"age", "columntype":"INTEGER", "IndexType":"BPLUS" } ] }' "http://localhost:8501/testdb.test.eth/"

// Output: {"affectedrowcount":1}
```

Create a table by specifying table name and column details. Columns must consist of the following parameters:

* **indextype**: enter the desired indextype, defined [HERE](https://github.com/wolkdb/swarm.wolk.com/wiki/8.-SWARMDB-Types#table-index-types)
* **columnname**: enter the desired column name (no spaces allowed)
* **columntype**: enter the desired columntype, defined [HERE](https://github.com/wolkdb/swarm.wolk.com/wiki/8.-SWARMDB-Types#table-column-types)
* **primary**: enter `1` if the column is the primary key and `0` for any other column.

## Open Table 

```javascript
//Open Table
var tableName = "contacts";
conn.openTable(tableName);

// Output: No return value
```

```go
//Open Table for manipulating
//func (db *SWARMDBDatabase) OpenTable(name string) (tbl *SWARMDBTable, err error)

tableName := "contacts"
tbl, err := db.OpenTable(tableName)
if err != nil {
    fmt.Printf(err.Error())
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
var tableName = "contacts";
conn.describeTable(tableName, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output:
{"data":[{"ColumnName":"email","ColumnType":"STRING","IndexType":"BPLUS","Primary":1},{"ColumnName":"name","ColumnType":"STRING","IndexType":"HASH","Primary":0},{"ColumnName":"age","ColumnType":"INTEGER","IndexType":"BPLUS","Primary":0}]}
```

```go
//Describe Table
//func (tbl *SWARMDBTable) DescribeTable() (description []Row, err error) 
description, err := tbl.DescribeTable()
if err != nil {
    fmt.Printf(err.Error())
}

// Output:
type swarmdblib.Row map[string]interface{}
description = 
  []swarmdblib.Row{
    swarmdblib.Row{
      "ColumnName": "email",
      "ColumnType": swarmdblib.CT_STRING,
      "IndexType":  swarmdblib.IT_BPLUSTREE,
      "Primary":    int(1)
    },
    swarmdblib.Row{
      "ColumnName": "age",
      "ColumnType": swarmdblib.CT_INTEGER,
      "IndexType":  swarmdblib.IT_BPLUSTREE,
      "Primary":    int(0)
    },
  }
```

```plaintext
// List the column info for the contacts table (in the testdb database owned by test.eth).
curl -X POST -d '{ "requesttype":"DescribeTable" }'  "http://localhost:8501/testdb.test.eth/contacts"

// Output:
{"data":[{"ColumnName":"email","ColumnType":"STRING","IndexType":"BPLUS","Primary":1},{"ColumnName":"name","ColumnType":"STRING","IndexType":"HASH","Primary":0},{"ColumnName":"age","ColumnType":"INTEGER","IndexType":"BPLUS","Primary":0}]}
```
Shows the Columns of an existing Table.

## List Tables

```javascript
//List Tables
conn.listTables(function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output:
{"data":[{"table":"contacts"}],"matchedrowcount":1}
```

```go
//List Tables
//func (db *SWARMDBDatabase) ListTables() (tables []Row, err error)

tblList, err := db.ListTables()
if err != nil {
    fmt.Printf(err.Error())
}

// Output:
type swarmdblib.Row map[string]interface{}
tblList = []swarmdblib.Row{
  swarmdblib.Row{"table": "contacts"},
}
```

```plaintext
// List the tables belonging to the testdb database (owned by test.eth)
curl -X POST -d '{ "requesttype":"ListTables" }' "http://localhost:8501/testdb.test.eth/"

// Output:
{"data":[{"table":"contacts"}],"matchedrowcount":1}
```
Lists all Tables associated with a given Database. Requires an opened connection to a database.

# Write

Writing a row (or rows) to SWARMDB tables may be done via the PUT call or running a SQL INSERT/UPDATE query.

Note: The PUT statement overwrites the existing row with the exact value specified if the primary key value already exists

Either CreateTable or OpenTable should be called prior to Write calls.

## Put
```javascript
//Add row(s) to table 

//Adding single row 
var rowToAdd = [ { "name": "Bertie Basset", "age": 8, "email": "bertie@gmail.com" } ];
conn.put( rowToAdd,  function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output: {"affectedrowcount":1}


//Adding multiple rows 
var rowsToAdd = [ { "name": "Bertie Basset", "age": 7, "email": "bertie@gmail.com" }, {"email": "keisha@gmail.com", "age": 3, "name": "Keisha Shepherd"} ];
conn.put( rowsToAdd,  function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output: {"affectedrowcount":2}
```

```go
//Add row(s) to table
//func (tbl *SWARMDBTable) Put(row interface{}) error

//Adding single Row
rowToAdd := swarmdblib.Row{"email": "bertie@gmail.com", "age": 8, "name": "Bertie Basset"}
err = tbl.Put(rowToAdd)
if err != nil {
    fmt.Printf(err.Error())
}

//Adding multiple Rows
rowsToAdd :=
  []swarmdblib.Row{
        swarmdblib.Row{"email": "bertie@gmail.com", "age": 7, "name": "Bertie Basset"},
        swarmdblib.Row{"email": "keisha@gmail.com", "age": 3, "name": "Keisha Shepherd"},
  }
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
//Insert a row using Query
var sqlInsert = "INSERT INTO contacts(email, name, age) VALUES('warren@gmail.com','Warren',6)";
conn.query(sqlInsert, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output: {"affectedrowcount":1}
```

```go
//Insert a row using Query
//func (tbl *SWARMDBTable) Query(query string) (response []Row, err error)

sqlInsert := "INSERT INTO contacts(email, name, age) VALUES('warren@gmail.com','Warren',6)";
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
//Update a row using Query
var sqlUpdate = "UPDATE contacts SET age=8 WHERE email='bertie@gmail.com'";
conn.query(sqlUpdate, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output: {"affectedrowcount":1}
```

```go
//Update a row using Query
//func (tbl *SWARMDBTable) Query(query string) (response []Row, err error)

sqlUpdate := "UPDATE contacts SET age=8 WHERE email='bertie@gmail.com'";
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

Reading row(s) from SWARMDB tables may be done via the GET call or running a SQL Select query.

In Node.js, either createTable or openTable should be called prior to Read calls.

## Get
```javascript
//Retrieve a row using primary key

var primaryKey = "bertie@gmail.com";
conn.get(primaryKey, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output:
{"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```

```go
//Retrieve a row using primary key
//func (tbl *SWARMDBTable) Get(key string) (rows []Row, err error)

primaryKey = "bertie@gmail.com";
retrievedRow, err := tbl.Get(primaryKey)
if err != nil {
   fmt.Printf(err.Error())
}

// Output:
type swarmdblib.Row map[string]interface{}
retrievedRow = 
  []swarmdblib.Row{
    swarmdblib.Row{"name":"Bertie Basset", "age": 7, "email": "bertie@gmail.com"},
  }
```

```plaintext
// Try to retrieve a row using the primary key value 'bertie@gmail.com' from the contacts table (in the testdb database, owned by test.eth)
curl "http://localhost:8501/testdb.test.eth/contacts/bertie@gmail.com"

// Output:
{"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```
Get calls allow for the retrieval of a single row by specifying the value of a rows primary key.

## Select
```javascript
//Retrieve a Row using Query

var sqlSelect = "SELECT email, name, age FROM contacts WHERE email = 'bertie@gmail.com'";
conn.query(sqlSelect, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output:
{"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
```

```go
//Retrieve a Row using Query
//func (tbl *SWARMDBTable) Query(query string) (response []Row, err error)

sqlSelect := "SELECT email, name, age FROM contacts WHERE email = 'bertie@gmail.com'";
retrievedRows, err = tbl.Query(sqlSelect)

// Output:
type swarmdblib.Row map[string]interface{}
retrievedRows = []swarmdblib.Row {
  swarmdblib.Row{"email": "bertie@gmail.com", "name":"Bertie Basset", "age": 7},
}
```

```plaintext
// Retrieve a Row using Query from testdb database (owned by test.eth owner ENS)
curl -X POST -d '{ "requesttype":"Query", "query":"SELECT email, name, age FROM contacts WHERE email=\"bertie@gmail.com\" " }'  "http://localhost:8501/testdb.test.eth/"

// Output:
{"data":[{"email":"bertie@gmail.com","name":"Bertie Basset","age":7}]}
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

// Output:
{"data":[{"database":"testdb"}],"matchedrowcount":1}
```
-->

NOT SUPPORTED IN PROOF OF CONCEPT RELEASE: Removes a row in a Table by primary key value.
## DropTable

```javascript
//Drop Table
var tableName = "contacts";
conn.dropTable(tableName, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output: {"affectedrowcount":1}
```

```go
//Drop Table
//func (db *SWARMDBDatabase) DropTable(name string) (err error)

tableName := "contacts"
err := db.DropTable(tableName)
if err != nil {
    fmt.Printf(err.Error())
}

```

```plaintext
//Drop contacts table from testdb (which is owned by test.eth) 
curl -X POST -d '{ "requesttype":"DropTable"  }'  "http://localhost:8501/testdb.test.eth/contacts"

// Output: {"affectedrowcount":1}
```

Removes a Table.

## DropDatabase

```javascript
//Drop Database
var owner = "test.eth";
var databaseName= "testdb";
conn.dropDatabase(owner, databaseName, function (err, result) {
  if (err) {
    throw err;
  }
  console.log(result);
});

// Output: {"affectedrowcount":1}
```

```go
//Drop Database
//func (dbc *SWARMDBConnection) DropDatabase(name string) (err error) 

databaseName := "testdb"
err := conn.DropDatabase(databaseName)
if err != nil {
    fmt.Printf(err.Error())
}

```

```plaintext
// Drop testdb database owned by test.eth.
curl -X POST -d '{ "requesttype":"DropDatabase" }'  "http://localhost:8501/testdb.test.eth/"

// Output: {"affectedrowcount":1}
```

Removes a Database.
