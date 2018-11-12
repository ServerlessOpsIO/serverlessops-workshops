# Serverless Load Testing

In this module we will add load tests for the [*wild-rydes-ride-fleet GetUnicorn* function](https://github.com/ServerlessOpsIO/wild-rydes/blob/master/handlers/request_ride.py).

## Artillery
We'll be using a tool called [Artillery](https://artillery.io) to simulate request load on the _wild-rydes-ride-fleet_ service.

Because Artillery runs locally and is constrained by the request rate a single host can manage, you might look at [serverless-artillery](https://github.com/Nordstrom/serverless-artillery) for heavier load testing in a CI/CD pipeline.

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
### 2. Examine load testing configuration
Examine the file *tests/load/get_unicorn.yml*.

### 3. Run load tests

Run the load tests.

```
$ artillery run tests/load/get_unicorn.yml
```

## Q&A

### Q. Why are there two separate tests?
Why two tests when testing the API Gateway / Lambda integration exercises the entire integration path?

<details>
<summary><strong>Answer</strong></summary>
<p>

Because more granular testing makes it easier to determine where the integration is failing. A failed API Gateway / Lambda test with a passing Lambda / DynamoDB test indicates the issue is between API Gateway and Lambda.
</p>
</details>


