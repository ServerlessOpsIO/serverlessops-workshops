# Serverless Application Security

In this module we'll add a new feature to Wild Rydes. However, this new feature has a variety of application security issues that require fixing. We'll find and fix these issues.

## Goals and Objectives:

**Objectives:**
* Understand the serverless security concerns of serverless.

**Goals:**
* Evaluate a new application and fix the application security issues with it.

## Instructions

### 1. Review / Fix Secrets Management
The _SendToGitter_ Lambda function requires an access token to send messages. A (mock) token exists in the code. Find the token in the *wild-rydes-feedback* *handlers/send_to_gitter.py* file, remove it, and use a more secure way to distribute the token value to the function.

You can solve this problem a variety of ways. Some approaches are:

1) **Deploy Time Environmental Variables:** Pass in the value using an environmental variable during deploy time. *Do not use this method except as a last resort. It's better than nothing but the next options are better.*
1) **AWS SSM Parameter Store**: Use [AWS Param Store](https://aws.amazon.com/systems-manager/features/#Parameter_Store) to centrally store the value and fetch it from there. There are two methods of using Param Store. One is via a deploy time value lookup by Serverless Framework and the second is having the _SendToGitter_ function lookup the value in Param Store during execution. When choosing between the two, pay attention to the amount of exposure the key will receive based on the method chosen
1) **AWS Secrets Manager**: Use [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) to manage the token value. While Secrets Manager and Param Store appear similar, compare the feature set and cost of Secrets Manager against Param Store.

If you're unsure what to do, choose AWS SSM Parameter Store.

<details>
<summary><strong>Hint</strong></summary>
<p>

The token value is: lz7puf62hei8f2mty2teuwq0m740i73kxm65iihy

</p>
</details>

### 2. Insecure Application Dependency

The wild-rydes-feedback has an application dependency vulnerability. Click here to see more about the vulnerability. [![Known Vulnerabilities](https://snyk.io/test/github/ServerlessOpsIO/wild-rydes-feedback/badge.svg)](https://snyk.io/test/github/ServerlessOpsIO/wild-rydes-feedback)

Fix the vulnerability by update the library.

<details>
<summary><strong>Hint</strong></summary>
<p>

Update the _requests_ module in _requirements.txt_ from version 2.19.1 to the latest version found here:

* https://pypi.org/project/requests/
</p>
</details>


## Q&A

### 1. Secrets Management

Q. Which Secrets Management method did you choose and why?

Q. Explain the feature differences between Secrets Manager and Param Store.

<details>
<summary><strong>Hint</strong></summary>
<p>

Compare the ability to rotate values using each service.
</p>
</details>

Q. Explain the price differences between Secrets Manager and Param Store
<details>
<summary><strong>Hint</strong></summary>
<p>

See the respective pricing pages:

* [SSM Param Store](https://aws.amazon.com/systems-manager/pricing/)
* [Secrets Manager](https://aws.amazon.com/secrets-manager/pricing/)
<details>
<summary><strong>Hint</strong></summary>
<p>


