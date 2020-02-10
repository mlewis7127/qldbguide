---
title: "How QLDB Works"
date: 2019-12-29T23:20:38Z
lastmod: 2020-01-03
weight: 20
draft: false
---

# How QLDB Works

A basic overview of how QLDB works is shown in the diagram below:

![How QLDB Works](/images/How-QLDB-Works.png)


Applications interact with QLDB using a SQL-compatible language called PartiQL.

All changes to data in QLDB is carried within a database transaction. These transactions are constructed locally and the interim transaction state is stored ephemerally with no locks acquired on the resources. This state contains all reads as well as writes. No data is written durably until it is committed. QLDB uses Optimistic Concurrency Control (OCC). This operates on the principle that multiple transactions can frequently complete without intefering with each other.

QLDB is implemented with a journal-first architecture. No record can be updated without going through the journal first. When an application attempts to commit a transaction, the proposed changes are submitted to the journal, along with information about what data has been read. QLDB will reject the commit if another transaction has intefered with the data. This allows the application to retry. If the transaction is accepted, it will be committed to the journal. This means that the journal only contains committed transactions, and is not concerned about rollbacks.

QLDB provides queryable views of your data committed to the journal. Once a transaction has been successfully committed, QLDB projects the data into user defined tables. These views are maintained in real time, and are eventually consistent.

There are 3 different ways of querying data held in QLDB:

* *User* - the latest non-deleted revision of your application-defined data
* *Committed* - the latest non-deleted revision of your application-defined data along with the system-generated metadata
* *History* - the entire revision history of your data that returns both the application-defined data and the metadata can be queried using the History function

At any point in time, the entire content of the journal can be exported for a variety of purposes such as analytics, auditing, verification, backup, and exporting to other systems. Within the export are data objects for every block in the journal. These contain transaction metadata along with entries that represent the document revisions that were committed in the transaction and the PartiQL statements that committed them.

QLDB also builds a continuously-updated Merkle tree of the journal blocks. At any point in time, a digest can be requested, which represents the root of the Merkle tree. This uniquely represents the entire history of document revisions at this time. These digests can also be published wider, to allow third parties to carry out the same integrity proofs.