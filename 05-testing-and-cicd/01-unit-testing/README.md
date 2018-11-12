# Serverless Unit Testing

In this module we will add unit tests to the [*wild-rydes-ride-fleet GetUnicorn* function](https://github.com/ServerlessOpsIO/wild-rydes/blob/master/handlers/request_ride.py).

## Objectives and Goals

__Objectives:__
* Understand how to implement unit tests for a serverless function.

__Goal:__
* Create unit tests for the _GetUnicorn_ function

## Moto: AWS Mocking
The Python Moto module will be used to mock out the AWS DynamoDB service

## Instructions

### 1. Checkout testing branch of wild-rydes-ride-fleet

Checkout the _testing_ branch of _wild-rydes-ride-fleet_. This branch has a basic unit testing framework setup.

```
$ cd $WORKSHOP/wild-rydes-ride-fleet
$ git checkout testing
```
<details>
<summary><strong>Output</strong></summary>
<p>

```
$ cd $WORKSHOP/wild-rydes-ride-fleet
$ git checkout testing
Switched to branch 'testing'
```
</p>
</details>

### 2. Examine unit test template
Examine the file *tests/unit/handlers/get_unicorn.py*.

### 3. Create a test event
To properly test this service you will need a test event. The wild-rydes-ride-fleet service takes an HTTP GET request to API Gateway.

<details>
<summary><strong>Hint 1</strong></summary>
<p>
* [Sample Events Published by Event Sources](https://docs.aws.amazon.com/lambda/latest/dg/eventsources.html)
</p>
</details>

<details>
<summary><strong>Hint 2</strong></summary>
<p>
* [API Gateway Proxy Request Event](https://docs.aws.amazon.com/lambda/latest/dg/eventsources.html#eventsources-api-gateway-request)
</p>
</details>

<!-- FIXME: Obtain real event -->
<details>
<summary><strong>Answer</strong></summary>
<p>
```json
```
</p>
</details>

### 4. Add unit tests

Add unit tests for the Python _handler()_ function in the Lambda function _GetUnicorn_. We'll also add unit tests for the Python functions *_get_unicorn()* which is called in the _handler()_ function.

* [handlers/get_unicorn.py](https://github.com/ServerlessOpsIO/wild-rydes-ride-fleet/blob/master/handlers/get_unicorn.py#L34-L46)
```python
def handler(event, context):
    '''Function entry'''
    _logger.debug('Request: {}'.format(json.dumps(event)))

    resp = _get_unicorn()

    resp = {
        'statusCode': 201,
        'body': json.dumps(resp),
    }

    _logger.debug('Response: {}'.format(json.dumps(resp)))
    return resp

```

#### Create unit test for *_get_unicorn()*
The *_get_unicorn()* function takes no arguments and returns a random unicorn from a DynamoDB table.

...Explain what to add...

Now run the unit test and ensure it passes.

```
$ pytest -v tests/unit/handlers/get_unicorn.py
```

#### Create unit test for *handler()*
The *handler()* function takes an API Gateway event in the form of a JSON doc and a Lambda context object. The function returns a JSON doc with a status for API Gateway and a body that contains the information of a unicorn.

...Explain what to add...

Now run the unit test and ensure it passes.

```
$ pytest -v tests/unit/handlers/get_unicorn.py
```

## Q&A

### 1. Event Generation
Q. How might you obtain an event to test with?
<details>
<summary><strong>Hint</strong></summary>
<p>

Investigate CloudWatch logging.
</p>
</details>

<details>
<summary><strong>answer</strong></summary>
<p>

Deploy the function with debug level logging enabled. The function will then log the event received to CloudWatch.

```python
def handler(event, context):
    '''Function entry'''
    _logger.debug('Request: {}'.format(json.dumps(event)))
```

You can then obtain the event in your CloudWatch logs.
</p>
</details>

