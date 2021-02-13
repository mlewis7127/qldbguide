---
title: "Current Limitations"
date: 2020-02-21T18:43:05Z
lastmod: 2020-07-11
weight: 9
draft: false
---

## Unique Key

The unique identifier of a document is a QLDB generated identifier. All QLDB-assigned IDs are universally unique identifiers (UUID) that are each represented in a Base62-encoded string. This id is returned as the `documentId` when a new document is created, and can be viewed as the `id` attribute in the `metadata` section when querying the committed view. 

QLDB does not support the concept of a primary key such as a partition key in DynamoDB. This means that when using an application generated id which must be unique, a check must be made in the application code, before inserting the document. An example code snippet is shown below:

{{< codeblock "language-javascript" >}}
await qldbDriver.executeLambda(async (txn) => {
    // Check if the record already exist
    const recordsReturned = await checkIdUnique(txn, id);
    if (recordsReturned === 0) {
        // record unique
    } else {
        // record not unique
    }
});

{{< /codeblock  >}}

Luckily the check for the unique identifier and the creation of a new document will be executed with one atomic transaction.

Best practice is to ensure that the QLDB-assigned ID is also added to the application data. An example code snippet for this is shown below:

{{< codeblock "language-javascript" >}}
  await qldbDriver.executeLambda(async (txn) => {
    // Creating record returns unique document Id
    const result = await insertDocument(txn, docToInsert);
    const docIdArray = result.getResultList();
    const docId = docIdArray[0].get('documentId').stringValue();
    // Update document with documentId
    await addDocId(txn, docId, otherId);
    ...
  });

{{< /codeblock  >}}

{{< spacer >}}

## IAM Permission Granularity

Interaction with QLDB can be broken down into two areas:

* Access to the control plane APIs via the QLDB class in the AWS SDK
* Access to the transactional data plane via the QLDB Driver class in one of the officially supported drivers for QLDB

You can use IAM to control access to the control plane APIs. These include APIs to create or delete a ledger, export the ledger, retrieve a block object, retrieve the digest or retrieve a revision data object. 

All interaction to the transactional data plane to execute PartiQL statements against a ledger requires permission to the `qldb:SendCommand` action. This action allows for any PartiQL statement to be executed. This means it allows an application to create, query and modify all documents, as well as view the entire revision history for any document.

{{< codeblock "language-yaml" >}}
  Policies:
    - Version: 2012-10-17
      Statement:
        - Effect: Allow 
          Action:
            - qldb:SendCommand
          Resource: !Sub arn:aws:qldb:${AWS::Region}:${AWS::AccountId}:ledger/${QldbLedger}
          
{{< /codeblock  >}}

Although there are no fine-grained permissions for executing PartiQL statements, a typical serverless application architecture for interacting with QLDB is shown below:

![QLDB Example Architecture](/images/QLDB-Guide-Architecture.png)

Best practice design would include assigning a unique IAM role per individual function, allowing access to be controlled.

{{< spacer >}}

## PartiQL Operations Support

As a result of the strong isolation levels offered by QLDB, there is limited support for PartiQL operations. For example:

  * There is no support for `ORDER BY` to allow for sorting
  * There is no support for `LIMIT` to support pagination

The workaround is to stream out data to another database engine using QLDB streams.

{{< spacer >}}

## KMS

There is no current support for KMS CMKs for server-side encryption, only QLDB-managed KMS keys.

{{< spacer >}}

## Backup and Restore

While you can export ION documents to S3, there is no way to restore them right now.

{{< spacer >}}




  