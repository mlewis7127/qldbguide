---
title: "What is QLDB"
date: 2019-12-08T22:50:19Z
lastmod: 2020-01-03
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


## Why use QLDB

AWS provide a broad portfolio of purpose-built databases. This allows a developer to pick the right tool for the job, rather than using a one size fits all solution for everything. More details on the positioning of QLDB within this portfolio is provided in the section on [database freedom](../database-freedom/).

The concept of a ledger goes back centuries with the first book on double-entry accounting published in 1494. With this method, transactions are first recorded in the journal in chronological order, which is known as the book of primary entry. They are then transferred into the appropriate account or ledger, which is known as the book of second entry. This creates a complete audit trail of every single change.

QLDB creates an audit trail of every change that can be easily queried, with the committed data being immutable and with the ability for the records to be cryptographically verified. This is becoming ever more important in a digital world, with a focus on interparty trust in a supply chain and regulatory and security compliance.

A classic example is from the UK, with the abolition of the paper tax disc in 2014. Up to this point, a physical token had to be displayed in the windscreen of a vehicle, to prove it had been taxed to be driven on public roads. The requirement to tax the vehicle did not go away. Instead, all enforcement activity is carried out against the electronic record. This means that systems not only have to be accurate, but need the ability to go back and track all changes to a record, when the change was made, and why it was carried out. In addition, there is a need to verify that this data has not been altered in any way after the event.

Solutions to these requirements have previously been carried out with traditional database technologies. However, this requires significant engineering effort to create custom audit trails, which is complex, time-consuming and error prone. This gap is now filled with the release of QLDB to support any solution that requires interparty trust or meet compliance requirements. QLDB is a good fit for any event sourced application, and can be considered an ideal Domain Driven Design aggregate store.

QLDB itself originated from a need that AWS had internally for an immutable and searchable transaction log to track every single data plane change that traditional relational databases did not support well. Today, you cannot launch an EC2 instance, send a message on Kinesis, or scale an autoscaling group without using QLDB technology. It is this internal technology, used for many years within AWS, that forms the backbone of the QLDB product.