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

#### wild-rydes-api-authorizor
Deploy the new *wild-rydes-data-lake* service.

```
$ cd $WORKSHOP
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-data-lake.git
$ cd wild-rydes-data-lake
$ git checkout -b workshop-security-01
$ npm install
$ sls deploy -v
```

#### wild-rydes-ride-record
```
$ cd $WORKSHOP/wild-rydes-ride-record
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-ride-record.git
$ git checkout -b workshop-security-01
$ npm install
$ sls deploy -v
```

### 2. Obtain API key for wild-rydes-ride-fleet


### 3. Update wild-rydes-ride-fleet
Update API Gateway to to call the API key service.


### 4. Request a member of fleet using `curl`
Attempt to use curl to request a ride.

```
$ curl ...
```

You should receive an access denied error.


Now request using a token.
```
$ curl
```

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


