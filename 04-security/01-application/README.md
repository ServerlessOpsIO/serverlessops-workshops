# Serverless Application Security

In this module we plan to add a new feature to Wild Rydes. To do so, we've created a new service called _wild-rydes-feedback_. However, this new service has a variety of application security issues that require fixing.

We'll work to find and fix these issues.

## Goals and Objectives:

**Objectives:**
* Understand basic security concerns of serverless.

**Goals:**
* Add an API Gateway authorizer
* Secure and manage an API token.
* Find and fix a security vulnerability in a third-party code resource

## Application Security

### API Gateway Authorizers


### Vulnerable Function Dependencies



### Managing Secrets

#### AWS Systems Manager Param Store


#### AWS Secrets Manager


## Instructions

### 1. Deploy services

#### wild-rydes-api-auth
Deploy the new *wild-rydes-api-auth* service.

```
$ cd $WORKSHOP
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-api-auth.git
$ cd wild-rydes-data-lake
$ git checkout -b workshop-security-01
$ npm install
$ sls deploy -v
```

#### wild-rydes-ride-fleet
```
$ cd $WORKSHOP/wild-rydes-ride-fleet
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-ride-fleet.git
$ git checkout -b workshop-security-01
$ npm install
$ sls deploy -v
```

### 2. Update wild-rydes-ride-fleet
Update _wild-rydes-ride-fleet_ to to call the *wild-rydes-api-auth* *Authorizer* function when invoking the _RequestUnicorn_ Lambda function. To do this you’ll update the _RequestUnicorn_ function’s HTTP event.

You’ll do this by adding an _authorizer_ property to the HTTP trigger event that defines a function ARN and header to pull the API key from. You can just make the change below.

*handlers/get_unicorn.py*
```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -51,6 +51,9 @@ functions:
       - http:
           path: /unicorn
           method: get
+          authorizer:
+            arn: ${ssm:/wild-rydes-api-auth/${self:provider.stage}/Arn}
+            identitySource: method.request.header.x-api-key

   LoadTable:
     handler: handlers/load_table.handler
```

There’s a bit of variability you can achieve with authorizers and the Serverless Framework documentation covers this here:

* [Serverless Framework: HTTP Endpoints with Custom Authorizers](https://serverless.com/framework/docs/providers/aws/events/apigateway#http-endpoints-with-custom-authorizers)

With your function updated, deploy it.

```
$ sls deploy
```
### 3. Obtain API key for _wild-rydes-ride-fleet_
Generate an API key for _wild-rydes-ride-fleet_ in the _wild-rydes-api-auth_ service.

Start by getting the HTTP endpoint for creating API keys. Do this by running `sls info` in the _wild-rydes-api-auth_ directory and in the output under _endpoints_ look for the endpoint that takes an HTTP POST request. (It should be the first endpoint listed.)

```
$ cd $WORKSHOP/wild-rydes-api-auth
$ sls info
```
<details>
<summary><strong>Output</strong></summary>
<p>

```
Service Information
service: wild-rydes-api-auth
stage: user0
region: us-east-2
stack: wild-rydes-api-auth-user-0
api keys:
  None
endpoints:
  POST - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key
  PUT - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}
  GET - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}
  GET - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}/{timestamp}
  PUT - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}/{timestamp}
  PUT - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}/{timestamp}/active
  PUT - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}/{timestamp}/inactive
  PUT - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}/{timestamp}/ttl
  DELETE - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}
  DELETE - https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/user0/key/{id}/{timestamp}
functions:
  Authorizer: wild-rydes-api-auth-user0-Authorizer
  CreateKey: wild-rydes-api-auth-user0-CreateKey
  GetKey: wild-rydes-api-auth-user0-GetKey
  UpdateKey: wild-rydes-api-auth-user0-UpdateKey
  DeleteKey: wild-rydes-api-auth-user0-DeleteKey
layers:
  None
```

The value you’re looking for is `https://kbnrvuvmbl.execute-api.us-east-2.amazonaws.com/<USER#>/key`
</p>
</details>

Now make a request to the _wild-rydes-api-auth_ service to generate an API key for _wild-rydes-ride-fleet_. The payload of the request is JSON with the name of the API Gateway for your _wild-rydes-ride-fleet_ service (remember to replace `<USER#>` with your own user number / stage name) and the URL is the URL you just obtained from running `sls info`.
```
curl -X POST --data '{"Id":"<USER#>-wild-rydes-ride-fleet"}' <URL>
```
<details>
<summary><strong>Output</strong></summary>
<p>

```
{
  "Id": "<USER#>-wild-rydes-ride-fleet",
  "Key": "<API_KEY>",
  "DateTime": 1546041232,
  "Active": true,
  "TTL": 0
}
```
</p>
</details>

### 4. Request a member of fleet using `curl`
Attempt to use curl to request a ride.

Start by getting an HTTP endpoint for requesting a ride from _wild-rydes-ride-fleet_. Do this by running `sls info` in the _wild-rydes-ride-fleet_ directory. There will only be one endpoint in the _endpoints_ section of the output.

```
$ cd $WORKSHOP/wild-rydes-ride-fleet
$ sls info
```
<details>
<summary><strong>Output</strong></summary>
<p>

```
Service Information
service: wild-rydes-ride-fleet
stage: user0
region: us-east-2
stack: wild-rydes-ride-fleet-user0
api keys:
  None
endpoints:
  GET - https://hhrs2nyo0b.execute-api.us-east-2.amazonaws.com/user0/unicorn
functions:
  RequestUnicorn: wild-rydes-ride-fleet-user0-RequestUnicorn
  LoadTable: wild-rydes-ride-fleet-user0-LoadTable
layers:
  None
```

</p>
</details>

Now make a request to the _wild-rydes-ride-fleet_ service using `curl` with no API key. You should get an access denied error.
```
$ curl <URL>
```
<details>
<summary><strong>Output</strong></summary>
<p>

```
{
  "message": "Unauthorized"
}
```
</p>
</details>


Now make the same request but with the API key you created in step 3. This time your request should succeed.
```
$ curl -H "x-api-key: <API_KEY>" <URL>
```
<details>
<summary><strong>Output</strong></summary>
<p>

```
{
  "DeviceId": "212dee58-f26a-11e8-b171-8c859074f8c7",
  "Name": "Bucephalus",
  "Color": "Golden"
}
```
</p>
</details>


### 5. Securely store API key in SSM Parameter Store


```
aws ssm put-parameter --name /wild-rydes-ride-fleet/<USER#>/api_key --value <API_KEY> --type SecureString --key-id alias/serverlessops/master
```

<details>
<summary><strong>Output</strong></summary>
<p>

```
{
    "Version": 1
}
```
</p>
</details>


<!-- We can split here if we want to -->
### 6. Update *wild-rydes* to use *wild-rydes-ride-fleet* API key.
<!-- FIXME: May not be possioble to encrypt variable in an automated way. -->
Update *wild-rydes* to obtain the *wild-rydes-ride-fleet* API key and an use it when making requests for fleet members. You'll pass the SSM Parameter name, NOT THE PARAMETER VALUE, to the function via an environmental variable. Your function will then fetch the parameter's value. This is different than what we've done before! We want to keep the API key securely encrypted until the function is invoked.

#### Update _serverless.yml_
Update the *serverless.yml* file.

*serverless.yml*
```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -11,9 +11,13 @@ custom:
   region: "${opt:region, 'us-east-2'}"
   log_level: "${env:LOG_LEVEL, 'INFO'}"

+  ssm_encryption_key_arn: "${ssm:/infra/kms/prime/alias/serverlessops/master/Arn}"
+
   thundraApiKey: "${ssm:/thundra/root/api-key}"

   request_unicorn_url: "${cf:wild-rydes-ride-fleet-${self:custom.stage}.RequestUnicornUrl}"
+  request_unicorn_api_key_ssm_param: "/wild-rydes-ride-fleet/${self:custom.stage}/api_key"
+
   ride_record_url: "${ssm:/wild-rydes-ride-record/${self:custom.stage}/URL}"

   hostedZoneName: "${ssm:/route53/root/ServerlessOpsDomain}"
@@ -61,6 +65,21 @@ provider:
         - "sns:Publish"
       Resource:
         - Ref: RidesSnsTopic
+    - Effect: "Allow"
+      Action:
+        - "ssm:GetParameter"
+      Resource:
+        - Fn::Join:
+          - ":"
+          - - "arn:aws:ssm"
+            - Ref: AWS::Region
+            - Ref: AWS::AccountId
+            - "parameter${self:custom.request_unicorn_api_key_ssm_param}"
+    - Effect: "Allow"
+      Action:
+        - "kms:Decrypt"
+      Resource:
+        - "${self:custom.ssm_encryption_key_arn}"


 functions:
@@ -72,6 +91,7 @@ functions:
     environment:
       LOG_LEVEL: "${self:custom.log_level}"
       REQUEST_UNICORN_URL: "${self:custom.request_unicorn_url}"
+      REQUEST_UNICORN_API_KEY_SSM_PARAM: "${self:custom.request_unicorn_api_key_ssm_param}"
       RIDES_SNS_TOPIC_ARN:
         Ref: RidesSnsTopic
     events:
```

#### Update the function handler
*handlers/request_ride.py*
```diff
--- a/handlers/request_ride.py
+++ b/handlers/request_ride.py
@@ -24,6 +24,11 @@ _logger.addHandler(ThundraLogHandler())

 REQUEST_UNICORN_URL = os.environ.get('REQUEST_UNICORN_URL')

+SSM_CLIENT = boto3.client('ssm')
+REQUEST_UNICORN_API_KEY_SSM_PARAM = os.environ.get('REQUEST_UNICORN_API_KEY_SSM_PARAM')
+REQUEST_UNICORN_API_KEY = SSM_CLIENT.get_parameter(Name=REQUEST_UNICORN_API_KEY_SSM_PARAM, WithDecryption=True)['Parameter']['Value']
+
+
 RIDES_SNS_TOPIC_ARN = os.environ.get('RIDES_SNS_TOPIC_ARN')
 SNS_CLIENT = boto3.client('sns')

@@ -56,9 +61,14 @@ def _get_timestamp_from_uuid(u):


 @Traceable(trace_args=True, trace_return_value=True)
-def _get_unicorn(url=REQUEST_UNICORN_URL):
+def _get_unicorn(url=REQUEST_UNICORN_URL, api_key=REQUEST_UNICORN_API_KEY):
     '''Return a unicorn from the fleet'''
-    unicorn = requests.get(REQUEST_UNICORN_URL)
+    unicorn = requests.get(
+        REQUEST_UNICORN_URL,
+        headers={
+            'x-api-key': REQUEST_UNICORN_API_KEY
+        }
+    )
     return unicorn.json()


```


<!-- Update wild-rydes to use vulnerable version of requests -->
### 7. Insecure Application Dependency

The wild-rydes service has an application dependency vulnerability. We've been alerted of this issue through [Snyk](https://www.snyk.io) which provides monitoring for application dependency vulnerabilities. Click the link below to be brought to the wild-rydes GitHub repository. (This link brings you to a branch of the repository with the Snyk badge added.)

* https://github.com/ServerlessOpsIO/wild-rydes/tree/workshop-security-02

Now look for and click the [![Known Vulnerabilities](https://snyk.io/test/github/ServerlessOpsIO/wild-rydes/badge.svg?targetFile=requirements.txt)](https://snyk.io/test/github/ServerlessOpsIO/wild-rydes?targetFile=requirements.txt) button to be taken to the vulnerability explanation page. You'll see this issue can leak authentication credentials. (This vulnerability does not affect our current usage though.) Click the *More about this issue* link. The next page explains under the *Remediation* heading that the issue was resolved in release 2.20. Update the requests module in the *requetsts.txt* from version *2.19* to *2.20*.


```diff
--- a/requirements.txt
+++ b/requirements.txt
@@ -1,4 +1,4 @@
 boto3
 cfn_resource
 thundra
-requests==2.19
+requests==2.20
```

## Q&A

### 1. Secrets Management
Q. Our means of passing the API key to the *RequestRide* function is fine except it makes rotating the API key hard. Explain why.

Q. What are some alternative ways of securely passing the API key to *RequestRide* that alleviate the problems with rotating the key value.

<!-- FIXME: This question no longer makes sense since we just use param store. -->
Q. Which Secrets Management method did you choose and why?

**Q. EXTRA CREDIT:** Perform the API key lookup at function run time instead of at deploy time. Use (ssm-cache-python)[https://github.com/alexcasalboni/ssm-cache-python] for lookups and caching of responses to reduce function invocation time and prevent hitting Parameter Store API limits.

### 2. Custom authorizers

Q. **EXTRA CREDIT:** Add API Gateway caching so less function invocations and DynamoDB lookups are made.
