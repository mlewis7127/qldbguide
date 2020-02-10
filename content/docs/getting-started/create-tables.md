---
title: "Create Tables"
date: 2019-12-28T21:33:45Z
lastmod: 2020-01-25
draft: false
weight: 20
---

Once you have a ledger, the next step is to create a table in the ledger. When you interact with QLDB, you use a SQL-compatible language called PartiQL. We have already covered the fact that QLDB uses a journal-first architecture, where no record can be updated without going through the journal first. Once committed to the journal, the changes are then projected into user created tables, which can be queried.

QLDB can be considered a schemaless database, as there is no schema enforced on the data in documents within any table. Any schema can only be enforced by an application once the data has been read.

Database design is important in QLDB. PartiQL has support for nested content which can significantly simplify how you interact with a ledger. In addition, currently QLDB has limited indexing capability and does not support all PartiQL operations. There are restrictions that you take into account.

* Indexes can improve query performance, however
  * They can only be created on empty tables
  * They can only be created on a single field
  * They cannot be dropped once created
  * There is a maximum of 5 indexes per table
  * Query performance is only improved when you use an equality predicate e.g. fieldName = XYZ
* There is a maximum of 20 active tables per ledger


### Create table via AWS Console
The simplest way to get started is to create a table through the AWS Console. With this option, you specify the ledger previously created, click on `query ledger`, and enter the PartiQL statement in the query editor window as shown below:

![Create Table through Console](/images/qldb-create-table-console.png)


### Create table via custom resource

There is currently no way of creating a table and an index in CloudFormation or via the CLI. One way to ensure that the table and indexes are created along with the ledger is to make use of a custom resource in CloudFormation. Custom resources enable you to write custom provisioning logic in templates that AWS CloudFormation runs anytime you create, update (if you changed the custom resource), or delete stacks.

The following is a snippet from a `serverless.yml` file to show how this is achieved:

```
resources:
  Resources:
    qldbGuideLedger:
      Type: AWS::QLDB::Ledger
      Properties:
        Name: qldb-simple-demo-${self:provider.stage}
        DeletionProtection: false
        PermissionsMode: ALLOW_ALL
        Tags:
          - 
            Key: name
            Value: qldb-simple-demo

    qldbTable:
      Type: Custom::qldbTable
      DependsOn: qldbGuideLedger
      Properties:
        ServiceToken: !GetAtt CreateTableLambdaFunction.Arn
        Version: 1.0  #change this to force redeploy

    qldbIndex:
      Type: Custom::qldbIndexes
      DependsOn: qldbTable
      Properties:
        ServiceToken: !GetAtt CreateIndexLambdaFunction.Arn
        Version: 1.0  #change this to force redeploy
```

This shows how the custom Lambda function to create the index is only invoked once the Lambda function to create the table has successfully run, and this in turn is dependent on the creation of the ledger itself.

The `ServiceToken` is the ARN of the function that CloudFormation invokes when you create, update or delete the stack. The name of `CreateTableLamdaFunction.ARN` is the Logical ID in the CloudFormation that is created by the Serverless Framework for a function defined as `createTable` in the functions section.

The full working example can be found in [QLDB Simple Demo](https://github.com/mlewis7127/qldb-simple-demo) and has been tagged using v0.2
