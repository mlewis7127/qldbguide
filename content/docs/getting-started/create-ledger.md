---
title: "Create Ledger"
date: 2019-12-28T21:33:32Z
lastmod: 2019-12-28
draft: false
weight: 10
---

The first step in using QLDB is to create a ledger. There are a number of options for this.

### Create ledger via AWS Console
The easiest way to get started is by creating a ledger through the AWS Console. With this option, you simply specify a name for the ledger, and any option tags.

![Create Ledger through Console](/images/qldb-create-ledger-console.png)


### Create ledger via AWS CLI
You can also create a ledger directly via the AWS Command Line Interface (CLI), using the createLedger call. With this, you must specify a ledger name and a permissions mode. The only permissions mode currently supported is ALLOW_ALL


```
aws qldb create-ledger --name <ledger-name> --permissions-mode ALLOW_ALL --tags name=qldb-guide
```

When you create a ledger, deletion protection is enabled by default. This is a feature in QLDB that prevents ledgers from being deleted by any user. You can disable deletion protection on ledger creation by using the `--no-deletion-protection` parameter.

Optionally, you can also specify tags to attach to your ledger.


### Create ledger via AWS CloudFormation
You can create a ledger using CloudFormation. The example file below uses the same details as the CLI example above.

```
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
```
To deploy the template, run the following from a terminal window in the same directory:

```
aws cloudformation deploy --template-file ./create-ledger-cf.json --stack-name qldb-demo 
```

### Create ledger via Serverless Framework
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




