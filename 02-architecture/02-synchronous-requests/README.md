# Serverless Synchronous Requests

In this module we'll cover making synchronous requests in between Lambda functions. We'll do that by refactoring *wild-rydes-ride-fleet* *GetUnicorn* to handle direct invocation by *RequestRide* in *wild-rydes*, bypassing API Gateway.

## Goals and Objectives
**Objectives:**
* Understand how to perform synchronous requests when you may have made a web request

**Goals:**
* Refactor *wild-rydes-ride-fleet* *GetUnicorn* to be invoked directly

## Synchronous Requests

## Instructions

### 1. Deploy new services / update existing
Deploy updates to Wild Rydes application services. This step is to ensure that your code matches the code in the workshop examples.

#### wild-rydes-ride-record
Deploy the wild-rydes-ride-record service. Start by cloning the repository from GitHub, then check out the workshop-operations-01 branch for this workshop module, and finally deploy the application.

```
$ cd $WORKSHOP/
$ rm -rf wild-rydes-ride-record
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-ride-record.git
$ cd wild-rydes-ride-record
$ git checkout -b workshop-operations-02
$ npm install
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>

```
```
</p>
</details>

#### wild-rydes
Deploy the wild-rydes service. Start by cloning the repository from GitHub, then check out the workshop-operations-01 branch for this workshop module, and finally deploy the application.

```
$ cd $WORKSHOP/wild-rydes
$ git checkout -b workshop-operations-01 origin/workshop-operations-01
$ sls deploy -v
```

<details>
<summary><strong>Output</strong></summary>
<p>

```
```
</p>
</details>


### 2. Refactor *wild-rydes-ride-fleet* *handlers/get_unicorn.py* for direct invoke

### 3. Add new Lambda function handler in *serverless.yml*

### 4. Refactor *RequestRide* in *wild-rydes* to invoke *GetUnicornInvoke*

## Q&A
