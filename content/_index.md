+++
title = "Amazon QLDB Guide"
+++
{{< block "grid-2 mt-2" >}}
{{< column >}}
# Amazon QLDB Guide

### Quantum Ledger Database (QLDB) explained

**A guide to the AWS centralised ledger database**

{{< button "docs/" "Read the Docs" >}}

Inspired by Alex DeBrie's [DynamoDB Guide](https://www.dynamodbguide.com/), which in turn was inspired by Craig Kerstein's [Postgres Guide](http://postgresguide.com/)

{{< /column >}}
{{< column >}}
![logo](/images/QLDB-Guide.svg)
{{< /column >}}
{{< /block >}}
{{< spacer >}}
{{< darksection >}}
{{< block "grid-3" >}}
<div id="no2" class="code">
<h4>Create a Ledger</h4>
Create a new ledger with tables and indexes
</div>

<div id="no3" class="code">
<h4>Insert and Update Data</h4>
Insert a new document and update to create multiple document revisions
</div>

<div id="no4" class="code">
<h4>Query data and Crytographically Verify</h4>
Query the current state of a document or its historical states. Crytographically
verify the change history of the data
</div>
{{< /block >}}
{{</ darksection >}}
{{< greysection >}}
<h2>Amazon Quantum Ledger Database (QLDB) is</h2>
a fully managed ledger database that provides a transparent, immutable, and cryptographically verifiable transaction log 
owned by a central trusted authority. Amazon QLDB tracks each and every application data change and maintains a complete 
and verifiable history of changes over time

{{< spacer >}}

{{< animatedtypingblock "exampleQuery" >}}
SELECT v.VIN FROM Vehicle [ AS ] v
{{</ animatedtypingblock >}}
{{</ greysection >}}
