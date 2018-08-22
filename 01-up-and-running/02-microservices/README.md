# Serverless Microservices

In this module we'll breakdown the Wild Rydes application into three separate microservices. This serverless architecture style is increasingly likely to be employed as an application scales.

## Goals and Objectives:

**Objectives:**
* Understand constructing a serverless application with microservices

Goals:
* Deploy separate microservices to create Wild Rydes.

## Tech Stack

The Wild Rydes application has been broken down into three separate microservices. They are:

* [wild-rydes-website](https://github.com/ServerlessOpsIO/wild-rydes-website) (backend) (frontend)
* [wild-rydes-ride-requests](https://github.com/ServerlessOpsIO/wild-rydes-ride-requests) (backend)
* [wild-rydes-ride-fleet](https://github.com/ServerlessOpsIO/wild-rydes-ride-fleet) (backend) (backend / data layer)

![Wild Rydes Microservices](../../images/wild-rydes-arch.png)

## Instructions

### 1. Deploy wild-rydes-ride-fleet

Deploy the _wild-rydes-ride-fleet_ service. This service is a RESTful API that fetches available unicorns and returns their info to the requesting service. The service is composed of API Gateway, an AWS Lambda function, and a DynamoDB table.

```
$ cd wild-rydes-ride-fleet
$ npm install
$ sls deploy -v
```

### 2. Deploy wild-rydes-ride-requests

Deploy the _wild-rydes-ride-requests_. This a RESTful web API that fetches an availble ride from the wild-rydes-ride-fleet and returns the pickup information to the ride requester. The service is composed of API Gateway and an AWS Lambda function.
```
$ cd wild-rydes-ride-requests
$ npm install
$ sls deploy -v
```

### 3. Deploy wild-rydes-website

Deploy the _wild-rydes-website service_. This is a statically hosted website served from an S3 bucket and a DNS record in Route53.


Before the service can be deployed it needs to know the location of the _wild-rydes-ride-requests_ endpoint. Obtain the _RequestRideUrl_ stack output value by running `sls info -v`.
```
$ cd wild-rydes-ride-requests   # If not already in that directory
$ sls info -v
```

<details>
<summary><strong>Output</strong></summary>
<p>

```
Service Information
service: wild-rydes-website
stage: dev
region: us-east-1
stack: wild-rydes-website-dev
api keys:
  None
endpoints:
  None
functions:
  None

Stack Outputs
StaticSiteS3BucketName: wild-rydes-website-dev.dev.training.serverlessops.io
StaticSiteS3BucketWebsiteURL: http://wild-rydes-website-dev.dev.training.serverlessops.io
ServerlessDeploymentBucketName: wild-rydes-website-dev-serverlessdeploymentbucket-ou14fw4ivpm9
```
</p>
</details>

With the value obtained, update the file `static/js/config.js`
```
$ cd wild-rydes-ride-website
$ nano static/js/config.js
```

Update the _api.invokeUrl_ value.
```javascript
window._config = {
    cognito: {
        userPoolId: '', // e.g. us-east-2_uXboG5pAb
        userPoolClientId: '', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
        region: '', // e.g. us-east-2
        disabled: true
    },
    api: {
      invokeUrl: '' // <-- Update this,
    }
};
```

Now deploy the service.
```
$ cd wild-rydes-ride-website
$ npm install
$ sls deploy -v
```

### 4. Navigate To Application

Navigate to the newly deployed application. In the output of the previous step, look for the _StaticSiteS3BucketWebsiteURL_ in _Stack Outputs_. This is the URL of the newly deployed application.  Navigate to it in a web browser. Request a ride and ensure the process is successful.


![Wild Rydes Web Site](../../images/wild-rydes-site.png)


### 5. Tail application logs

Tail the Lambda function logs from the wild-rydes-ride-requests service. Serverless Framework's `logs` command will poll and dump a Lambda function's logs from CloudWatch to the terminal window.

Start by getting the functions in the application stack using Serverless Framework's `info` command.

```
$ sls info
```
<details>
<summary><strong>Output</strong></summary>
<p>

```
Service Information
service: wild-rydes-ride-requests
stage: dev
region: us-east-1
stack: wild-rydes-ride-requests-dev
api keys:
  None
endpoints:
  POST - https://a0wh3ig8vh.execute-api.us-east-1.amazonaws.com/dev/ride
functions:
  RequestRide: wild-rydes-ride-requests-dev-RequestRide
  LoadTable: wild-rydes-ride-requests-dev-LoadTable
  StaticSiteConfig: wild-rydes-ride-requests-dev-StaticSiteConfig
```
</p>
</details>

The function's name is under the _functions_ key and called _RequestRide_. We'll ignore LoadTable and _StaticSiteConfig_ for now.

Begin tailing the `RequestRide` logs. This will show the log output from usage of the application in the previous step.

```
$ sls logs -f RequestRide -t
```

<details>
<summary><strong>Output</strong></summary>
<p>

```
2018-08-20 12:09:06.145 (-04:00)                [INFO]  2018-08-20T16:09:06.145ZSTART RequestId: 5c6c0221-a493-11e8-88e8-cd6d1f1b5e45 Version: $LATEST
2018-08-20 12:09:06.199 (-04:00)        5c6c0221-a493-11e8-88e8-cd6d1f1b5e45    [INFO]  Request: {"resource": "/ride", "path": "/ride", "httpMethod": "POST", "headers": {"Accept": "*/*", "Accept-Encoding": "gzip, deflate, br", "Accept-Language": "en-US,en;q=0.9", "CloudFront-Forwarded-Proto": "https", "CloudFront-Is-Desktop-Viewer": "true", "CloudFront-Is-Mobile-Viewer": "false", "CloudFront-Is-SmartTV-Viewer": "false", "CloudFront-Is-Tablet-Viewer": "false", "CloudFront-Viewer-Country": "US", "content-type": "application/json", "Host": "a0wh3ig8vh.execute-api.us-east-1.amazonaws.com", "origin": "http://wild-rydes-dev.dev.training.serverlessops.io", "Referer": "http://wild-rydes-dev.dev.training.serverlessops.io/ride.html", "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36", "Via": "2.0 83d82856eafc6ceb7ba06a257022fa7c.cloudfront.net (CloudFront)", "X-Amz-Cf-Id": "RJLS5Ymq-ucDHUUNnkqL98NGHLhaLMnz9nT9L4n9E3Pp9GTHolb8DA==", "X-Amzn-Trace-Id": "Root=1-5b7ae7a1-8b57b88c497a27d840f08ffc", "X-Forwarded-For": "73.17.175.174, 52.46.29.64", "X-Forwarded-Port": "443", "X-Forwarded-Proto": "https"}, "queryStringParameters": null, "pathParameters": null, "stageVariables": null, "requestContext": {"resourceId": "eznzv3", "resourcePath": "/ride", "httpMethod": "POST", "extendedRequestId": "L7khMHvjIAMFsMQ=", "requestTime": "20/Aug/2018:16:09:05 +0000", "path": "/dev/ride", "accountId": "144121712529", "protocol": "HTTP/1.1", "stage": "dev", "requestTimeEpoch": 1534781345249, "requestId": "5c6b65a9-a493-11e8-bb26-4769c4cfde0e", "identity": {"cognitoIdentityPoolId": null, "accountId": null, "cognitoIdentityId": null, "caller": null, "sourceIp": "73.17.175.174", "accessKey": null, "cognitoAuthenticationType": null, "cognitoAuthenticationProvider": null, "userArn": null, "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36", "user": null}, "apiId": "a0wh3ig8vh"}, "body": "{\"PickupLocation\":{\"Latitude\":42.36317996431076,\"Longitude\":-71.05193588435529}}", "isBase64Encoded": false}
2018-08-20 12:09:06.259 (-04:00)        5c6c0221-a493-11e8-88e8-cd6d1f1b5e45    [INFO]  Starting new HTTPS connection (1): dynamodb.us-east-1.amazonaws.com
2018-08-20 12:09:06.414 (-04:00)        5c6c0221-a493-11e8-88e8-cd6d1f1b5e45    [INFO]  Response: {"statusCode": 201, "body": "{\"RideId\": \"5cfc9954-a493-11e8-a910-425746ae81de\", \"Unicorn\": {\"Name\": \"Bucephalus\", \"Color\": \"Golden\"}, \"RequestTime\": \"2018-08-20 16:09:06.200610\"}", "headers": {"Access-Control-Allow-Origin": "*"}}
END RequestId: 5c6c0221-a493-11e8-88e8-cd6d1f1b5e45
REPORT RequestId: 5c6c0221-a493-11e8-88e8-cd6d1f1b5e45  Duration: 214.69 ms     Billed Duration: 300 ms         Memory Size: 128 MB     Max Memory Used: 41 MB
```
</p>
</details>

If you received the error `No existing streams for the function` then either you did not request a ride in the previous step or logs have been delayed in reaching CloudWatch.


### 6. Invoke function

Invoke the _RequestRide_ function without going through the application frontend or API Gateway. The file _tests/events/request-ride-event.json_ is a mock API Gateway event that resembles the data that would be passed by API GAteway to the Lambda function.

```
$ sls invoke -f PostRideRecord -p tests/events/request-ride-event.json
```

<details>
<summary><strong>Output</strong></summary>
<p>

```json
{
    "statusCode": 201,
    "body": "{\"RideId\": \"30c565ea-a494-11e8-a910-425746ae81de\", \"Unicorn\": {\"Name\": \"Shadowfax\", \"Color\": \"White\"}, \"RequestTime\": \"2018-08-20 16:15:01.515825\"}",
    "headers": {
        "Access-Control-Allow-Origin": "*"
    }
}
```
</p>
</details>

## Questions

### 1. Service Discovery

Q. How does wild-rydes-ride-requests know the wild-rydes-ride-fleet API endpoint?
<details>
<summary><strong>Answer</strong></summary>
<p>

</p>
</details>

Q. How does wild-rydes-ride-website know the wild-rydes-ride-requests API endpoint?
<details>
<summary><strong>Answer</strong></summary>
<p>

</p>
</details>

### 2. Relationship between S3 bucket and Route53 record.

Q. Explain the relationship between the S3 bucket and Route53 for DNS resolution. (Hint: Look at the bucket name and compare it with the Route53 record name.)
<details>
<summary><strong>Answer</strong></summary>
<p>

</p>
</details>

Q. Why doesn't the frontend use HTTPS?
<details>
<summary><strong>Answer</strong></summary>
<p>

The SSL certificate for S3 only supports Amazon S3's own domain names. To use an SSL cert with our own domain name we'dd need to use AWS CloudFront as our CDN and have it serve content from our S3 bucket. Deploying CloudFront can be time consuming so it was dropped from this module.
</p>
</details>


