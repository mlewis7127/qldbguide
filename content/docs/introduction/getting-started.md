---
title: "Getting Started"
date: 2020-02-21T18:32:44Z
lastmod: 2020-02-21
weight: 20
draft: false
---

The first step in using QLDB is to create a ledger. There are a number of options for this.

## Create Ledger

**Create ledger via AWS Console**

The easiest way to get started is by creating a ledger through the AWS Console. With this option, you simply specify a name for the ledger, and any option tags.

![Create Ledger through Console](/images/qldb-create-ledger-console.png)


**Create ledger via AWS CLI**

You can also create a ledger directly via the AWS Command Line Interface (CLI), using the createLedger call. With this, you must specify a ledger name and a permissions mode. The only permissions mode currently supported is ALLOW_ALL

{{< codeblock "language-shell" >}}
aws qldb create-ledger --name <ledger-name> --permissions-mode ALLOW_ALL --tags name=qldb-guide
{{< /codeblock  >}}

When you create a ledger, deletion protection is enabled by default. This is a feature in QLDB that prevents ledgers from being deleted by any user. You can disable deletion protection on ledger creation by using the `--no-deletion-protection` parameter.

Optionally, you can also specify tags to attach to your ledger.


**Create ledger via AWS CloudFormation**

You can create a ledger using CloudFormation. The example file below uses the same details as the CLI example above.

{{< codeblock "language-json" >}}
create-ledger-cf.json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  "Resources" : {
    "myQLDBLedger": {
      "Type": "AWS::QLDB::Ledger",
      "Properties": {
        "DeletionProtection": true,
        "Name": "<ledger-name>",
        "PermissionsMode": "ALLOW_ALL",
        "Tags": [
          {
            "Key": "name",
            "Value": "qldb-guide"
          }
        ]
      }
    }
  }
}
{{< /codeblock >}}
To deploy the template, run the following from a terminal window in the same directory:

```
aws cloudformation deploy --template-file ./create-ledger-cf.json --stack-name qldb-demo 
```

**Create ledger via Serverless Framework**

QLDB is a fully serverless database, and it is likely that many people will use AWS Lambda to integrate with it. Many frameworks exist for building serverless applications such as AWS SAM and the Serverless Framework.

Serverless Framework allows you to use CloudFormation in a `resources` section. The example below will create a QLDB ledger using the same parameters as the other examples.

```
service: qldbguidedemo

provider:
  name: aws
  runtime: nodejs10.x
  stage: dev
  region: eu-west-1

...

resources:
  Resources:
    qldbGuideLedger:
      Type: AWS::QLDB::Ledger
      Properties:
        Name: <ledger-name>
        DeletionProtection: false
        PermissionsMode: ALLOW_ALL
        Tags:
          - 
            Key: name
            Value: qldb-guide
```


## Create Tables

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


**Create table via AWS Console**

The simplest way to get started is to create a table through the AWS Console. With this option, you specify the ledger previously created, click on `query ledger`, and enter the PartiQL statement in the query editor window as shown below:

![Create Table through Console](/images/qldb-create-table-console.png)


**Create table via custom resource**

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


## Insert, Query and Modify Data


