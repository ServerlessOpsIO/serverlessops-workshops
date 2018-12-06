# Serverless Failure

<!-- TASKS:
* load test
* observe failures
    * DDB auto-scaling
    * function timeout?
* check logs
* deploy fix
* add CloudWatch alarms?
-->


<!--
This workshop covers serverless reliability and perforamnce.

* Fire artillery at your application
    * Then try and use it
    * it doesn't work because DynamoDB is overloaded.
    * What if I fire and don't tell people?
        * The class is slowly layering on tools to figure out why.

* How else can we make applications fail?

* metrics, monitoring, alerting, logging (CloudWatch)
  * How do our applications fail?
* start breaking applications...

* logging
    * ship structured logs from from functions to S3 for Atehna.

-->

In this module we’ll increase load on the application to trigger a failure.

## Goals and Objectives

__Objectives:__

* Understand how to find failures in our application

Goals:

* Pinpoint application failure that is causing some suers to receive 500 errors.

## Tech Stack

### Wild Rydes

### Tools

#### Thundra

#### Serverless Artillery

## Serverless Failure

* How?
* why?
* Tools?
  * Logging
  * Monitoring

## Instructions

### 1. Run Artillery to generate failure
<!-- FIXME: Need a way to show failures are occurring -->
<!-- Can we run artillery from EC2 and generate faults more easily? -->

```
$ slsart invoke --aws-profile ${AWS_DEFAULT_PROFILE} --stage training-dev -p artillery.yml
```
* Run artillery with heavier load
* Observe non-200 responses


### 2. Deploy new services / update existing

#### wild-rydes-ride-record

```
$ cd $WORKSHOP/
$ rm -rf wild-rydes-ride-record
$ git clone git@github.com:ServerlessOpsIO/wild-rydes-ride-record.git
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

```
$ cd $WORKSHOP/wild-rydes
$ git pull origin workshop-operations
$ git checkout -b workshop-operations-01
$ npm install
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


<!--
### 2. Deploy serverless-artillery
<!-- Maybe we have one commonly deployed one? -->
Get endpoint and save as an environmental variable.  Use `sls info -v` to obtain the value of the _ServiceEndpoint_ stack output.
```
$ sls info -v
```

Add target location to file
```
$ nano artillery.yml
```

```
$ cd tests
$ npm install -g serverless-artillery
$ slsart deploy --aws-profile ${AWS_PROFILE} --stage training-dev -v
```
-->
### 2. Add Thundra support to *wild-rydes-ride-record* *PutRideRecord*

### 3. Run Artillery to generate failure

```
$ slsart invoke --aws-profile ${AWS_DEFAULT_PROFILE} --stage training-dev -p artillery.yml
```
* Run artillery with heavier load
* Observe non-200 responses


### 4. Look at failed invocations in Thundra

### 6. Add logging to *PutRideRecord* in *wild-rydes-ride-record*

### 7. Add DynamoDB auto-scaling to *RideRecord* table.

### 8. Run Artillery to confirm fixes

* Run artillery and see no errors


## Q&A

### 1. DynamoDB auto-scaling

Q. How does DynamoDB auto-scaling work

Q. How does this fail when traffic bursts too quickly?

Q. What is an alternative way to solving the failure?
<!-- Use on-demand DDB provisioning. -->

Q. How might you re-architect your application to better handle this?


### 2 Monitoring

Q. What AWS service can you use for monitoring?

Q. What alarms should be added?
<!-- show how to add CloudWatch alarms. -->
