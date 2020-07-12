---
title: "Interacting with QLDB"
date: 2020-02-21T18:32:44Z
lastmod: 2020-07-11
weight: 25
draft: false
---

# Interacting with QLDB

> **Note** QLDB currently provides supported drivers for Java, Python and Nodejs. All examples in this guide use Nodejs

## Environment Setup

A typical setup of dependencies required to interact with QLDB is shown below:

{{< codeblock "language-json" >}}
{
 "dependencies": {
    "amazon-qldb-driver-nodejs": "^1.0.0",
    "aws-sdk": "^2.706.0",
    "aws-xray-sdk-core": "^3.1.0",
    "ion-js": "^4.0.1",
    "jsbi": "^3.1.3"
  },
}
{{< /codeblock  >}}

* The `amazon-qldb-driver-nodejs` module is the official driver provided by AWS
* The `aws-sdk` and `aws-xray-sdk-core` modules are needed to support tracing with X-Ray
* The `ion-js` and `jsbi` modules are needed to easily interact with Amazon ION documents

{{< spacer >}}

## Connect to Ledger

The first step in writing code to interact with QLDB is to create an instance of `QldbDriver`. This can be done as shown below, passing in the name of the Ledger to connect.

{{< codeblock "language-javascript" >}}
{
const { QldbDriver } = require('amazon-qldb-driver-nodejs');

function createQldbDriver(
  ledgerName = process.env.LEDGER_NAME,
  serviceConfigurationOptions = {}
) {
  const qldbDriver = new QldbDriver(ledgerName, serviceConfigurationOptions);
  return qldbDriver;
}

{{< /codeblock  >}}


## Execute Lambda

## Transaction Control

## CRUD Operations






{{< spacer >}}


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
demonstrated when we add query support to the QLDB Repository where the Result class is parsed to a List of IonStruct.

Please note in this example it is possible to get a document based on email as it has to be unique in this demo

If you wish to query for the document Id explicitly it is possible to do this as follows: 

{{< markupcodeblock  "language-java" >}}
@Override
public IonValue getDocumentId(final String uniqueObjectRef) {
    final String query = "select metadata.id as docId from _ql_committed_licence where data.email = ?";
    qldbSession = LedgerConnection.createQldbSession();
    IonValue documentId = qldbSession.execute(txn -> {
        try {
            final List<IonValue> parameters = Collections.singletonList(ION_MAPPER.writeValueAsIonValue(uniqueObjectRef));
            final Result result = txn.execute(query, parameters);
            
            List<IonStruct> documentList = new ArrayList<>();
            result.iterator().forEachRemaining(row - > {
                docList.add((IonStruct)) row);
            });
            // in this example there is only one record expected in the list
            // this is iterating rather than returning single to demonstrate
            return docList.get(0).get("docId");
        }
        catch (IOException ioe) {
            throw new IllegalStateException(ioe);
        }
    });
    return documentId;
}
{{< /markupcodeblock >}}

#### Querying Documents

Querying documents in QLDB is very similar in process to that of creating documents. The key areas highlighted in the 
example below demonstrate how to use the list of ION structs returned from the query execution to map back to Java using
Jackson.

{{< markupcodeblock  "language-java" >}}

public BicycleLicence findByEmail(final String email) {
    List<BicycleLicences> licences = new ArrayList<>();
    qldbSession = LedgerConnection.createQldbSession();
    final String query = String.format("SELECT * FROM %s where email = ?", "licence");
    qldbSession.execute(txn -> {
        List<IonValue> parameters = null;
        try {
            parameters = Collections.singletonList(ION_MAPPER.writeValueAsIonValue(email);
            List<IonStruct> documents = toIonStructs(txn.execute(query, parameters));
            for(IonStruct struct : documents) {
                StringBuilder stringBuilder = new StringBuilder();
                try (IonWriter jsonWriter = IonTextWriterBuilder.json().withPrettyPrinting().build(stringBuilder)) {
                    rewrite(struct.toString(), jsonWriter);
                }
                BicycleLicence licence = OBJECT_MAPPER.readValue(stringBuilder.toString(), BicycleLicence.class);
                licences.add(licence);
            }
        }
        catch(IOException ioe) {
            throw new IllegalStateException(ioe);
        }
    });
    ....
    // return object from list  
    
public void rewrite(String textIon, IonWriter writer) throws IOException {
    IonReader reader = IonReaderBuilder.standard().build(textIon);
    writer.writeValues(reader);
}

public static List<IonStruct> toIonStructs(final Result result) {
    final List<IonStruct> documentList = new ArrayList<>();
    result.iterator().forEachRemaining(row -> documentList.add((IonStruct) row);
    return documentList;
}
{{< /markupcodeblock >}}

