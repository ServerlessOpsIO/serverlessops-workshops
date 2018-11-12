<!--
We might reuse an existing service that has more features so that we can introduce brokenness.
-->

# Serverless Integration Testing

In this module we will add integration tests to the [*wild-rydes-ride-fleet GetUnicorn* function](https://github.com/ServerlessOpsIO/wild-rydes/blob/master/handlers/request_ride.py).

Take note, while the integration tests are executed locally, you will be testing your application stack deployed in AWS. _Be sure to have wild-rydes-ride-fleet_ already deployed.

## Objectives and Goals

__Objectives:__
* Understand how to implement integration tests for a serverless function.

__Goal:__
* Create integration tests for the _GetUnicorn_ function

## Instructions

### 1. Make sure you are in wild-rydes-ride-fleet
In this module we'll continue to use the _wild-rydes-ride-fleet_ service from the previous module.

```
$ cd $WORKSHOP/wild-rydes-ride-fleet
```
### 2. Examine integration test template
Examine the file *tests/integration/handlers/get_unicorn.py*.

### 3. Add integration tests

Add integration tests for the Python _handler()_ function in the Lambda function _GetUnicorn_. Your integration tests should ensure API Gateway, AWS Lambda, and DynamoDB function properly together.


#### Test Lambda / DynamoDB Integration
Write a test that invokes the _GetUnicorn_ Lambda function The prupose of this test is to ensure that the function has the appropriate IAM permissions to successfully do its job.

...Explain what to add...

```
$ pytest -v tests/integration/handlers/get_unicorn.py
```

Your integration test should execute but fail with the following error.
```
...error...
```

Fix the error by fixing the IAM permissions.

<details>
<summary><strong>Answer</strong></summary>
<p>

</p>
</details>


#### Test API Gateway / Lambda Integration
Write a test that sends a GET request to your API endpoint and checks for a successful response.

...Explain what to add...

Now run the unit test and ensure it passes.

```
$ pytest -v tests/integration/handlers/get_unicorn.py
```

Your integration test should execute but fail with the following error.
```
...error...
```

Fix the error by fixing...

<details>
<summary><strong>Answer</strong></summary>
<p>

</p>
</details>


## Q&A

### Q. Why are there two separate tests?
Why two tests when testing the API Gateway / Lambda integration exercises the entire integration path?

<details>
<summary><strong>Answer</strong></summary>
<p>

Because more granular testing makes it easier to determine where the integration is failing. A failed API Gateway / Lambda test with a passing Lambda / DynamoDB test indicates the issue is between API Gateway and Lambda.
</p>
</details>

