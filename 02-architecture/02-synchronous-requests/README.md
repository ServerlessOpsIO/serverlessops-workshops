# Serverless Synchronous Requests

In this module we'll cover making synchronous requests in between Lambda functions. We'll do that by refactoring *wild-rydes-ride-fleet* *GetUnicorn* to handle direct invocation by *RequestRide* in *wild-rydes*, bypassing API Gateway.

## Goals and Objectives
**Objectives:**
* Understand how to perform synchronous requests when you may have made a web request

**Goals:**
* Refactor *wild-rydes-ride-fleet* *GetUnicorn* to be invoked directly

## Synchronous Requests

## Instructions

In the previous module we noted how our architecture started as a traditional web microservices architecture and how some communication between services could be refactored into an asynchronous event driven architecture which results in faster responses to the user. But not every operation easily fits that pattern easily. And while continuing with a web microservices pattern using API Gateway and Lambda is just fine in many cases, it does add an additional cost (API Gateway requests are charged independent of Lambda invokations) and API Gateway can add additional request latency.

One common operation is fetching data from a data store and returning it to a user. For example, this communication below where out application requests a unicorn from the fleet so it can tell the user who will be picking them up.

![API Gateway Request](../../wild-rydes-apig-request.png)

While this can be done with an asynchronous event-driven architecture, that can be complex to implement and beyond the complexity required for what you're trying to accomplish. And as noted earlier API Gateway has potential minor drawbacks related to cost and latency. If you decide you want to make synchronous requests but it's advantageous for you to not use API Gateway then what do you do?

In this module we'll demonstrate how to have one Lambda function invoke another directly. Instead of an HTTP endpoint providing a stable interface to trigger our code, we will trigger a function handler directly using an [AWS SDK API for Lambda function invocation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lambda.html#Lambda.Client.invoke).

![Lambda Invoke Request](../../wild-rydes-lambda-invoke.png)

### 1. Update wild-rydes-ride-fleet
Deploy the wild-rydes-ride-fleet service. Check out the workshop-operations-01 branch for this workshop module and then deploy the application.

```
$ cd $WORKSHOP/wild-rydes-ride-fleet
$ git checkout -b workshop-architecture-02 origin/workshop-architecture-02
$ sls deploy -v
```

### 2. Refactor *wild-rydes-ride-fleet* *handlers/get_unicorn.py* for direct invocation.

### 3. Add new Lambda function handler in *serverless.yml*

### 4. Refactor *RequestRide* in *wild-rydes* to invoke *GetUnicornInvoke*

## Q&A
