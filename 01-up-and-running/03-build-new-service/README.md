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

[sls-aws-python-36](https://github.com/ServerlessOpsIO/sls-aws-python-36)

The template will create a serverless.yml file that looks as follows:

```yaml
service: <NAME>

plugins:
  - serverless-python-requirements


custom:
  stage: "${opt:stage, env:SLS_STAGE, 'dev'}"
  profile: "${opt:aws-profile, env:AWS_PROFILE, env:AWS_DEFAULT_PROFILE, 'default'}"
  log_level: "${env:LOG_LEVEL, 'INFO'}"

  pythonRequirements:
    dockerizePip: false


provider:
  name: aws
  runtime: python3.6
  stage: ${self:custom.stage}
  profile: ${self:custom.profile}
  environment:
    LOG_LEVEL: ${self:custom.log_level}
  stackTags:
    x-service: ${self:service}
    x-stack: ${self:service}-${self:provider.stage}


functions:
  Action:
    handler: handlers/action.handler
    description: "<DESCRIPTION>"
    memorySize: 128
    timeout: 30

```

This file is....

## Instructions

### 1. Initialize new project

Initialize a new project using our project template.

```
$ sls create -n wild-rides-ride-record -p wild-rides-ride-record -u https://github.com/ServerlessOpsIO/sls-aws-python-36
$ cd wild-rides-ride-record
$ npm install
$ npm run setup
```

### 2. Write serverless.yml file

#### Add DynamoDB Table
Add a DynamoDB table to the serverless.yml file.

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

#### Add stack output

### 3. Write Lambda Function

### 4. Deploy

### 5. Test

### 6. Update wild-rydes-ride-request service.

## Questions

### 1. Serverless Framework Plugins

### 2. Parts of the serverless.yml file

Q. What are the different sections of the file for?

* plugins
* custom
* provider
* resources


