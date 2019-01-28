# Serverless Performance

<!-- TASKS:
* deploy
* observe performance issue
* Instrument function
* deploy fix
* add CloudWatch alarms?
-->

<!-- possible extra credit: pricing times out a small percentage of the time. -->

In this workshop we've added additional services to the Wild Rydes application. However some users have started to notice periodic slowdowns in the web experience. We will find these issues and fix them to make out users happy.

## Goals and Objectives

__Objectives:__

* Understand how to find and solve performance issues in a serverless application

Goals:

* Pinpoint application performance issues and resolve them.

## Function instrumentation & tracing

<!-- discuss difference between function tracing and profiling versus distributed tracing -->

## Instructions



### 2. Deploy new services / update existing

#### wild-rydes-pricing


```
$ cd $WORKSHOP
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-pricing.git
$ cd wild-rydes-pricing
$ git checkout -b workshop-operations-02
$ npm install
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


#### wild-rydes-fleet-location

```
$ cd $WORKSHOP
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-fleet-location.git
$ cd wild-rydes-fleet-location
$ git checkout -b workshop-operations-02
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
$ git pull origin workshop-operations-02
$ git checkout -b workshop-operations-02
$ npm install
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


### 1. Request rides from frontend

Navigate to the website frontend in your browser and request rides multiple times. (ex. 10 times.) Notice that some requests take noticeably longer than others.

### 2. Use Artillery to confirm problem

Use [Artillery](https://artillery.io/) to confirm problem

```
$ npm install -g artillery
$ artillery ....
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>

<!-- Maybe make this a Q&A question?
Look at the `Request Latency` of the output. What are the `min`,`max`, `p95`, and `p99` values?
-->

<!-- explain what those values are -->

### 3. Navigate and log in to Thundra

### 4. Use Thundra to find outliers

* Navigate to function
* filter by duration
* observe differences in duration

### 5. Add function tracing to _RequestRide_

https://docs.thundra.io/docs/python-trace-support

Add function tracing to the following Python functions inside the _RequestRide_ Lambda function. We've ignored some functions for brevity because we know they're unlikely to be the source of slow downs.

#### *_get_ride()*

The _handler()_ python function calls this to get ride info. Notably it calls *_get_unicorn()* to get the actual unicorn from the fleet.


<details>
<summary><strong>Answer</strong></summary>
<p>
```python
```
</p>
</details>

#### *_get_unicorn()*
This python function makes a request to the _wild-rydes-ride-fleet_ service for a unicorn. This function is a possible culprit as it traverses the network to make a request via API Gateway that triggers a Lambda function which in turn must query a DynamoDB table

<details>
<summary><strong>Answer</strong></summary>
<p>
```python
```
</p>
</details>

#### *_get_ride_pricing()*

This python function makes a request to the _wild-rydes-pricing_ service for the pricing multiple (surge or discount).  This function is a possible culprit as it traverses the network to make a request via API Gateway that triggers a Lambda function.

<details>
<summary><strong>Answer</strong></summary>
<p>
```python
```
</p>
</details>

#### *handler()*

### 6. Deploy updated wild-rydes service

Deploy the updated wild-rydes service with the _RequestRide_ Lambda function that has tracing enabled.

```
$ cd $WORKSHOP/wild-rydes
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>

### 9. Use Artillery to trigger problem

```
$ artillery ....
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>

### 7. Use Thundra to examine a slow response

* Navigate to function
* sort by duration
* pick recent invocation
* Look at tracing.

Based on the tracing you should see the source of the slowness is the following service.

<details>
<summary><strong>Answer</strong></summary>
<p>
wild-rydes-pricing
</p>
</details>

### 8. Fix slow service

Fix the slowness of the service identified in the previous step.

<details>
<summary><strong>Hint</strong></summary>
<p>
Remove references to *_make_service_randomly_slow()* in *handlers/get_pricing.py*
</p>
</details>

Now deploy the updated service by running *sls deploy -v*

```
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>
</p>
</details>


### 9. Confirm Fix using Artillery

* run artillery
* confirm latest invocations aren't slow.

```
$ artillery ....
```

## Q&A

### 1. Alerting

Q. What alerts should you add?
<!-- show examples -->
