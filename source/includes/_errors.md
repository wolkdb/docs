# Errors

[This is undergoing revision now.]

The Wolk API uses the following error codes:

Error Code | Meaning
---------- | -------
400	| Bad JSON supplied: [JSON]
401	| SQL Parsing error: [SQLError in SQL]
402	| Key Not Found: [key]
403	| Table Does Not Exist:  Table: [tableName] Owner: [ownerID]
404	| Column Does Not Exist in table definition: [columnName]
405	| No Primary Key specified in Create Table
406	| Multiple Primary keys specified in Create Table
407	| Invalid ColumnType: [columnType]
408	| Invalid IndexType: [indexType]
409	| Max Allowed Columns exceeded - [Num Columns] supplied, max is [MaxNumColumns]
410	| Max Allowed Record size exceeded - [RecordSize] provided,
411	| Error on Write [Details]
412	| Error on Read [Details]
413	| ENS Error
414	| Invalid Challenge Response
415	| Insufficient Permissions
416	| No Table Owner specified
417	| Table Owner mismatch
418	| Request Invalid
419	| Invalid Signature Length: Must be 65 characters
420	| Invalid Signature: Unable to Retrieve Public Key
421	| User Address not configured on connected node.
422	| Unable to decode response
423	| Error Decoding Signed Message
424	| Error In HTTP Request
425	| Invalid Query Request. Missing Rawquery
426	| Table Name Missing
427	| Mismatched Column Types
428	| Row missing primary key
429	| Row contains unknown column [columnName]
430	| TableOwner Missing
431	| Scans on Column [column] not unsupported due to indextype
432	| WHERE Clause contains invalid column [column name]
433	| GET Request Missing Key
434	| Record with key [keyvalue] already exists. If you wish to modify, please use UPDATE SQL statement or PUT
435	| Invalid Row Data
436	| Unable to converty byte array to Row Object
437	| ColumnType [type] not SUPPORTED. Value [value] rejected
438	| Invalid Request Body
439	| Failure to store [CHUNKTYPE] Chunk
440	| Enable to retrieve [CHUNKTYPE] Chunk
