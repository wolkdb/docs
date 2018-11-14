---
title: WOLK API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript: Node.js

includes:
  - errors

search: true
---

# Introduction
wolkjs is a client library for the Wolk API, implemented in JavaScript. It's available on npm as a node module.

To use wolkjs, for your desired database type (NoSQL or SQL), you MUST either

* Install WolkDB via source code or the docker image for your db type
* Register your address on the WolkDB Public Gateway. (Available in Dec. 2018)

Instructions for setting up and testing your own node can be found HERE.

Please note that the library is still under development and may not be stable.

If you require further assistance, feel free to shoot us an email: [services@wolk.com](mailto:services@wolk.com)

# Install
To use WOLK API, you need to install the corresponding library.

```javascript
$ npm install wolkjs
```

# Connect to Plasma Node
Open a connection by specifying the host and port of the Plasma node.

```javascript
var wolkjsAPI = require("wolkjs");

var PLASMAHOST = "plasma.wolk.com";
var PLASMAPORT = "80";
var PLASMAADDR = "http://" + PLASMAHOST + ":" + PLASMAPORT;

var plasmanode = new wolkjsAPI(PLASMAADDR);
```

# Plasma Operations

## Retrieve Plasma Tokens and Token Balances
Retrieve plasma tokens and token balance by specifying address and block number

```javascript
var registered_address = "0xAA96B58F7ee51d2445fE596C3535874F5E206B14";
var blockNumber = "latest";
var plasmaBalance = plasmanode.getPlasmaBalance(registered_address, blockNumber);
// plasmaBalance result:
{ account:
   { tokens: [ '0x1962836485220a62' ],
     denomination: '0x2386f26fc10000',
     balance: '0x2386f26fc10000' } 
}
```

## Retrieve BlockchainID
Retrieve blockchainID by specifying tokenID and block number

```javascript
var tokenId = plasmaBalance.account.tokens[0];
var tokenObject = plasmanode.getPlasmaToken(tokenId, "latest");
// tokenObject result
{ token:
   { denomination: '0x2386f26fc10000',
     prevBlock: '0x3',
     owner: '0xaa96b58f7ee51d2445fe596c3535874f5e206b14',
     balance: '0x2386f26fc10000',
     allowance: '0x0',
     spent: '0x0' },
  tokenInfo:
   { depositIndex: '0x3c',
     denomination: '0x2386f26fc10000',
     depositor: '0xaa96b58f7ee51d2445fe596c3535874f5e206b14',
     tokenID: '0x1962836485220a62' } 
}

var blockchainID = plasmanode.getBlockchainID(tokenId, tokenObject.tokenInfo.depositIndex);
// blockchainID result: '0xcd778162be0d4f2'
```

## Send Initial Anchor Transaction
To register your blockchain with the plasma node by sending the initial anchor transaction

```javascript
// For all initial Anchor Transactions, use 0x03c85f1da84d9c6313e0c34bcb5ace945a9b12105988895252b88ce5b769f82b
var genesisBlockHash = "0x03c85f1da84d9c6313e0c34bcb5ace945a9b12105988895252b88ce5b769f82b";

var rlp = require('rlp');
var web3 = require('web3');
web3Obj = new web3();
var atxExtra = [ [ registered_address ], [] ];
var anchorTxJson = { "blockchainID": blockchainID, "blocknumber": "0x0", "blockhash": genesisBlockHash, "extra": atxExtra, "sig": '' };
var anchorTx = [
  anchorTxJson.blockchainID,
  anchorTxJson.blocknumber,
  anchorTxJson.blockhash,
  anchorTxJson.extra,
  '',
];
var encodedAnchorTx = rlp.encode(anchorTx);
var encodedAnchorTxAsHex = encodedAnchorTx.toString('hex');
var shortHash = web3.sha3(encodedAnchorTxAsHex, {encoding:'hex'});

// Example of signing with Metamask
alert("Check Metamask to Approve Signing and Sending Order");
web3.eth.sign(registered_address, shortHash,
  function(err, sig) {
    if(!err) {
      console.log("Sending AnchorTX")
      anchorTx.sig = sig;
      plasmanode.sendAnchorTransaction(anchorTx)
    }
  }
}
```

# Configure Wolk Node
For the next steps you will need to have a running WolkDB node
 * If you set up your own WolkDB node, make sure to set the blockchainId to the blockchainID retrieved via the steps described above
 * If you registered your address directly with the WolkDB Public Gateway, you should have received the corresponding blockchainID for the selected database type.  Please contact services@wolk.com if you did not.  Those using the public gateway MUST use the blockchainID returned for their requests to work.
 
 Steps for downloading and launching docker can be found at: ___

# Connect to Wolk Node
Open a connection by specifying configuration options like host and port

```javascript
var HOST = "localhost"
var PORT = "24000"
var WOLKADDR = "http://" + HOST + ":" + PORT

var wolknode = new wolkjsAPI(WOLKADDR)
```

# NoSQL Operations

## SetKey
Set a key value pair

```javascript
wolknode.setKey("blockchainID", "key", "val")

// setKey result:
{ txhash:
   '2471d3ed40ba6ddb3ff0ea12cdae2f378d6ef146025d5a2b3ef02325af9d0677' }
```

## GetKey
Get the value corresponding to a given key

```javascript
var value = wolknode.getKey("blockchainID", "key")

// getKey result:
{ proof: '{"token":"07855b46a623a8ec","proofBits":"0","proof":[]}',
  value: 'valueA' }
```

# SQL Operations

## Create database
Create a database by specifying your blockchainID, the requesttype ("CreateDatabase"), an owner (will be specific to your blockchain) and database name.

```javascript
var createDatabaseCommand = { "requesttype":"CreateDatabase", "owner":"wolkdb.eth", "database":"testdb" }
wolknode.exec(blockchainID, createDatabaseCommand)
// result:
{ txhash: "4450a4f4fde8f0b49d83d2ec3b1185cdc66b287748e546afa55571d887021cc8" }
```

## List databases
List existing databases by specifying your blockchainID, the requesttype ("ListDatabases"), an owner (will be specific to your blockchain) and (optionally) the blocknumber.  If blocknumber is not included, the request defaults to returning data that corresponds with the latest block minted.

```javascript
var listDatabasesCommand = { "requesttype":"ListDatabases", "owner":"wolkdb.eth", "blocknumber":1 })
wolknode.exec(blockchainID, listDatabasesCommand)
// result:
{
  result: {
    data: [{
        database: "testdb"
    }],
    matchedrowcount: 1
  }
}
```

## Create table
Create a table by specifying your blockchainID and a CREATE TABLE SQL statement.

Note: current supported types for columns are `string`, `integer`, `float` and a table must have a primary key.

```javascript
var createTableSQL = "CREATE TABLE demoitems(ID string primary key, Name string, Description string, Price float, Currency string, Hash string)";
var createTableCommand = { "requesttype":"CreateTable", "owner":"wolkdb.eth", "database":"testdb", "query": createTableSQL }
sql.exec(blockchainID, createTableCommand)
// result:
{
  txhash: "98827bbd28f1cf65f91dbc0abdb7e50e6bb4a9d4fb69283b1ca5ff828c686a9b"
}
```

## Describe Table
To see the table definition of a table, specify your blockchainID along with the owner by specifying table name.

```javascript
var describeTableCommand = { "requesttype":"DescribeTable", "owner":"wolkdb.eth", "database":"testdb", "table":"demoitems" }
sql.exec(blockchainID, describeTableCommand)
```

## List Tables
To list the tables that belong to a given database, specify your blockchainID, owner, database and (optionally) blocknumber

```javascript
var listTablesCommand = { "requesttype":"ListTables", "owner":"wolkdb.eth", "database":"testdb", "blocknumber":3 }
sql.exec(blockchainID, listTablesCommand)

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

## INSERT Query

```javascript
var sql1 = "INSERT INTO demoitems(ID, Name, Description, Price, Currency, Hash) VALUES('ID1','Item1','Hot Item #1',0.11,'ethereum', 'Qmd286K6pohQcTKYqnS1YhWrCiS4gz7Xi34sdwMe9USZ7u')";
var sql2 = "INSERT INTO demoitems(ID, Name, Description, Price, Currency, Hash) VALUES('ID2','Item2','Hot Item #2',0.09,'ethereum', 'Qme1gTwdtFaUMoqwTNWX5SdYmNfiAj4N88qujbxmsX5z6u')";
var queryCommand = { "requesttype":"Query", "owner":"wolkdb.eth", "database":"testdb", "query": sql1 };
sql.exec(blockchainID, queryCommand);

var queryCommand = { "requesttype":"Query", "owner":"wolkdb.eth", "database":"testdb", "query": sql2 }
sql.exec(blockchainID, queryCommand)
```

## SELECT Query

```javascript
var sql3 = "SELECT * FROM demoitems"
var sql4 = "SELECT * FROM demoitems WHERE price > 0.10"
var queryCommand = { "requesttype":"Query", "owner":"wolkdb.eth", "database":"testdb", "query": sql3 }
var rows = sql.exec(blockchainID, queryCommand)
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

## UPDATE Query

```javascript
var sql5 = "UPDATE demoitems SET Price = 0.13 WHERE ID='ID2'"
var queryCommand = { "requesttype":"Query", "owner":"wolkdb.eth", "database":"testdb", "query": sql5 }
sql.exec(blockchainID, queryCommand)
```


## DELETE Query

```javascript
var sql6 = "DELETE FROM demoitems WHERE ID='ID1'"
var queryCommand = { "requesttype":"Query", "owner":"wolkdb.eth", "database":"testdb", "query": sql6 }
sql.exec(blockchainID, queryCommand)
```

## Drop table

Drop a table by specifying table name.

```javascript
var dropTableCommand = { "requesttype":"DropTable", "owner":"wolkdb.eth", "database":"testdb", "table":"demoitems" }
sql.exec(blockchainID, dropTableCommand)
```

## Drop database

Drop a database by specifying owner and database name.

```javascript
var dropDatabaseCommand = { "requesttype":"DropDatabase", "owner":"wolkdb.eth", "database":"testdb" }
sql.exec(blockchainID, dropTableCommand)
```