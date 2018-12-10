# Serverless Failure

<!--
Maybe this becomes metrics and monitoring?
-->

<!--
Every instruction should follow
* what to do
* why to do it
* how to do it.
-->

In this module we’ll increase load on the application to trigger a failure somewhere in the application.

## Goals and Objectives

__Objectives:__

* Understand how to find failures and performance issues in our application.
* Understand how to instrument a function to obtain metrics, logs, and trace data.

Goals:

* Pinpoint application failure that is causing some suers to receive 500 errors.
* Add the Thundra agent to Wyld Rydes applications.

## Tech Stack

### Wild Rydes

This workshop module will involve the following Wild Rydes services.
 * [wild-rydes-ride-record](https://github.com/ServerlessOpsIO/wild-rydes-ride-record): Service for recording rides requested.
 * [wild-rydes](https://github.com/ServerlessOpsIO/wild-rydes): Frontend website and ride request service.

*Note: if you've done the [Serverless Up and Running workshop](../../01-up-and-running/02-build-new-service) we'll be replacing what you've deployed with versions meant for this workshop. Without replacing your version with the version for this workshop certain steps may not succeed.*

### Tools

#### Thundra

#### Artillery

## Serverless Failure

* How?
* why?
* Tools?
  * Logging
  * Monitoring

## Instructions

### 1. Deploy new services / update existing
Deploy updates to wild-rydes services. These updates have changes specifically targeted for the completion of this workshop.

#### wild-rydes-ride-record
Deploy the *wild-rydes-ride-record* service. We'll be working on issues stemming from this service.

Start by cloning the repository from GitHub, then check out the *workshop-operations-01* branch for this workshop module, and finally deploy the application.

```
$ cd $WORKSHOP/
$ rm -rf wild-rydes-ride-record
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-ride-record.git
$ git checkout -b workshop-operations-01
$ npm install
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


#### wild-rydes
Deploy the *wild-rydes* service. It has updates to work with the *wild-rydes-ride-record* service. In later steps you'll be instrumenting this function to collect runtime and log data from invocations to the Thundra platform.

Start by cloning the repository from GitHub, then check out the *workshop-operations-01* branch for this workshop module, and finally deploy the application.
```
$ cd $WORKSHOP/wild-rydes
$ git pull origin workshop-operations-01
$ git checkout -b workshop-operations-01
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


### 2. Install Artillery
Install the Artillery utility. You'll run this to generate load on your application.

```
$ npm install -g artillery --unsafe-perm=true
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


### 3. Run Artillery
Run the Artillery tool against the web endoint for *RequestRide* in *wild-rydes*. This will trigger failures to investigate.

Start by getting the value of *ServiceEndpoint* from the *wild-rydes* stack outputs. Use `sls info -v` to obtain the value. You'll need this value so you know what API endpoint to target with Artillery.
```
$ cd $WORKSHOP/wild-rydes
$ sls info -v
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>

Once you have the *ServiceEndpoint* value, generate load on the *RequestRide* function. The configuration file for Artillery will run the command for 3 minutes.

```
$ artillery run -t <ServiceEndpoint> -c tests/artillery-high-load-config.yml tests/artillery.yml
```

In the Output under `Codes` section of the final report you should see HTTP 5XX codes

```
All virtual users finished
Summary report @ 00:51:39(+0000) 2018-12-07
  Scenarios launched:  1800
  Scenarios completed: 1800
  Requests completed:  1800
  RPS sent: 8.66
  Request latency:
    min: 874.4
    max: 29500.8
    median: 1269.2
    p95: 29360.8
    p99: 29365.6
  Scenario counts:
    0: 1800 (100%)
  Codes:
    201: 1492
    502: 106
    504: 202
```

Error codes:
* 502: Function returned bad response. Typically the Lambda function errored our.
<!-- FIXME: Perhaps this the problem to be solved for module 2 -->
* 504: API Gateway timed out. API Gateway has a 29 second limit for any integrations (Lambda functions, authorizers, etc.) to run. <!-- This error can occur for any number of reasons but in this workshop it's probably the combination of a Lambda cold start and the length of time spent attempting to send data to the *wild-rydes-ride-record* service. -->

*NOTE: You might only get a single error code. The issue we're triggering can trigger both HTTP error codes as a symptom.*

<!-- FIXME: We should maybe look at min/max/median/p95/p99? Maybe wait until Thundra has dashboards. -->

### 4. Add Thundra support to *wild-rydes* *RequestRide*
<!-- Start collecting traces and logs -->
Start collecting metrics and logs from *RequestRide* in *wild-rydes*. We'll instrument this function by using the [Thundra Python agent](https://github.com/thundra-io/thundra-lambda-agent-python). (The *PutRideRecord* function in *wild-rydes-ride-record* has already been done for you.) By instrumenting the function we'll allow the Thundra platform to start collecting invocation metrics, logs, and trace the length of time spent performing different actions in the code.

#### Add Thundra module and initialize it in the *RequestRide* function.
<!-- Use automated setup when it works.-->
Add the Thundra Python module to wild-rydes. By adding the module we allow Thundra to start collecting basic invocation metrics such as duration, memory usage, and CPU usage.

<!-- install Python module -->
Start by installing the Python module for Thundra.
```
$ cd $WORKSHOP/wild-rydes
$ echo thundra >> requirements.txt
$ pip install -r requirements.txt
```
<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


##### In *serverless.yml*
<!-- Add API key to serverless.yml -->
Add the Thundra API key to *serverless.yml*. This API key is necessary for the Thundra agent to ship to the platform. You'll fetch the key value from AWS SSM Parameter Store where we've centrally stored it and then configure it be an environmental variable value in the Lambda function's runtime environment.

In the *serverless.yml* file create the key *custom.thundraApiKey*. (You're creating a key called *thundraApiKey* under the *custom* key in the file.) The API key's value is stored in AWS SSM Parameter Store under the name `/thundra/root/api-key`. Use Serverless Framework's ability to [lookup values in SSM Parameter Store](https://serverless.com/framework/docs/providers/aws/guide/variables/#reference-variables-using-the-ssm-parameter-store) to populate the key value. *You don't need the key value it self, you just need to implement the ability to look it up.*

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -11,6 +11,8 @@ custom:
   region: "${opt:region, 'us-east-2'}"
   log_level: "${env:LOG_LEVEL, 'INFO'}"

+  thundraApiKey: "${ssm:/thundra/root/api-key}"
+
   request_unicorn_url: "${cf:wild-rydes-ride-fleet-${self:custom.stage}.RequestUnicornUrl}"
   ride_record_url: "${ssm:/wild-rydes-ride-record/training-dev/URL}"

```
</p>
</details>

<!-- pass API key to function environment -->
Next, create create an environmental variable key called `THUNDRA_API_KEY`. Its value will come from [referencing the *custom.thundraApiKey* key](https://serverless.com/framework/docs/providers/aws/guide/variables/#reference-properties-in-serverlessyml) in the previous step.

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -37,6 +39,7 @@ provider:
   cfnRole: "arn:aws:iam::${env:AWS_ACCOUNT}:role/CloudFormationDeployRole"
   environment:
     LOG_LEVEL: ${self:custom.log_level}
+    THUNDRA_API_KEY: ${self:custom.thundraApiKey}
   stackTags:
     serverless:service: ${self:service}
   iamRoleStatements:
```
</p>
</details>

##### In *handlers/request_ride.py*
Import the *thundra* Python module in handlers/request_ride.py. Then, initialize it using the `THUNDRA_API_KEY` value obtained from the runtime environment. Finally, add the Python function decorator to the *handler()* function in the file. See the file diff below for what needs to be added.

```diff
--- a/handlers/request_ride.py
+++ b/handlers/request_ride.py
@@ -8,6 +8,11 @@

 from botocore.vendored import requests

+from thundra.thundra_agent import Thundra
+THUNDRA_API_KEY = os.environ.get('THUNDRA_API_KEY', '')
+thundra = Thundra(api_key=THUNDRA_API_KEY)
+
+
 log_level = os.environ.get('LOG_LEVEL', 'INFO')
 logging.root.setLevel(logging.getLevelName(log_level))  # type:ignore
 _logger = logging.getLogger(__name__)
@@ -61,6 +66,7 @@ def _post_ride_record(ride, url=RIDE_RECORD_URL):
     return resp


+@thundra
 def handler(event, context):
     '''Function entry'''
     _logger.info('Request: {}'.format(json.dumps(event)))
```

Your Lambda function now has the Thundra agent intregrated. This will collect basic metrics like invocation duration and consumed memory. You'll need to do more work to add logging and tracing.

#### Add logging to *RequestRide* in *wild-rydes*
<!-- Start capturing logs so you can see both failures. -->
Add the Thundra logging handler. Anytime we use Python's logging facility those messages will be relayed to Thundra. See the file diff below for what needs to be added.

```diff
--- a/handlers/request_ride.py
+++ b/handlers/request_ride.py
@@ -13,9 +13,11 @@
 thundra = Thundra(api_key=THUNDRA_API_KEY)


+from thundra.plugins.log.thundra_log_handler import ThundraLogHandler
 log_level = os.environ.get('LOG_LEVEL', 'INFO')
 logging.root.setLevel(logging.getLevelName(log_level))  # type:ignore
 _logger = logging.getLogger(__name__)
+_logger.addHandler(ThundraLogHandler())

 REQUEST_UNICORN_URL = os.environ.get('REQUEST_UNICORN_URL')
 RIDE_RECORD_URL = os.environ.get('RIDE_RECORD_URL')
 ```

#### Add tracing to *RequestRide* in wild-rydes
<!-- Is there a performance impact? If so, how much? -->
Add tracing to the Lambda function. Thundra will record the time spend in each function which will allow us to find bottlenecks. See the file diff below for what needs to be added.

<!-- import module -->
<!-- Add tracing support -->
```diff
--- a/handlers/request_ride.py
+++ b/handlers/request_ride.py
@@ -9,6 +9,8 @@
 from botocore.vendored import requests

 from thundra.thundra_agent import Thundra
+from thundra.plugins.trace.traceable import Traceable
+
 THUNDRA_API_KEY = os.environ.get('THUNDRA_API_KEY', '')
 thundra = Thundra(api_key=THUNDRA_API_KEY)

@@ -23,11 +25,13 @@
 RIDE_RECORD_URL = os.environ.get('RIDE_RECORD_URL')


+@Traceable(trace_args=True, trace_return_value=True)
 def _generate_ride_id():
     '''Generate a ride ID.'''
     return uuid.uuid1()


+@Traceable(trace_args=True, trace_return_value=True)
 def _get_ride(pickup_location):
     '''Get a ride.'''
     ride_id = _generate_ride_id()
@@ -42,22 +46,26 @@ def _get_ride(pickup_location):
     return resp


+@Traceable(trace_args=True, trace_return_value=True)
 def _get_timestamp_from_uuid(u):
     '''Return a timestamp from the given UUID'''
     return datetime.fromtimestamp((u.time - 0x01b21dd213814000) * 100 / 1e9)


+@Traceable(trace_args=True, trace_return_value=True)
 def _get_unicorn(url=REQUEST_UNICORN_URL):
     '''Return a unicorn from the fleet'''
     unicorn = requests.get(REQUEST_UNICORN_URL)
     return unicorn.json()


+@Traceable(trace_args=True, trace_return_value=True)
 def _get_pickup_location(body):
     '''Return pickup location from event'''
     return body.get('PickupLocation')


+@Traceable(trace_args=True, trace_return_value=True)
 def _post_ride_record(ride, url=RIDE_RECORD_URL):
     '''Record ride info'''
     resp = requests.post(
```


### 5. Deploy updated *wild-rydes*
Redeploy the updated *wild-rydes* service. When we use the application services now, data will be shipped to Thundra and we can use that to find the cause of the error messages we've incurred.

```
$ cd $WORKSHOP/wild-rydes
$ sls deploy -v
```

### 6. Run Artillery to generate failure
Run Artillery again to generate load on the Wild Rydes application. You should see similar numbers of errors in your output as you did before but Thundra will now have data for us.

```
$ artillery run -t <ServiceEndpoint> -c tests/artillery-high-load-config.yml tests/artillery.yml
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


### 7. Look at failed invocations in Thundra
<!-- we have two issues: 1) bad response from wild-rydes-ride-record, 2) retries to wild-rydes-ride-record cause RequestRide to timeout -->

<!-- login -->
Login to Thundra
* https://console.thundra.io/landing

<!-- Navigate to functions -->


<!-- Select RequestRide; filter for errors; investigate one of them -->


<!-- see timeout -->
<!-- function marked with X -->

<!-- Select PutRideRecord; filter for errors; investigate one of them -->
<!-- see error -->



## Q&A

### 2. Monitoring

Q. What AWS service can you use for monitoring?

Q. What alarms should be added?
<!-- show how to add CloudWatch alarms. -->

Q. In this workshop we manually instrumented our function to collect data from it. What is an alternative method of instrumenting functions?

<details>
<summary><strong>Hint 1</strong></summary>
<p>

AWS lets you add additional code to the Lambda runtime environment through a feature called layers.
* [AWS Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)
</p>
</details>

<details>
<summary><strong>Hint 2</strong></summary>
<p>

Several services in the AWS serverless space now offer AWS Layers support
* [Thundra Lambda Layers and Custom Runtime Support](https://blog.thundra.io/thundra-lambda-layers-and-custom-runtime-support)
</p>
</details>

<details>
<summary><strong>Answer</strong></summary>
<p>

Several companies have started to offer automated instrumentation or monitoring by having you add a layer for the company's product to your Lambda function.
</p>
</details>


