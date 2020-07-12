---
title: "Current Limitations"
date: 2020-02-21T18:43:05Z
lastmod: 2020-07-11
weight: 50
draft: false
---

# Current Limitations

## Unique Key

{{< spacer >}}

## IAM Permission Granularity

{{< codeblock "language-yaml" >}}
iamRoleStatements:
    - Effect: Allow
      Action: 
        - qldb:SendCommand
      Resource: arn:aws:qldb:#{AWS::Region}:#{AWS::AccountId}:ledger/qldb-simple-demo
{{< /codeblock  >}}

{{< spacer >}}

## PartiQL Operations Support

{{< spacer >}}

## KMS

{{< spacer >}}

## Backup and Restore

{{< spacer >}}




  