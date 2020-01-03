---
title: "What is QLDB"
date: 2019-12-08T22:50:19Z
lastmod: 2019-12-29
weight: 10
draft: false
---

### Overview

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


### Why use QLDB

AWS provide a broad portfolio of purpose-built databases. This allows a developer to pick the right tool for the job, rather than using a single product for everything. More details on the positioning of QLDB within this portfolio is provided in the section on [database freedom](../database-freedom/).

The concept of a ledger goes back centuries. The first book on double-entry accounting was published in 1494. This method made it easier to reconcile all changes, and to balance the books. It created a complete audit trail of every single change. The fact that QLDB creates an audit trail of every change that can be easily queried, especially when the records themselves are immutable and can be cryptographically verified, is incredibly powerful. This enables systems of record applications to prove the state of a record at any point in time, and to understand what caused a change. It also makes QLDB a fantastic fit for event sourced applications, and the ideal Domain Driven Design aggregate store.

QLDB itself originated from a need that AWS had internally for an immutable and searchable transaction log to track every single data plane change. Today, you cannot launch an EC2 instance, send a message on Kinesis or scale an autoscaling group without using QLDB technology. 