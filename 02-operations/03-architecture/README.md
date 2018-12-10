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
... DDB auto-scaling v. on-demand
<!--
We should cover the questions about this in the intro
* sns -> Lambda -> DDB (ASG)
* sns -> Lambda -> DDB (on-demand)
* sns -> SQS - > Lambda -> DDB (ASG)

-->

...API Gateway Cost...
...dead letter queues, invocation retries...

## Instructions
<!--
    The instructions are written for brevity and clarity but result in an outage.
-->
### 1. Add SNS topic to _wild-rydes_
Create an SNS topic which *RequestRide* will publish ride information too. Instead of publishing to the *wild-rydes-ride-record* service, waiting for a response, and handling errors from the *wild-rydes-ride-record* service, *RequestRide* will publish a message to SNS and continue on. This change will:

* Decrease the average duration of the *RequestRide* function.
* Reduce the need for error handling in *RequestRide* or liklihood of downstream services inducing failure.
* Allow for adding more downstream services without altering *RequestRide**RequestRide*


#### Add SNS topic to serverless.yml
Create an SNS topic in the `Resources` section of *serverless.yml*.

<details>
<summary><strong>Hint 1</strong></summary>
<p>
</p>
</details>

<details>
<summary><strong>Hint 2</strong></summary>
<p>
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>
</p>
</details>

#### Export Topic name via AWS SSM Param Store
So that other services can subscribe to the topic, export the topic ARN to AWS SSM Parameter Store.

<details>
<summary><strong>Hint 1</strong></summary>
<p>
</p>
</details>

<details>
<summary><strong>Hint 2</strong></summary>
<p>
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>
</p>
</details>

#### Refactor *RequestRide* to publish to new SNS topic
The *RequestRide* function now needs to publish to the SNS topic.

Edit _serverless.yml_ to have the SNS topic's ARN to the *RequestRide* function as an environmental variable.
<details>
<summary><strong>Hint 1</strong></summary>
<p>
</p>
</details>

<details>
<summary><strong>Hint 2</strong></summary>
<p>
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>
</p>
</details>


Refactor *handlers/request_ride.py* to publish ride information to the SNS topic instead of the *wild-rydes-ride-record* via HTTP.
<details>
<summary><strong>Hint 1</strong></summary>
<p>
</p>
</details>

<details>
<summary><strong>Hint 2</strong></summary>
<p>
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>
</p>
</details>

#### Deploy the updated *wild-rydes* service
Deploy the newly updated *wild-rydes*.

*NOTE: This will create a minor service disruption as ride records will not be sent to wild-rydes-ride-record until that service is updated in the following steps. We are proceeding this way for clarity in this workshop.*

```
$ cd $WORKSHOP/wild-rydes
$ sls deploy -v
```

### 2. Add DynamoDB scaling in *wild-rydes-ride-record*
Add the ability to scale to your DynamoDB table by enabling on-demand pricing. There isn't enough data to properly size the read and write capacity on the table so this is the best choice for scaling our table.


### 3. Refactor handlers/put_ride_record.py in wild-rydes-ride-record to support API Gateway and SNS events
Refactor *handlers/put_ride_record.py* so there are Lambda functions for both API Gateway and SNS. This allows the code to handle different use cases with minimal code duplication. You'll do this by replacing *PutRideRecord* with *PutRideRecordSns* and *PutRideRecordHttp*.

#### Create handlers/put_ride_record.put_ride_record
Create the Python function put_ride_record() in the file *handlers/put_ride_record.py*. This will handle the common code for inserting ride data into DynamoDB. The function will take a ride event data and put it into DynamoDB.

The ride event has the following shape:
```
```

<!-- FIXME: Add instructions -->
```diff
```

#### Create handlers/put_ride_record.handler_http
Create the Python function *handler_http()* in the file *handlers/put_ride_record.py*. This function will take an API Gateway event, extract the ride information from the `body` key, and pass it to the *put_ride_record()* Python function.

<details>
<summary><strong>API Gateway Event</strong></summary>
<p>

```
```
</p>
</details>

<details>
<summary><strong>Code</strong></summary>
<p>

```python
```
</p>
</details>


#### Create handlers/put_ride_record.handler_sns
Create the Python function *handler_http()* in the file *handlers/put_ride_record.py*. This function will take an SNS event, extract the ride information from the `Records[0].Sns.Message` key, and pass it to the *put_ride_record()* Python function.

<details>
<summary><strong>API Gateway Event</strong></summary>
<p>

```
```
</p>
</details>

<details>
<summary><strong>Code</strong></summary>
<p>

```python
```
</p>
</details>

### 4. Create new function handlers for different event sources in wild-rydes-ride-record
Add new function handlers for the new Python functions.

#### Update *PutRideRecord* function to become *PutRideRecordHttp* in *serverless.yml*
<!-- FIXME: Will this cause a service disruption? -->
Rename *PutRideRecord* to *PutRideRecordHttp*.

*NOTE: This will change the name of the CloudWatch log group where log messages are sent to. In our case this change doesn't matter but it may in a production nvironment. We're renaming the function for clarity and ease of understanding.*

<details>
<summary><strong>Code</strong></summary>
<p>

```python
```
</p>
</details>

#### Create PutRideRecordSns function in serverless.yml
Add a Lambda function for *PutRideRecordSns*. This function will be much like *PutRideRecordHttp* but will take an SNS event. You will need to get the ARN of the SNS topic from step 1 via SSM Parameter Store.

How to trigger a Lambda function via an SNS event can be found here:
* [Serverless Framework SNS Events](https://serverless.com/framework/docs/providers/aws/events/sns/)

<details>
<summary><strong>Hint 1</strong></summary>
<p>

* Obtain the SNS topic ARN from AWS SSM Param Store
* Add the SNS event to the *PutRideRecordSns* function

</p>
</details>

<details>
<summary><strong>Code</strong></summary>
<p>

```python
```
</p>
</details>

### 6. Add dead letter queue to _PutRideRecordSns_
Add a dead letter queue for the _PutRideRecordSns_ Lambda function. We need a place to store messages delivered by SNS when if _PutRideRecordSns_ function failed to successfully execute after it has exhausted its retry limit. While we've handled the DynamoDB scaling issue that would most easily trigger this scenario, we should still handle the inevitable unpredictability of the cloud.

We'll use the Serverless Framework plugin called.....

<details>
<summary><strong>Code</strong></summary>
<p>

```yaml
```
</p>
</details>

### 7. Deploy *wild-rides-ryde-record*
Deploy the updates *wild-rides-ryde-record* service.

### 8. Use Artillery to trigger the RequestRide function.
Using Artillery, trigger the *RequestRide* function. This time you should see noe errors.

## Q&A

### 1. DynamoDB auto-scaling

Q. How does DynamoDB auto-scaling work

Q. How does this fail when traffic bursts too quickly?

Q. What is an alternative way to solving the failure?
<!-- Use on-demand DDB provisioning. -->

Q. How might you re-architect your application to better handle this?

### 2. API Gateway Costs

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

### 3. DynamoDB
Q. Calculate the approximate cost of the following designs for a million messages.
<!--
We should cover the questions about this in the intro
* sns -> Lambda -> DDB (ASG)
* sns -> Lambda -> DDB (on-demand)
* sns -> SQS - > Lambda -> DDB (ASG)
-->

### 4. SNS
Q. Create a dead letter queue processor function.
