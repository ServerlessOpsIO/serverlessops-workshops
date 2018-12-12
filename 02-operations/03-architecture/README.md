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
<!-- FIXME: Add diagram of current arch and future arch -->

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
<!-- FIXME: Add diagram of what to build -->
Create an SNS topic which *RequestRide* will publish ride information too. Instead of publishing to the *wild-rydes-ride-record* service, waiting for a response, and handling errors from the *wild-rydes-ride-record* service, *RequestRide* will publish a message to SNS and continue on.

This change will:
* Decrease the average duration of the *RequestRide* function.
* Reduce the need for error handling in *RequestRide* or liklihood of downstream services inducing failure.
* Allow for adding more downstream services without altering *RequestRide**RequestRide*


#### Add SNS topic to *serverless.yml*
Create an SNS topic in the `Resources` section of *serverless.yml*. This SNS topic is where *RequestRide* will publish messages to for containing information about the rides it has dispatched. These messages are intended to be processed by downstream services that do not need to run before the Wild Rydes user has been notified of their impending ride.

The CloudFormation resource name (not the SNS Topic name) should be _RidesSnsTopic_.

<details>
<summary><strong>Hint 1</strong></summary>
<p>
Reference the CloudFormation documentation here:

* [CloudFormation AWS::SNS::Topic](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sns-topic.html)

None of the resource properties are required, including topic name. If you decided to set the *TopicName* property then you would have to ensure the SNS topic's name did not conflict with other parallel deployments of *wild-rydes*. You're probably best off letting CloudFormation do the work of configuring the SNS topic's name.
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -83,6 +83,10 @@ functions:

 resources:
   Resources:
+
+    RidesSnsTopic:
+      Type: AWS::SNS::Topic
+
     ## Specifying the S3 Bucket
     StaticSite:
       Type: AWS::S3::Bucket
```
</p>
</details>

#### Publish SNS topic ARN
So that other services can subscribe to the SNS topic, export the topic ARN to AWS SSM Parameter Store. Create an SSM Param Store parameter named `/wild-rydes/STAGE/SnsTopicArn`. (Where STAGE is replaced by the value of *${self:provider.stage}* from Serverless Framework.) The value of the parameter can be obtained by using the Cloudfromation *Ref* function on the *RidesSnsTopic* CloudFormation resource. The *Ref* function applied to a *AWS::SNS::Topic* Cloudformation resource yields the ARN of the resource.

<details>
<summary><strong>Hint 1</strong></summary>
<p>
Reference the CloudFormation documentation here:

* [CloudFormation AWS::SSM::Parameter](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ssm-parameter.html)
* [CloudFormation Ref function](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html)

</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -87,6 +87,14 @@ resources:
     RidesSnsTopic:
       Type: AWS::SNS::Topic

+    RidesSnsTopicArnSsmParam:
+      Type: "AWS::SSM::Parameter"
+      Properties:
+        Name: "/${self:service}/${self:provider.stage}/SnsTopicArn"
+        Type: String
+        Value:
+          Ref: RidesSnsTopic
+
     ## Specifying the S3 Bucket
     StaticSite:
       Type: AWS::S3::Bucket
```
</p>
</details>

#### Refactor *RequestRide* to publish to new SNS topic
The *RequestRide* function now needs to publish to the SNS topic.

Edit _serverless.yml_ to pass the SNS topic's ARN to the *RequestRide* function as an environmental variable named *RIDES_SNS_TOPIC_ARN*. Use the CloudFormation *Ref* function again to get the ARN of the SNS topic you  created in previous steps. While you're here, remove the *RIDE_RECORD_URL* environmental variable as it will no longer be necessary.

<details>
<summary><strong>Hint 1</strong></summary>
<p>
Serverless Framework Documentation:

* [AWS Lambda Environmental Variables](https://serverless.com/framework/docs/providers/aws/guide/functions#environment-variables)
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>

```
--- a/serverless.yml
+++ b/serverless.yml
@@ -67,7 +67,8 @@ functions:
     environment:
       LOG_LEVEL: "${self:custom.log_level}"
       REQUEST_UNICORN_URL: "${self:custom.request_unicorn_url}"
-      RIDE_RECORD_URL: "${self:custom.ride_record_url}"
+      RIDES_SNS_TOPIC_ARN:
+        Ref: RidesSnsTopic
     events:
       - http:
           path: /ride
```
</p>
</details>


Refactor *handlers/request_ride.py* to publish ride information to the new SNS topic instead of the *wild-rydes-ride-record* via HTTP. In a later step we'll have a Lambda function from the *wild-rydes-ride-record* service subscribe to the topic.

Start by importing the Python AWS SDK boto3 module. Then get the value of *RIDES_SNS_TOPIC_ARN* from the runtime environment. Next create a Boto3 *SNS.Client* object. Do these steps at the top of the file just before Python functions start being defined. (This is so the SNS client object persists across multiple Lambda function invocations.)

Furthger down in the code, you can replace the *_post_ride_record()* Python function with one called *_publish_ride_record()*. The *_publish_ride_record()* function will take two arguments, one containing ride information and a second containing the SNS topic ARN to publish to.. The ride argument is no different than the ride argument that was passed to *_post_ride_record()*. The function will call the SNS.Client.publish() function to publish the ride message as the message body and the function does not return a value. Remember to convert the ride information to a JSON string using by calling the *json.dumps()* Python function.

Finally, remember to add the Thundra *Traceable* decorator to the *_publish_ride_record()* function.


<details>
<summary><strong>Hint 1</strong></summary>
<p>

Python Documentation:

* [Boto3 SNS.Client](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/sns.html#client)

</p>
</details>

<details>
<summary><strong>Hint 2</strong></summary>
<p>

```diff
--- a/handlers/request_ride.py
+++ b/handlers/request_ride.py
@@ -6,6 +6,7 @@ import json
 import os
 import uuid

+import boto3
 from botocore.vendored import requests

 from thundra.thundra_agent import Thundra
@@ -22,8 +22,14 @@ _logger = logging.getLogger(__name__)
 _logger.addHandler(ThundraLogHandler())

 REQUEST_UNICORN_URL = os.environ.get('REQUEST_UNICORN_URL')
+
+# FIXME: SNS client
+# - change this line to create a variable called RIDES_SNS_TOPIC_ARN by
+#   fetching the RIDES_SNS_TOPIC_ARN environmental variable.
 RIDE_RECORD_URL = os.environ.get('RIDE_RECORD_URL')

+# - Create an SNS Client object called SNS_CLIENT by calling boto3.client('sns')
+

 @Traceable(trace_args=True, trace_return_value=True)
 def _generate_ride_id():
@@ -65,6 +71,7 @@ def _get_pickup_location(body):
     return body.get('PickupLocation')


+# FIXME: Remove this function.
 @Traceable(trace_args=True, trace_return_value=True)
 def _post_ride_record(ride, url=RIDE_RECORD_URL):
     '''Record ride info'''
@@ -76,6 +83,18 @@ def _post_ride_record(ride, url=RIDE_RECORD_URL):
     return resp


+# FIXME: Publish a ride to SNS
+# - The RIDES_SNS_TOPIC_ARN variables is one you created earlier in the file
+#   by fetching a value from the runtime environment.
+# - Publish a message by calling SNS_CLIENT.publish(). SNS_CLIENT is the name
+#   of the SNS.Client object you created earlier.
+# - Remember to convert the ride variable to JSON by calling json.dumps(ride).
+@Traceable(trace_args=True, trace_return_value=True)
+def _publish_ride_record(ride, sns_topic_arn=RIDES_SNS_TOPIC_ARN):
+    '''Publish ride info to SNS'''
+    pass
+
+
 @thundra
 def handler(event, context):
     '''Function entry'''
@@ -84,6 +103,7 @@ def handler(event, context):
     body = json.loads(event.get('body'))
     pickup_location = _get_pickup_location(body)
     ride_resp = _get_ride(pickup_location)
+    # FIXME: replace _post_ride_record() with _publish_ride_record()
     _post_ride_record(ride_resp)

     resp = {
```
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/handlers/request_ride.py
+++ b/handlers/request_ride.py
@@ -6,6 +6,7 @@ import json
 import os
 import uuid

+import boto3
 from botocore.vendored import requests

 from thundra.thundra_agent import Thundra
@@ -22,7 +23,9 @@ _logger = logging.getLogger(__name__)
 _logger.addHandler(ThundraLogHandler())

 REQUEST_UNICORN_URL = os.environ.get('REQUEST_UNICORN_URL')
-RIDE_RECORD_URL = os.environ.get('RIDE_RECORD_URL')
+
+RIDES_SNS_TOPIC_ARN = os.environ.get('RIDES_SNS_TOPIC_ARN')
+SNS_CLIENT = boto3.client('sns')


 @Traceable(trace_args=True, trace_return_value=True)
@@ -66,15 +69,13 @@ def _get_pickup_location(body):


 @Traceable(trace_args=True, trace_return_value=True)
-def _post_ride_record(ride, url=RIDE_RECORD_URL):
-    '''Record ride info'''
-    resp = requests.post(
-        url,
-        json=ride
+def _publish_ride_record(ride, sns_topic_arn=RIDES_SNS_TOPIC_ARN):
+    '''Publish ride info to SNS'''
+    SNS_CLIENT.publish(
+        TopicArn=sns_topic_arn,
+        Message=json.dumps(ride)
     )

-    return resp
-

 @thundra
 def handler(event, context):
@@ -84,7 +85,7 @@ def handler(event, context):
     body = json.loads(event.get('body'))
     pickup_location = _get_pickup_location(body)
     ride_resp = _get_ride(pickup_location)
-    _post_ride_record(ride_resp)
+    _publish_ride_record(ride_resp)

     resp = {
         'statusCode': 201,
```
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

To do this, remove the *ProvisionedThroughput* configuration on the *RideRecordTable* Cloudfromation resource and add the *BillingMode* key with a value of *PAY_PER_REQUEST*.

<details>
<summary><strong>Hint 1</strong></summary>
<p>

Reference the CloudFormation documentation here:

* [CloudFormation AWS::DynamoDB::Table](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html)

Remove the [*ProvisionedThroughput*](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html#cfn-dynamodb-table-provisionedthroughput) key and add the [*BillingMode*](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html#cfn-dynamodb-table-billingmode) key.
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -65,9 +65,7 @@ resources:
         KeySchema:
           - AttributeName: ${self:custom.ddb_table_hash_key}
             KeyType: HASH
-        ProvisionedThroughput:
-          ReadCapacityUnits: 1
-          WriteCapacityUnits: 1
+        BillingMode: PAY_PER_REQUEST

     ServiceUrlSsmParam:
       Type: "AWS::SSM::Parameter"
```
</p>
</details>


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
<summary><strong>Answer</strong></summary>
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
<summary><strong>Answer</strong></summary>
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
<summary><strong>Answer</strong></summary>
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
<summary><strong>Answer</strong></summary>
<p>

```python
```
</p>
</details>

### 6. Add dead letter queue to _PutRideRecordSns_
Add a dead letter queue for the _PutRideRecordSns_ Lambda function. We need a place to store messages delivered by SNS when if _PutRideRecordSns_ function failed to successfully execute after it has exhausted its retry limit. While we've handled the DynamoDB scaling issue that would most easily trigger this scenario, we should still handle the inevitable unpredictability of the cloud.

We'll use the Serverless Framework plugin called.....

<details>
<summary><strong>Answer</strong></summary>
<p>

```yaml
```
</p>
</details>

### 7. Deploy *wild-rides-ryde-record*
Deploy the updates *wild-rides-ryde-record* service.

```
$ sls deploy -v
```

### 8. Use Artillery to trigger the RequestRide function.
Using Artillery, trigger the *RequestRide* function. This time you should see noe errors.
```
$ artillery run -t <ServiceEndpoint> -c tests/artillery-high-load-config.yml tests/artillery.yml
```
```
All virtual users finished
Summary report @ 00:20:35(+0000) 2018-12-11
  Scenarios launched:  1800
  Scenarios completed: 1800
  Requests completed:  1800
  RPS sent: 8.7
  Request latency:
    min: 818.6
    max: 29396.2
    median: 26876.8
    p95: 29345.7
    p99: 29352.4
  Scenario counts:
    0: 1800 (100%)
  Codes:
    201: 1800
```

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
