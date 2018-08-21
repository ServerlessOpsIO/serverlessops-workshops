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


## Serverless Framework serverless.yml


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

With the value obtained, update the file `static/js/config.js`
```
$ cd wild-rydes-ride-website
$ nano static/js/config.js

```

Update the _api.invokeUrl_ value.
```yaml
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

## Questions

### 1. Service Discovery

Q. How does wild-rydes-ride-requests know the wild-rydes-ride-fleet API endpoint?

Q. How does wild-rydes-ride-website know the wild-rydes-ride-requests API endpoint?

### 2. Relationship between S3 bucket and Route53 record.

Q. Explain the relationship between the S3 bucket and Route53 for DNS resolution. (Hint: Look at the bucket name and compare it with the Route53 record name.)

Q. Why doesn't the frontend use HTTPS?


