# Serverless API Access

Currently our applications endpoints are unauthenticated. That means anyone that is able to figure out our API endpoints can send request to them directly.

## Goals and Objectives:

**Objectives:**
* Understand how to secure API endpoints.

**Goals:**
* Use AWS Cognito to secure Wild Rydes and its service endpoints

## API Security / AWS Cognito
Our API gateway endpoints are exposed to the internet with now firewall or access controls. As a result, anyone can access them directly. We'll use [AWS Cognito](https://aws.amazon.com/cognito/) to provide authentication and authorization services on our endpoints. Adding AWS Cognito will require adding the user signup workflow, adding a login process, and sending authorization info to our endpoints.

* [AWS Cognito Documentation](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)

## Instructions

### 1. Update wild-rydes frontend
In this step we'll create Cognito resources for managing application users and deploy an updated site frontend with a user signup workflow.

Checkout the the _api-access_ branch of the wild-rydes Github repository.

```
$ cd ${WORKSHOP}/wild-rydes
$ git checkout api-access
```

Add Cognito resources to the service.

<details>
<summary><strong>Hint</strong></summary>
<p>

Show Cognito CFN Docs
</p>
</details>
<details>
<summary><strong>Answer</strong></summary>
<p>

Show Cognito CFN
</p>
</details>

The front end has already had its code updated to introduce the signup and login workflow. But, you will need to provide the Cognito pool information in the _static/js/config.js_ file.
<details>
<summary><strong>Hint</strong></summary>
<p>

Show Cognito CFN Docs
</p>
</details>
<details>
<summary><strong>Answer</strong></summary>
<p>

Show Cognito CFN
</p>
</details>

Now you can deploy the updated application.
```
$ npm install
$ sls deploy -v
```

### 2. Sign and register for Wild Rydes
Navigate to the newly updated application. If you can't remember the location of the website, do the following steps and get the _StaticSiteS3BucketWebsiteURL_ in _Stack Outputs_:

```
$ cd $WORKSHOP/wild-rydes
$ sls info -v
```

Click the _Giddy Up_ button which will take you to the user registration page. Signup for the service with a valid email and provide a password. You'll receive an email to confirm your account. Once you confirm, login to the service.

### 3. Update wild-rydes backend
Update API Gateway to check Cognito credentials.

<details>
<summary><strong>Hint</strong></summary>
<p>

Show Cognito / API Gateway docs
</p>
</details>
<details>
<summary><strong>Answer</strong></summary>
<p>

Show serverless.yml
</p>
</details>

Update _RequestRide_ function to make requests to both _wild-rydes-ride-fleet_ and _wild-rydes-feedback_ using Cognito credentials.

<details>
<summary><strong>Hint</strong></summary>
<p>

Show Cognito / API Gateway docs
</p>
</details>
<details>
<summary><strong>Answer</strong></summary>
<p>

Show serverless.yml
</p>
</details>

Now deploy service.
```
$ sls deploy -v
```

### 4. Update wild-rydes-ride-fleet
Update API Gateway to require AWS Cognito.

<details>
<summary><strong>Hint</strong></summary>
<p>

Show Cognito / API Gateway docs
</p>
</details>
<details>
<summary><strong>Answer</strong></summary>
<p>

Show serverless.yml
</p>
</details>

### 5. Update wild-rydes-feedback.
Update API Gateway to require AWS Cognito.

<details>
<summary><strong>Hint</strong></summary>
<p>

Show Cognito / API Gateway docs
</p>
</details>
<details>
<summary><strong>Answer</strong></summary>
<p>

Show serverless.yml
</p>
</details>


### 6. Request a ride from site
Navigate to site and successfully request a ride.

### 7. Request a ride using `curl`
Attempt to use curl to request a ride.

```
$ curl ...
```

You should receive an access denied error.

## Q&A

### API endpoints

Q. How would someone find our endpoint URLs

<details>
<summary><strong>Hint</strong></summary>
<p>


</p>
</details>

