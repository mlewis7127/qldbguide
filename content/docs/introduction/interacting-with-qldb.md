---
title: "Interacting with QLDB"
date: 2020-02-21T18:32:44Z
lastmod: 2020-07-11
weight: 25
draft: false
---

# Interacting with QLDB

> **Note** QLDB currently provides supported drivers for Java, Python, Nodejs and Go. All examples in this guide use Nodejs

## Environment Setup

A typical setup of dependencies required to interact with QLDB is shown below:

{{< codeblock "language-json" >}}
{
 "dependencies": {
    "amazon-qldb-driver-nodejs": "^1.0.0",
    "aws-sdk": "^2.706.0",
    "aws-xray-sdk-core": "^3.1.0",
    "ion-js": "^4.0.1",
    "jsbi": "^3.1.3"
  },
}
{{< /codeblock  >}}

* The `amazon-qldb-driver-nodejs` module is the official driver provided by AWS
* The `aws-sdk` and `aws-xray-sdk-core` modules are needed to support tracing with X-Ray
* The `ion-js` and `jsbi` modules are needed to easily interact with Amazon ION documents

{{< spacer >}}

## Connect to Ledger

The first step in writing code to interact with QLDB is to create an instance of `QldbDriver`. This can be done as shown below, passing in the name of the Ledger to connect.

{{< codeblock "language-javascript" >}}
{
const { QldbDriver } = require('amazon-qldb-driver-nodejs');
const qldbDriver = new QldbDriver(`LedgerName`);

function getQldbDriver(){
  return qldbDriver;
};
{{< /codeblock  >}}

The constructor for a new `QldbDriver` can take five parameters, although only the first one is mandatory:

* *ledgerName* - the name of the ledger to connect to
* *qldbClientOptions* - an object that contains options for configuring the low level client. More details can be found [here](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/QLDBSession.html#constructor-details).
* *retryLimit* - the number of times the driver will retry a transaction which failed (default 4 times)
* *poolLimit* - the number of sessions the driver can hold in the pool, with the default set to the maximum number of sockets specified in the global agent
* *timeoutMillis* - the time the driver will waot for a session to be available before giving up (default 30 secs)

{{< spacer >}}

## Execute Lambda

The `executeLambda` method on the `QldbDriver` is the primary method to execute a transaction against a QLDB ledger. When this method is invoked, the driver acquires a `Transaction` and hands it to the `TransactionExecutor` that is passed in. Once all execution is complete, the driver attempts to commit the transaction. If there is a failure, then the driver will attempt to retry the entire transaction block, so your code should be idempotent.

{{< codeblock "language-javascript" >}}
await qldbDriver.executeLambda(async txn => {
    const result = await txn.execute('SELECT * FROM Table')
    const values = result.getResultList()
    console.log(JSON.stringify(values, null, 2))
})
{{< /codeblock  >}}

> **Note** In the example above, if multiple statements are executed, they will all either succeed or be rolled back in one atomic transaction

{{< spacer >}}


## CRUD Operations

#### Creating a record

To create a new record, insert a document into a table:

{{< codeblock "language-javascript" >}}
await qldbDriver.executeLambda(async txn => {
    const document = [{'Name': 'name', 'Email': 'name@email.com', 'Telephone': '01234'}];
    const statement = 'INSERT INTO Table ?';
    const result = await txn.execute(statement, document);
    const values = result.getResultList()
    console.log(JSON.stringify(values, null, 2))
})
{{< /codeblock  >}}

The `execute` method returns a Promise that resolves to a `Result` object. This class represents the fully buffered set of results from QLDB in an array. When a new record is inserted, the result object contains the document ID of the record.

{{< codeblock "language-json" >}}
[
  {
    "documentId": "7ISClqWTgkcLNnBlgdtKYa"
  }
]
{{< /codeblock  >}}

If you need to create multiple records within a single transaction, these can all be carried out as part of the `executeLambda` call:

{{< codeblock "language-javascript" >}}
await qldbDriver.executeLambda(async (txn) => {
  let a = await txn.execute("INSERT INTO licence VALUE {'name': 'QLDB', 'type': 'Guide' }");
  let b = await txn.execute("INSERT INTO licence VALUE {'name': 'Centralised', 'type': 'Ledger' }");
  return {a: a, b: b};
})
{{< /codeblock  >}}

When this method is invoked, the driver acquires a `Transaction` which is handed to the `TransactionExecutor` instance passed in via the transaction function. The PartiQL statements executed are not immediately committed. The driver will only attempt to commit the transaction once all the execution is carried out. If there is a failure, then the driver will attempt to retry the entire transaction block.


{{< spacer >}}

#### Reading a record

To read a record, execute a select statement against a table:

{{< codeblock "language-javascript" >}}
const getRecord = async (id) => {
    await qldbDriver.executeLambda(async (txn) => {
        const query = `SELECT * FROM Table WHERE Attribute = ?`;
        const result = await txn.execute(query, id);
        const resultList = result.getResultList();
        ...
    },() => console.log("Retrying due to OCC conflict..."));
};
{{< /codeblock  >}}

The `Result` object returned represents the buffered set of results in an array. The `getResultList()` function returns the list of Ion values returned from the enquiry. You can check the `length` property to determine how many results were returned:

{{< codeblock "language-javascript" >}}
  const resultList = result.getResultList();
  ...
  if (resultList.length === 0) {
      // no record found processing
  } else if (resultList.length === 1) {
      // single record found processing
  } else {
      // multiple records found processing
  }
};
{{< /codeblock  >}}

If you just want to log the document revision returned by QLDB, you can use `JSON.stringify()`:

{{< codeblock "language-javascript" >}}
JSON.stringify(resultList[0], null, 2)};
{{< /codeblock  >}}

Otherwise, you can use simple calls to retrieve specific values out of the record that is returned. The following code will extract the `documentId` returned after inserting a new document.

{{< codeblock "language-javascript" >}}
const docIdArray = result.getResultList()
const docId = docIdArray[0].get("documentId").stringValue();
...
{{< /codeblock  >}}

You can also use multiple attributes with a WHERE predicate clause.

{{< codeblock "language-javascript" >}}
const query = `SELECT * FROM Table WHERE a = ? AND b = ?`;
const result = await txn.execute(query, "valueA", "valueB");
...
};
{{< /codeblock  >}}

In addition, inner joins are also supported.

{{< spacer >}}

#### Updating a record

To update a record, execute an update statement against a table:

{{< codeblock "language-javascript" >}}
await qldbDriver.executeLambda(async (txn) => {
  const statement = `UPDATE Table SET A = ?, B = ?, C = ? WHERE D = ?`;
  const result = await txn.execute(statement, valueA, valueB, valueC, valueD);
  const resultList = result.getResultList();
  ...
}
{{< /codeblock  >}}

In the same way as for creating a record, the result object returned contains the document ID of the record that has been updated.

{{< spacer >}}

#### Deleting a record

{{< spacer >}}