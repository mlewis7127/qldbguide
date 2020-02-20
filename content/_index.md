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
![logo](/images/QLDB_light-bg.png)
{{< /column >}}
{{< /block >}}

{{< block "grid-3 mt-3" >}}
{{< column >}}
<div id="no2">
<div><h4>Create a Ledger</h4></div>
<div class="code">
    CreateLedgerRequest request = new CreateLedgerRequest()
                .withName(ledgerName)
                .withPermissionsMode(PermissionsMode.ALLOW_ALL);
</div>
</div>
{{< /column >}}
{{< column >}}
<div id="no3">
<div><h4>Create a Ledger</h4></div>
<div class="code">
    CreateLedgerRequest request = new CreateLedgerRequest()
                .withName(ledgerName)
                .withPermissionsMode(PermissionsMode.ALLOW_ALL);
</div>
{{< /column >}}
{{< column >}}
<div id="no4">
<div><h4>Create a Ledger</h4></div>
<div class="code">
    CreateLedgerRequest request = new CreateLedgerRequest()
                .withName(ledgerName)
                .withPermissionsMode(PermissionsMode.ALLOW_ALL);
</div>
{{< /column >}}
{{< /block >}}







