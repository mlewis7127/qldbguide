---
title: "What is QLDB"
date: 2019-12-08T22:50:19Z
lastmod: 2019-12-29
weight: 10
draft: false
---

Amazon Quantum Ledger Database (QLDB) is described on its [home page](https://aws.amazon.com/qldb/) as:

> a fully managed ledger database that provides a transparent, immutable, and cryptographically
> verifiable transaction log owned by a central trusted authority. Amazon QLDB tracks each and
> every application data change and maintains a complete and verifiable history of changes
> over time

There are some important features here expanded on in [key concepts](../key-concepts/).

* **Fully Managed** - QLDB is a fully serverless offering that automatically scales to meet demand
* **Ledger Database** - QLDB is a new class of database focused on a ledger
* **Transparent and Immutable** - QLDB has a built-in immutable journal that is append-only with no ability to delete a record. It allows you to easily access the entire change history of any record
* **Cryptographically Verified** - QLDB uses a SHA-256 crytographic hash function with each transaction it commits and stored using hash chaining. This can be used to prove the integrity of any transaction.
* **Central trusted authority** - QLDB is focused on use cases where there is a single owner of the ledger. This is where it differentiates from distributed ledger technologies that require multiple parties to agree a transaction is valid with no single owner.
* **Familiarity** - QLDB supports [PartiQL](https://partiql.org/) which is a SQL-compatible open standard query language. QLDB supports [Amazon ION](http://amzn.github.io/ion-docs/) document data format. This is a superset of JSON that supports a rich type system.








