---
title: "Getting Started"
date: 2020-02-21T18:32:44Z
lastmod: 2020-02-21
weight: 20
draft: false
---

# Getting Started

## Create Ledger

The first step is to create a ledger. There are a number of ways to do this from the AWS Console, AWS CLI and CloudFormation.

**Create ledger via AWS Console**

The easiest way to get started is by creating a ledger through the AWS Console. With this option, you simply specify a name for the ledger, and any option tags.

![Create Ledger through Console](/images/qldb-create-ledger-console.png)


**Create ledger via AWS CLI**

You can also create a ledger directly via the AWS Command Line Interface (CLI), using the createLedger call. With this, you must specify a ledger name and a permissions mode. The only permissions mode currently supported is `ALLOW_ALL`

{{< codeblock "language-shell" >}}
aws qldb create-ledger --name qldb-guide --permissions-mode ALLOW_ALL
{{< /codeblock  >}}

When you create a ledger, deletion protection is enabled by default. This is a feature in QLDB that prevents ledgers from being deleted by any user. You can disable deletion protection on ledger creation by using the `--no-deletion-protection` parameter.

Optionally, you can also specify tags to attach to your ledger.


**Create ledger via AWS CloudFormation**

You can create a ledger using CloudFormation. The example file below uses the same details as the CLI example above.

{{< codeblock "language-yml" >}}
create-ledger-cf.yml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
    qldbGuideLedger:
        Type: AWS::QLDB::Ledger
        Properties:
          Name: qldb-guide
          DeletionProtection: false
          PermissionsMode: ALLOW_ALL
          Tags:
            - 
              Key: name
              Value: qldb-guide
{{< /codeblock >}}

To deploy the template, run the following from a terminal window in the same directory:

{{< codeblock "language-shell" >}}
aws cloudformation deploy --template-file ./create-ledger-cf.yml --stack-name qldb-demo 
{{< /codeblock  >}}


{{< spacer >}}
## Create Table and Index

After creating a ledger, the next step is to create a table and optionally indexes. QLDB is a schemaless database, with no schema enforced on the data in documents within any table.

Indexes are used to improve query performance, and there are a number of limitations:

  * They can only be created on empty tables
  * They can only be created on a single field
  * They cannot be dropped once created
  * There is a maximum of 5 indexes per table
  * Query performance is only improved when you use an equality predicate e.g. fieldName = XYZ

> **NOTE**: There is no current way to create a table or index through the AWS CLI or CloudFormation

There is no current way to create a table or index through the AWS CLI or CloudFormation. The simplest way is to create tables and indexes through the AWS Console. With this option, you click on `Query editor` and then select a ledger. You can then execute the relevant PartiQL statement.

The PartiQL to create a table is:

{{< codeblock "language-shell" >}}
CREATE TABLE table
{{< /codeblock  >}}

The PartiQL to create an index is:

{{< codeblock "language-shell" >}}
CREATE INDEX ON table (field)
{{< /codeblock  >}}



![Create Table through Console](/images/qldb-create-table-console.png)

{{< spacer >}}

## Serverless Framework

One way to ensure that any tables or indexes are created at the same time as the ledger is to use a `custom resource` in CloudFormation. Custom resources allow you to write custom provisioning logic in templates that AWS CloudFormation runs anytime you create, update or delete stacks.

A simple demo project has been put together to supplement this guide which can be found at [QLDB Simple Demo](https://github.com/AWS-South-Wales-User-Group/qldb-simple-demo). 

QLDB is a fully serverless database. The code examples throughout this guide use AWS Lambda to integrate with QLDB. Many frameworks exist for building serverless applications such as AWS SAM and Serverless Framework. The sample project has been created using Serverless Framework. This allows you to use CloudFormation in a `resources` section. The following is a snippet from the `serverless.yml` file


QLDB is a fully serverless database, and it is likely that many people will use AWS Lambda to integrate with it. Many frameworks exist for building serverless applications such as AWS SAM and the Serverless Framework.

Serverless Framework allows you to use CloudFormation in a `resources` section. The example below creates a QLDB ledger whilst calling a custom resource to create a table and create an index.

{{< codeblock  "language-yaml" >}}
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
{{< /codeblock >}}

The custom Lambda function to create the index is only invoked once the Lambda function to create the table has successfully run, and this in turn is dependent on the creation of the ledger itself.

The `ServiceToken` is the ARN of the function that CloudFormation invokes when you create, update or delete the stack. The name of `CreateTableLamdaFunction.ARN` is the Logical ID in the CloudFormation that is created by the Serverless Framework for a function defined as `createTable` in the functions section.


{{< spacer >}}