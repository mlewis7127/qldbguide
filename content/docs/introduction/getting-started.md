---
title: "Getting Started"
date: 2020-02-21T18:32:44Z
lastmod: 2020-02-21
weight: 20
draft: false
---

# Getting Started

In order for you to be able to execute PartiQL statements in your language of choice, you will be required to add the
language dependency to your preferred build tool. The most common examples are listed below:

### Java

#### Maven

{{< markupcodeblock  >}}
<dependency>
    <groupId>software.amazon.qldb</groupId>
    <artifactId>amazon-qldb-driver-java</artifactId>
    <version>1.0.1</version>
</dependency>
{{< /markupcodeblock  >}}

#### Gradle

{{< codeblock "language-shell" >}}
dependencies {
    compile "software.amazon.qldb:amazon-qldb-driver-java:1.0.1"
}
{{< /codeblock  >}}

### NodeJS

{{< codeblock "language-json" >}}
{
  "dependencies": {
    "amazon-qldb-driver-nodejs": "0.1.0-preview.2"
   }
}
{{< /codeblock  >}}

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

{{< codeblock "language-shell" >}}
aws cloudformation deploy --template-file ./create-ledger-cf.json --stack-name qldb-demo 
{{< /codeblock  >}}

**Create ledger via Serverless Framework**

QLDB is a fully serverless database, and it is likely that many people will use AWS Lambda to integrate with it. Many frameworks exist for building serverless applications such as AWS SAM and the Serverless Framework.

Serverless Framework allows you to use CloudFormation in a `resources` section. The example below will create a QLDB ledger using the same parameters as the other examples.

{{< codeblock  "language-yaml" >}}
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
{{< /codeblock  >}}


{{< spacer >}}
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

This shows how the custom Lambda function to create the index is only invoked once the Lambda function to create the table has successfully run, and this in turn is dependent on the creation of the ledger itself.

The `ServiceToken` is the ARN of the function that CloudFormation invokes when you create, update or delete the stack. The name of `CreateTableLamdaFunction.ARN` is the Logical ID in the CloudFormation that is created by the Serverless Framework for a function defined as `createTable` in the functions section.

The full working example can be found in [QLDB Simple Demo](https://github.com/mlewis7127/qldb-simple-demo) and has been tagged using v0.2

{{< spacer >}}

## Insert, Query and Modify Data

In order to prepare an application to use QLDB the following elements should be added to your build tool. In this 
example we have used Maven:

{{< markupcodeblock  >}}
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>bom</artifactId>
            <version>2.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
{{< /markupcodeblock  >}}

So that it is possible to map between entities and the Ion dataformat, Jackson Object Mapper now supports converting 
between Ion and Java DTO's or JSON.

To add this dependency, please add the following to your pom.file:

{{< markupcodeblock  >}}
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-ion</artifactId>
    <version>2.10.3</version>
</dependency>
{{< /markupcodeblock  >}}

> Please note: in some of the associated demo applications the following library is also added as a dependency
> <dependency>
>     <groupId>com.amazonaws</groupId>
>     <artifactId>aws-java-sdk-qldb</artifactId>
>     <version>1.11.693</version>
> </dependency>
> This is due to the Jackson Mapper using older versions of the classes and means it is not possible to map between
> some of the types under the new package structure.

#### QLDB Ledger Connection Class

To be able to initialise PartiQL statements with the QLDB Driver, as per the AWS examples, a Ledger Connection class
is created:

{{< codeblock  "language-java" >}}
public static PooledQldbDriver pooledDriver = createPooledQldbDriver();

/**
 * Method to create a pooled qldb driver for creating sessions
 *
 * @return pooled qldb driver
 */
public static PooledQldbDriver createPooledQldbDriver() {
    AmazonQLDBSessionClientBuilder builder = AmazonQLDBSessionClientBuilder.standard();
    builder.setRegion(LedgerConstants.REGION);
    if(null != endpoint) {
        builder.setEndpointConfiguration(new AwsClientBuilder.EndpointConfiguration(endpoint, region));
    }
    if(null != credentialsProvider) {
        builder.setCredentials(credentialsProvider);
    }
    return PooledQldbDriver.builder()
        .withLedger(LedgerConstants.LEDGER_NAME)
        .withSessionClientBuilder(builder)
        .build();
}
{{< /codeblock >}}

In the LedgerConnection it will also be worth adding a function to get the AmazonQLDB Client if you intend on running
queries to verify documents which involve getting revisions. The following codeblock can be used to enable this:

{{< codeblock  "language-java" >}}
/**
 * Method to create an amazon qldb client that can be used when
 * verifying documents and getting revisions.
 *
 * @return amazon qldb client
 */
public static AmazonQLDB createQLDBClient() {
    AmazonQLDBClientBuilder builder = AmazonQLDBClientBuilder.standard();
    builder.setRegion(LedgerConstants.REGION);
    if(null != endpoint) {
        builder.setEndpointConfiguration(new AwsClientBuilder.EndpointConfiguration(endpoint, region));
    }
    if(null != credentialsProvider) {
        builder.setCredentials(credentialsProvider);
    }
    return builder.build();
}
{{< /codeblock >}}

To create a QLDB Session it is useful to add a helper function that can be used later in the repository classes.

{{< codeblock  "language-java" >}}
public static QldbSession createQldbSession() {
    return driver.getSession();
}
{{< /codeblock >}}

### Repository Setup

In the examples in this section we have used the Spring data JPA repository which provides the scaffolding for 
Auto-wiring and method structure. 

> Longer term we will look to add support to the Spring data repositories much like those familiar with Spring 
> will have seen support for other common databases https://spring.io/projects/spring-data.

To use the Spring or Spring boot support, the following can be added to the pom.xml file. Please note that Spring
is not necessary or used within te Repository class itself and does not need to be used.

{{< markupcodeblock >}}

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.8.RELEASE</version>
    <relativePath />
</parent>

....

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
{{< /markupcodeblock >}}

{{< markupcodeblock  "language-java" >}}
public class BicycleLicenceQLDBRepository implements CrudRepository<BicycleLicence, String> {

    // Used for Ion to Java Mapping
    private IonValueMapper ION_MAPPER = new IonValueMapper(IonSystemBuilder.standard().build());
    // Used for Java > Json Mapping
    private ObjectMapper OBJECT_MAPPER = new ObjectMapper();
    
    private PooledQldbDriver pooledQldbDriver;
    private QldbSession qldbSession;
    
    {
        pooledQldbDriver = LedgerConnextion.createPooledQldbDriver();
        MAPPER.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    }

}
{{< /markupcodeblock >}}

If you wish to use the Spring Repository style implementation then it can be referenced using @Autowired or by injecting
via the instantation through Constructors as follows:

{{< codeblock  "language-java" >}}
    // calling class
    
   .....
   
   private BicycleLicenceQldbRepository repository;
   
   public MyCallingClass(BicycleLicenceQldbRepository qldbRepository) {
       this.repository = qldbRepository;
   }
{{< /codeblock >}}

#### Inserting Documents

The CrudRepository interface forces method signature implementations for the key Create, Retrive, Update and Delete 
functions. The create method utilises the LedgerConnection pooled QLDB Driver to insert records in to the ledger.

{{< markupcodeblock  "language-java" >}}
@Override
public <S extends BicycleLicence> S save(S s) {
    qldbSession = LedgerConnection.createQldbSession();
    qldbSession.execute(txn -> {
        try {
            final String query = String.format("INSERT INTO %s ?, "licence");
            final IonValue document = (IonValue) ION_MAPPER.writeValueAsIonValue(s);
            final List<IonValue> parameters = Collections.singletonList(document);
            Result result = txn.execute(query, parameters);
        }
        catch (IOException ioe) {
            throw new IllegalStateException(ioe);
        }
    });
    return s;
}
{{< /markupcodeblock >}}

##### Retrieving the QLDB Document Id

It is possible that you can use the returning Result class to get hold of the generated Document Id, this will be
demonstrated when we add query support to the QLDB Repository where the Result class is parted to a List<IonStruct>.


