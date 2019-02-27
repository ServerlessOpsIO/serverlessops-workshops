# Serverless Architecture Patterns

In this workshop we'll explore serverless architecture patterns. We'll do that by refactoring our web based micro services, a pattern common with server or container-based services, to be more fitting for a serverless application.

The modules in this workshop are:

* [Asynchronous Event-driven Architecture](./01-asynchronous-event-driven)
* [Synchronous Requests Architecture](./02-synchronous-requests)

## Architectures Covered

There are many different architecture patterns when it comes to serverless on AWS. An introduction to many of them can be found here:

* [Serverless Microservice Patterns for AWS](https://www.jeremydaly.com/serverless-microservice-patterns-for-aws/)

Instead of focusing on all, or even a handful of these, we're going to focus on two high-level patterns. Those patterns are:

* Asynchronous Event-driven Architecture
* Synchronous Requests Architecture

Every Lambda function is triggered by an event and the [AWS documentation on Lambda supported event sources](https://docs.aws.amazon.com/lambda/latest/dg/invoking-lambda-function.html) is worth a read. This is what people mean when they say serverless is an "event-driven architecture". However, depending on the source, an event may be asynchronous or synchronous.  An asynchronous event is one where an event is created to trigger another Lambda function and our code does not wait. Where possible for performance we try and use asynchronous events. As an example, a Lambda function might publish data to an SNS topic where a waiting Lambda function will process the data. The first Lambda function is not concerned with any output from the second Lambda function and does not need to wait before continuing. Since our user doesn't need for us to insert their ride request into DynamoDB, we're going to make communication between *wild-rydes* and *wild-rydes-ride-record* asynchronous.

However, not every problem fits the asynchronous pattern. Many existing server or container web services environments are synchronous web requests. Rearchitecting an entire platform to be a large under taking too. A synchronous event is one where an event is created to trigger another Lambda function and our code waits for data to be returned. (This is what our application architecture is currently.) The *wild-rydes* service needs to request rides from the *wild-rydes-ride-fleet* service and get the fleet member data back to hand to the user. Continuing to use API Gateway in *wild-rydes-ride-fleet* is fine for small scale apps. However, API Gateway introduces additional costs and latency that causes your calling function to run longer and users to wait longer. We will refactor that service to handle direct invocation, bypassing API Gateway.

We've chosen only these two high-level concepts because once you've learned how to perform operations that can be done asynchronously instead of synchronously, and properly perform synchronous requests, you can immediately start solving a wide range of problems with a serverless approach.

