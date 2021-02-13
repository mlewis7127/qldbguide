+++
title = "Amazon QLDB Guide"
+++
{{< block "grid-2 mt-2" >}}
{{< column >}}
# Amazon QLDB Guide

### Quantum Ledger Database (QLDB) explained

**A guide to the AWS ledger database**

{{< button "docs/" "Read the Docs" >}} {{< button "https://demo.qldbguide.com" "Try the Demo">}}

Inspired by Alex DeBrie's [DynamoDB Guide](https://www.dynamodbguide.com/), which in turn was inspired by Craig Kerstein's [Postgres Guide](http://postgresguide.com/)

{{< /column >}}
{{< column >}}
![logo](/images/QLDB-Guide.svg)
{{< /column >}}
{{< /block >}}
{{< spacer >}}

{{< greysection >}}
<h2>Amazon Quantum Ledger Database (QLDB)</h2>
is a fully managed ledger database that provides a transparent, immutable, and cryptographically verifiable transaction log 
owned by a central trusted authority. Amazon QLDB tracks each and every application data change and maintains a complete 
and verifiable history of changes over time

<img src="/images/QLDB-overview.svg" />

{{</ greysection >}}

{{< whitesection >}}

<h2>PartiQL - SQL Compatible interaction</h2>
You interact with QLDB using PartiQL, a SQL compatible query language familiar to engineers. This allows you to create
tables, insert, update and delete records, and query data.

{{< spacer >}}
{{< animatedtypingblock "exampleQuery" >}}
SELECT v.VIN FROM Vehicle [ AS ] v
{{</ animatedtypingblock >}}
{{< spacer >}}
{{</ whitesection >}}




{{< darksection >}}
<h2>Journal First Architecture</h2>
{{< spacer >}}
{{< block "grid-3" >}}
<div id="no2" class="code">
<h4>Journal First</h4>
No record can be updated without going through the journal first
</div>

<div id="no3" class="code">
<h4>Committed Transactions</h4>
The journal contains only committed transactions
</div>

<div id="no4" class="code">
<h4>Materialized Views</h4>
Committed transactions are projected into materialized views that show both the current state, as well as the history of all revisions to a record.
</div>
{{< /block >}}
{{< spacer >}}
<img src="/images/journal-first.png" /> 

{{</ darksection >}}


{{< greysection >}}
<h2>Cryptographically Verified</h2>
As a transaction is committed, a SHA-256 hash is calculated and stored as part of the block. Each time a new block is 
added, the hash for that block is combined with the hash of the previous block (hash chaining). QLDB also builds a
continuously updating Merkle tree of journal blocks. Using the digest and a proof object for a given document revision,
you can cryptographically verify the specific document is in the same location in the journal and has not been altered
in any way.

<img src="/images/QLDB-Crypto.svg" /> 

{{</ greysection >}}