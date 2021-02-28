---
title: About the Demo
weight: 11
---

## Background

The `QLDB Demo` has been set up to allow to people to try out a working end to end demo application that utilises QLDB at its heart. The demo is based on the concept of a fictional `Bicycle Licence`. This is a licence that is held by an individual which allows them to ride a bicycle. There is the concept of penalty points which can be added or removed, but which exist as part of the licence.

QLDB maintains an immutable and cryptographically verifiable audit trail of all changes made to the licence. This brings the advantages of an event sourced application, and all changes have an event associated with them:

* `BicycleLicenceCreated`
* `PenaltyPointsAdded`
* `PenaltyPointsRemoved`
* `ContactAddressUpdated`
* `LicenceDeleted`

The system of record resides in QLDB, which acts as the source of truth. All changes are also streamed out in near real-time to other downstream purpose built database engines, showing the value of `QLDB Streams`. In terms of the demo, this means you can search for the licence in `Elasticsearch` or view all licences you have created from `DynamoDB`.

## Security

The demo itself requires a user to create an account which includes a valid email address they have access too. This is needed to receive a verification code to complete the signup process using `Amazon Cognito`.

Note that no access is provided to data such as the email address stored in the Cognito User Pool, and the data may be deleted at any point as new features are rolled out to the demo.

Access to the backend REST APIs is controlled using `Amazon Cognito` user pools as an authorizer. The web application makes an API call including the Identity Token of the signed-in user. The backend APIs are protected by `API Gateway` which validates the claimed identity against the one from the user pool.

The authenticated identity from the Cognito User Pool is retrieved from the claims in the `requestContext.authorizer.claims` section of the `event` passed into the `Lambda` function. This is stored as part of the document in QLDB and streamed out. This is used to ensure that you can only view records in `Elasticsearch` or `DynamoDB` that you created yourself.

## Architecture

The architecture that gets deployed for the QLDB Demo application is shown below:

![QLDB Demo Architecture](/images/qldbdemo-architecture.png)

The Web UI is a `React` application built using the `Amplify` framework that gets deployed to `S3` (and `CloudFront`). It uses the pre-built UI components to create the entire authentication flow. This utilises the `withAuthenticator` as a higher-order Component (`HoC`) that wraps `AmplifyAuthenticator`. It also ensures that a user has to be authenticated e.g. have logged in.
