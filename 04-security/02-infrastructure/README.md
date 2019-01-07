<!--
Good ideas: https://www.puresec.io/blog/aws-security-best-practices-config-rules-lambda-security
-->

# Serverless Infrastructure Security

<!-- We could also use wild-ryde-ride-data-lake instead for these tasks... A data lake is an awesoem place to have a breach! -->

In this module we'll add a new service on the backend of Wild Rydes to stream ride requests data to an S3 bucket for further data analysis. Additionally, we've updated wild-rydes-ride-record to support ride update and cancellation operations.  However, both our new service and updated service have a variety of cloud infrastructure security issues that require fixing. We'll find and fix these issues typically by fixing AWS IAM issues.

## Goals and Objectives:

**Objectives:**
* Understand the cloud infrastructure security concerns of serverless.

**Goals:**
* Evaluate a new application and fix the cloud infrastructure security issues with it.

## Infrastructure Security
### Access Controls / AWS IAM

This lab will primarily deal with AWS access controls. In particular, we will make use of [AWS IAM](https://aws.amazon.com/iam/) to properly secure our application infrastructure. Incorrect or far too broad IAM policies lead to many of the security issues that organizations see. This module will illustrate common mistakes people make and demonstrate best practices.

Terminology:

* _IAM Policy:_ A collection of IAM permission statements. These are typically attached to an IAM Role, Group, or User. (Best practice is to not attach policies to users.)
* _IAM Role:_ A collection of IAM policies. We attach IAM Roles to Lambda functions to give them permission to access other AWS resources.

IAM Permissions Reference:

* https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html

### Infrastructure Monitoring / AWS Config

Once infrastructure security issues have been detected and remediated there should be a process put in place to prevent the issue from happening again. We'll use [AWS Config](https://aws.amazon.com/config/) to monitor for and alert on issues.

## Instructions

### 1. Deploy services
Deploy the services used for this module. You'll deploy an updated *wild-rydes-ride-record* and the new *wild-rydes-ride-data-lake* service.

#### wild-rydes-ride-record
Deploy an updated *wild-rydes-ride-record* service.  The updated service will export the DynamoDB stream ARN to SSM Parameter Store. the *wild-rydes-ride-data-lake* service will subscribe to this stream and write data to the data lake.

```
$ cd $WORKSHOP/wild-rydes-ride-record
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-ride-record.git
$ git checkout -b workshop-security-02
$ npm install
$ sls deploy -v
```

#### wild-rydes-ride-data-lake
Deploy the new *wild-rydes-ride-data-lake* service. This service takes ride data and deposits it to S3 for analysis.

```
$ cd $WORKSHOP
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-ride-data-lake.git
$ cd wild-rydes-ride-data-lake
$ git checkout -b workshop-security-02
$ npm install
$ sls deploy -v
```

### 2. Review and fix security issues

This section has a number of security issues to be found and fixed. For now, stick to securing access to AWS resources.

#### Reduced function privileges / role per function
Serverless Framework, by default, creates a single IAM role for an entire service. That means every function has the same set of permissions and as a result Lambda functions have unnecessary access to other AWS resources. While we know of no ability to exploit this in our application, it violates a best practice.

Use the [serverless-iam-roles-per-function](https://www.npmjs.com/package/serverless-iam-roles-per-function) Serverless Framework plugin to give each function its own individual; role. You can take the existing IAM role statements and apply them to the proper function.

#### S3 bucket access
The S3 bucket that ride data is deposited to in wild-rydes-ride-data-lake is open to the world and this needs to be fixed. Limit bucket access to:

* The _SendToDataLake_ Lambda function

<details>
<summary><strong>Hint</strong></summary>
<p>

Remove the S3 bucket policy resource and update the _SendToDataLake_ IAM role.

</p>
</details>

#### Reduced resource access

All the functions in wild-rydes-ride record have more IAM permissions than are needed. Reduce them only to the operations they need in order to function.

<details>
<summary><strong>Hint</strong></summary>
<p>

* [SNS IAM Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonsns.html)
* [S3 IAM Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazons3.html)

</p>
</details>


### 3. Write an AWS Config Rule to alert on open buckets.

Create an [AWS Config](https://aws.amazon.com/config/) rule that will monitor and alert on the presence of open S3 buckets.

### 4. Deploy updated services

#### wild-rydes-ride-data-lake
Deploy the new *wild-rydes-ride-data-lake* service.

```
$ cd $WORKSHOP/wild-rydes-ride-data-lake
$ sls deploy -v
```

#### wild-rydes-ride-record
```
$ cd $WORKSHOP/wild-rydes-ride-record
$ sls deploy -v
```
## Q&A

### S3 Bucket Access

Q. We granted access by the _SendToS3Bucket_ function to the appropriate S3 bucket. How would we grant internal users access to the bucket?

<details>
<summary><strong>Hint</strong></summary>
<p>

1) Create an IAM policy, attach it to an AWS group, and ensure all users who need access are members of that group.
1) Create a policy to the bucket that grants access.

Where possible, attempt to use the first option.

</p>
</details>

### IAM Roles


### AWS Config

