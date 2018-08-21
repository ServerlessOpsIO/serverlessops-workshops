# Building A New Service

In this module you'll create a new serverless service from start to finish.

## Goals and Objectives:

**Objectives:**
* Apply the lessons of the past two modules to create a new service

**Goals:**
* Build and deploy a new service

## Feature Request
The Wild Rydes application dispatches rides but never records the rides dispatched. You'll create a new service that records rides requested.

...Show new service...

...Show event API...

## Serverless Framework Templates / serverless.yml

Serverless Framework requires a template to initialize a new project. While Serverless Framework has builtin templates, we provide our own which is available on GitHub.

FIXME: Create a temoplate just for this workshop.
[sls-aws-python-36](https://github.com/ServerlessOpsIO/sls-aws-python-36)

The serverless.yml file is composed of 5 primary sections:
* plugins
* custom
* provider
* resources


## Instructions

### 1. Initialize new project

Initialize a new project using our project template.

```
$ sls create -n wild-rides-ride-record -p wild-rides-ride-record -u https://github.com/ServerlessOpsIO/wild-rides-ride-record-template
```

Now change into the newly created project and install the Serverless Framework plugins and initialize the Python virtualenv and install module dependencies.

```
$ cd wild-rides-ride-record
$ npm install
$ npm run setup
```

Now examine the newly created _serverless.yml_ file in your editor.
<details>
<summary><strong>serverless.yml file</strong></summary>
<p>

```yaml
service: wild-rides-ride-record

plugins:
  - serverless-python-requirements


custom:


provider:
  name: aws
  runtime: python3.6
  stage: "${opt:stage, env:SLS_STAGE, 'dev'}"
  profile: "${opt:aws-profile, env:AWS_PROFILE, env:AWS_DEFAULT_PROFILE, 'default'}"
  environment:
    LOG_LEVEL: "${env:LOG_LEVEL, 'INFO'}"


functions:
  PostRideRecord:
    handler: handlers/action.handler
    description: "Create Ride Record In Table"
    memorySize: 128
    timeout: 30

resources:
  Resources:
    RideRecordTable:
      Type: AWS::DynamoDB::Table

  Outputs:
    RideRecordUrl:

```
</p>
</details>

### 2. Write serverless.yml file

#### Add DynamoDB Table
Add a DynamoDB table to the serverless.yml file. Your table should use the RideId of the JSON document accepted by the API endpoint as a unique hash key.

If you're familiar with CloudFormation, you can reference the documentation here:
* [AWS::DynamoDB::Table](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html)

<details>
<summary><strong>Hint</strong></summary>
<p>

```yaml
custom:
  # Add the key below to this section of the file.
  ddb_table_hash_key: 'RideId'

```

```yaml
resources:
  Resources:
    RideRecordTable:
      Type: AWS::DynamoDB::Table
      Properties:
        AttributeDefinitions:
          - AttributeName: ${self:custom.ddb_table_hash_key}
            AttributeType: S
        KeySchema:
          - AttributeName: ${self:custom.ddb_table_hash_key}
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 5
          WriteCapacityUnits: 5
```
</p>
</details>

#### Add Function

Add the configuration for the Lambda function. The function is triggered by an HTTP POST request to the `/record` endpoint. The function will need permission to write to the DynamoDB table and the function will need to know the name of the DynamoDB table.

<details>
<summary><strong>Hint</strong></summary>
<p>

```yaml
provider:
  name: aws
  runtime: python3.6
  stage: "${opt:stage, env:SLS_STAGE, 'dev'}"
  profile: "${opt:aws-profile, env:AWS_PROFILE, env:AWS_DEFAULT_PROFILE, 'default'}"
  environment:
    LOG_LEVEL: "${env:LOG_LEVEL, 'INFO'}"
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:PutItem
      Resource:
        Fn::GetAtt:
          - RideRecordTable
          - Arn

functions:
  PostRideRecord:
    handler: handlers/post_ride_record.handler
    description: "Create Ride Record In Table"
    memorySize: 128
    timeout: 30
    environment:
      DDB_TABLE_NAME:
        Ref: RideRecordTable
    events:
      - http:
          method: POST
          path: /record
```

</p>
</details>

#### Add stack output
Add a CloudFormation stack output so other services can lookup the HTTP endpoint location of this service.

The documentation for CloudFormation stack outputs is here:
* [Cloudformation Outputs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html)

<details>
<summary><strong>Hint</strong></summary>
<p>

```yaml
  Outputs:
    RideRecordUrl:
      Description: "URL of service"
      Value:
        Fn::Join:
          - ""
          - - "https://"
            - Ref: ApiGatewayRestApi
            - ".execute-api."
            - Ref: AWS::Region
            - ".amazonaws.com/${self:custom.stage}"
            - "/record"
      Export:
        Name: "${self:service}-${self:provider.stage}-RideRecordUrl"
```

</p>
</details>

### 3. Write Lambda Function


<details>
<summary><strong>Hint</strong></summary>
<p>

```python
'''Put ride record'''

import json
import logging
import os

import boto3

log_level = os.environ.get('LOG_LEVEL', 'INFO')
logging.root.setLevel(logging.getLevelName(log_level))  # type: ignore
_logger = logging.getLogger(__name__)

# DynamoDB
DDB_TABLE_NAME = os.environ.get('DDB_TABLE_NAME')
DDB_TABLE_HASH_KEY = os.environ.get('DDB_TABLE_HASH_KEY')
dynamodb = boto3.resource('dynamodb')
DDT = dynamodb.Table(DDB_TABLE_NAME)


def _get_body_from_event(event):
    '''Get data from event body'''
    return json.loads(event.get('body'))


def _put_ride_record(ride_record):
    '''Put record item'''
    DDT.put_item(
        TableName=DDB_TABLE_NAME,
        Item=ride_record
    )


def handler(event, context):
    '''Function entry'''
    _logger.debug('Event received: {}'.format(json.dumps(event)))

    ride_record = _get_body_from_event(event)
    _put_ride_record(ride_record)

    resp = {
        'statusCode': 201,
        'body': json.dumps({'success': True})
    }
    _logger.debug('Response: {}'.format(json.dumps(resp)))
    return resp

```
</p>
</details>


### 4. Deploy
Deploy wild-rydes-ride-record.

```
$ sls deploy -v
```

### 5. Test
Use `sls logs` and `sls invoke` to test your new service.

Tail the application logs. Serverless Framework's `logs` command will poll and dump the RequestRide function's logs from CloudWatch to the terminal window.

Start by getting the functions in the application stack using Serverless Framework's `info` command.

```
$ sls info
```


<details>
<summary><strong>Output</strong></summary>
<p>

```
Service Information
service: wild-rydes-ride-record
stage: dev
region: us-east-1
stack: wild-rydes-ride-record-dev
api keys:
  None
endpoints:
  POST - https://wrqjqpc28d.execute-api.us-east-1.amazonaws.com/dev/record
functions:
  PostRideRecord: wild-rydes-ride-record-dev-PostRideRecord

Stack Outputs
PostRideRecordLambdaFunctionQualifiedArn: arn:aws:lambda:us-east-1:144121712529:function:wild-rydes-ride-record-dev-PostRideRecord:7
RideRecordUrl: https://wrqjqpc28d.execute-api.us-east-1.amazonaws.com/dev/record
ServiceEndpoint: https://wrqjqpc28d.execute-api.us-east-1.amazonaws.com/dev
ServerlessDeploymentBucketName: wild-rydes-ride-record-d-serverlessdeploymentbuck-1jc157gnh1ebi
```
</p>
</details>

### 6. Update wild-rydes-ride-request service.

<details>
<summary><strong>Hint</strong></summary>
<p>

```yaml
```

</p>
</details>


## Questions

### 1. Serverless Framework Plugins

### 2. Parts of the serverless.yml file

Q. What are the different sections of the file for?

* plugins
* custom
* provider
* resources


