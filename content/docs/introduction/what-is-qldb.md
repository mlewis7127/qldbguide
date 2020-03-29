---
title: "What is QLDB"
date: 2019-12-08T22:50:19Z
lastmod: 2020-02-21
weight: 10
draft: false
---

# What is QLDB

## Overview

Amazon Quantum Ledger Database [(QLDB)](https://aws.amazon.com/qldb/) is:

> a fully managed ledger database that provides a transparent, immutable, and cryptographically
> verifiable transaction log owned by a central trusted authority. Amazon QLDB tracks each and
> every application data change and maintains a complete and verifiable history of changes
> over time

All of these important features are expanded on in [key concepts](../key-concepts/), but as a quick summary:

* **Fully Managed** - QLDB is a fully serverless offering that automatically scales to meet demand
* **Ledger Database** - QLDB is a new class of database built around a centralised ledger
* **Transparent and Immutable** - QLDB has a built-in immutable journal that is append-only with no ability to delete a record. It allows you to easily access the entire change history of any record
* **Cryptographically Verified** - QLDB uses a SHA-256 crytographic hash function with each transaction it commits which then uses hash chaining. This can prove the integrity of any transaction.
* **Central trusted authority** - QLDB is focused on use cases where there is a single owner of the ledger. This is where it differentiates from distributed ledger technologies that require multiple parties to agree a transaction is valid with no single owner.
* **Familiarity** - QLDB supports [PartiQL](https://partiql.org/) which is a SQL-compatible open standard query language. QLDB supports [Amazon ION](http://amzn.github.io/ion-docs/) document data format. This is a superset of JSON that supports a rich type system.

{{< spacer >}}
## Why use QLDB

AWS provide a broad portfolio of purpose-built databases. This allows a developer to pick the right tool for the job, rather than using a one size fits all solution for everything. More details on the positioning of QLDB within this portfolio is provided in the section on [database freedom](../database-freedom/).

The concept of a ledger goes back centuries with the first book on double-entry accounting published in 1494. With this method, transactions are first recorded in the journal in chronological order, which is known as the book of primary entry. They are then transferred into the appropriate account or ledger, which is known as the book of second entry. This creates a complete audit trail of every single change.

QLDB creates an audit trail of every change that can be easily queried, with the committed data being immutable and with the ability for the records to be cryptographically verified. This is becoming ever more important in a digital world, with a focus on interparty trust in a supply chain and regulatory and security compliance.

A classic example is from the UK, with the abolition of the paper tax disc in 2014. Up to this point, a physical token had to be displayed in the windscreen of a vehicle, to prove it had been taxed to be driven on public roads. The requirement to tax the vehicle did not go away. Instead, all enforcement activity is carried out against the electronic record. This means that systems not only have to be accurate, but need the ability to go back and track all changes to a record, when the change was made, and why it was carried out. In addition, there is a need to verify that this data has not been altered in any way after the event.

Solutions to these requirements have previously been carried out with traditional database technologies. However, this requires significant engineering effort to create custom audit trails, which is complex, time-consuming and error prone. This gap is now filled with the release of QLDB to support any solution that requires interparty trust or meet compliance requirements. QLDB is a good fit for any event sourced application, and can be considered an ideal Domain Driven Design aggregate store.

QLDB itself originated from a need that AWS had internally for an immutable and searchable transaction log to track every single data plane change that traditional relational databases did not support well. Today, you cannot launch an EC2 instance, send a message on Kinesis, or scale an autoscaling group without using QLDB technology. It is this internal technology, used for many years within AWS, that forms the backbone of the QLDB product.

{{< spacer >}}
## How QLDB works

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