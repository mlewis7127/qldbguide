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

The Web UI is a `React` application built using the `Amplify` framework that gets deployed to `S3` (and `CloudFront`). It uses the pre-built UI components to create the entire authentication flow. This utilises the `withAuthenticator` as a higher-order Component (`HoC`) that wraps `AmplifyAuthenticator`. It also ensures that a user has to be authenticated. When the user creates an account, they will automatically be setup in the `Cognito User Pool`

The `API Gateway` component acts as the gatekeeper to all backend services. By default API Gateway implements an account level rate limit. This is further broken down by setting specific request throttling limits on each API in the stage. A WAF ACL is also configured with a rate-based rule by IP address.

The `API Gateway` uses an `Authorizer` to control access to the API using the `Cognito User Pool`. This means that a user must be signed in to the web application to make a request. The React app gets the users current session using `Auth.currentSession()` which returns a `CognitoUserSession` object which contains a JWT `accessToken`, `idToken`, and `refreshToken`. It then calls `getIdToken().getJwtToken()` on the `CognitoUserSession` object, and sets this as the bearer token in the `Authorization` header of the HTTP request.

The user pool access token contains claims about the authenticated user, a list of the user's groups, and a list of scopes. The purpose of the access token is to authorize API operations in the context of the user in the user pool. The access token is represented as a JSON Web Token (JWT). The JWT signature is a hashed combination of the header and the payload. Amazon Cognito generates two pairs of RSA cryptographic keys for each user pool. One of the private keys is used to sign the token. `API Gateway` verifies the signature of the JWT token to make sure it is valid.

`API Gateway` use proxy integration to route authorized requests to the relevant `Lambda` function. A separate `Lambda` function is used to ensure adoption of the least privilege principle. A set of `Lambda` functions are used to update `QLDB` as the source of truth for Bicycle Licence information.

At the same time as `QLDB` is updated, `QLDB Streams` is configured to publish all changes onto a `Kinesis Data Stream`. `Lambda` functions are used to subscribe to the data stream, and then to update either `DynamoDB` or `Elasticsearch` with the relevant information, to show how streams processing provides an event driven architecture.