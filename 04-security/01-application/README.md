# Serverless Application Security

In this module we plan to add a new feature to Wild Rydes. To do so, we've created a new service called _wild-rydes-feedback_. However, this new service has a variety of application security issues that require fixing.

We'll work to find and fix these issues.

## Goals and Objectives:

**Objectives:**
* Understand basic security concerns of serverless.

**Goals:**
* Add an API Gateway authorizer
* Secure and manage an API token.
* Find and fix a security vulnerability in a third-party code r
esource

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


<!-- We can split here if we want to -->
### 5. Update wild-rydes to use wild-rydes-ride-fleet API key.
Update wild-rydes to obtain the wild-rydes-ride-fleet API key and an use it when making requests

Secrets management methods
1) **Deploy Time Environmental Variables:** Pass in the value using an environmental variable during deploy time. *Do not use this method except as a last resort. It's better than nothing but the next options are better.*
1) **AWS SSM Parameter Store**: Use [AWS Param Store](https://aws.amazon.com/systems-manager/features/#Parameter_Store) to centrally store the value and fetch it from there. There are two methods of using Param Store. One is via a deploy time value lookup by Serverless Framework and the second is having the _SendToGitter_ function lookup the value in Param Store during execution. When choosing between the two, pay attention to the amount of exposure the key will receive based on the method chosen
1) **AWS Secrets Manager**: Use [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) to manage the token value. While Secrets Manager and Param Store appear similar, compare the feature set and cost of Secrets Manager against Param Store.

If you're unsure what to do, choose AWS SSM Parameter Store.


<!-- Update wild-rydes to use vulnerable version of requests -->
### 6. Insecure Application Dependency

The wild-rydes-feedback has an application dependency vulnerability. Click here to see more about the vulnerability. [![Known Vulnerabilities](https://snyk.io/test/github/ServerlessOpsIO/wild-rydes-feedback/badge.svg)](https://snyk.io/test/github/ServerlessOpsIO/wild-rydes-feedback)

Fix the vulnerability by update the library.

<details>
<summary><strong>Hint</strong></summary>
<p>

Update the _requests_ module i
n _requirements.txt_ from version 2.19.1 to the latest version found here:

* https://pypi.org/project/requests/
</p>
</details>


## Q&A

### 1. Secrets Management

Q. Which Secrets Management method did you choose and why?

Q. Explain the feature differences between Secrets Manager and Param Store.

<details>
<summary><strong>Hint</strong></summary>
<p>

Compare the ability to rotate values using each service.
</p>
</details>

Q. Explain the price differences between Secrets Manager and Param Store
<details>
<summary><strong>Hint</strong></summary>
<p>

See the respective pricing pages:

* [SSM Param Store](https://aws.amazon.com/systems-manager/pricing/)
* [Secrets Manager](https://aws.amazon.com/secrets-manager/pricing/)
<details>
<summary><strong>Hint</strong></summary>
<p>


