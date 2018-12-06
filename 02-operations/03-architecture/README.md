# Serverless Architecture And Design

<!-- Focus on cost & performance (bypass APIG to lower cost and increase performance), and resiliency (DLQs) -->

<!-- tasks
    * refactor wild-rydes to have SNS
    * Refactor wild-rydes record to subscribe to SNS
    * Refcator handler to deal with both.
    * add DLQ to SNS
-->

In this module we're cover architectural patterns to address concerns about reliability, performance, and cost. We'll do this by refactoring the _wild-rydes-ride-record_ service.

## Goals and Objectives

__Objectives:__

* Understand architectural decisions that affect
    * Performance
    * Reliability
    * Cost

Goals:

* Increase speed performance of _wild-rydes_ and decrease cost of _wild-rydes-request-ride_ by bypassing API Gateway to _wild-rydes-request-ride_.
* Increase reliability of ride event delivery by implementing Lambda dead letter queues

## Serverless Architecture And Design
...API web service versus SNS event driven...
...API Gateway Cost...
...dead letter queues, invocation retries...

## Instructions

### 1. Add SNS topic to _wild-rydes_

#### Add SNS topic to serverless.yml

#### Export Topic name via AWS SSM Param Store

### 2. Refactor _RequestRide to publish to new SNS topic

### 3. Deploy wild-rydes and request a ride

### 4. Refactor handlers/put_ride_record.py in wild-rydes-ride-record to support API Gateway and SNS events

#### Create handlers/put_ride_record.put_ride_record

#### Create handlers/put_ride_record.handler_http

#### Create handlers/put_ride_record.handler_sns

### 5. Create new function handlers for different event sources in wild-rydes-ride-record

#### Update PutRideRecord function to become PutRideRecordHttp in serverless.yml
<!-- Note: Changing the function name changes thelog names -->

#### Create PutRideRecordSns function in serverless.yml

#### Subscribe PutRideRecordSns function to _wild-rydes_ SNS topic

### 6. Add dead letter queue to _PutRideRecordSns_

## Q&A

### 1. API Gateway Costs

Q. Calculate the costs of Lambda and API Gateway for _wild-rydes-ride-record_ _PutRideRecord_ at

  * 100ms run time
  * 128MB RAM
  * 10,000,000 requests in a month

### 2. Refactoring

Q. How might you reorder the steps in this workshop if you wanted to make incremental changes that you could deploy along the way instead of making one larger refactoring?

The instructions in this module are meant to be easy to follow but aren't necesarilly the best order of operations for refactoring an application. You can reorder these steps (and add additional steps) to you make incremental progress which makes tracking down mistakes easier.

<details>
<summary><strong>Output</strong></summary>

1. Refactor handlers/put_ride_record.py in _wild-rydes-ride-record_ to create *handlers/put_ride_record.put_ride_record* and *handlers/put_ride_record.handler_http*
1. Update PutRideRecord function to become PutRideRecordHttp in serverless.yml
1. Deploy *wild-rydes-ride-record*
1. Successfully request a ride
1. Add SNS topic to _wild-rydes_ and export topic name to SSM Param store
1. Deploy *wild-rydes*
1. Create handlers/put_ride_record.handler_sns
1. Create PutRideRecordSns function in serverless.yml
1. Subscribe PutRideRecordSns function to _wild-rydes_ SNS topic
1. Deploy *wild-rydes-ride-record*
1. Refactor _RequestRide_ to publish to new SNS topic
1. Deploy *wild-rydes* and request a ride

<p>
</p>
</details>


