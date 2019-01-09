<!--
Good ideas: https://www.puresec.io/blog/aws-security-best-practices-config-rules-lambda-security
-->

# Serverless Infrastructure Security

In this module we'll find and fix infrastructure security issues. We'll find and fix common mistakes in configuration that many people make. Much of this module will revolve around misconfiguration of IAM permissions.

We're going to deploy a new small service called *wild-rydes-data-lake* which takes ride data and places it into an S3 data lake to be used by analysis tools. We've also added functionality to *wild-rydes-ride-record*. Both of these services have securiuty issues we'll address.

## Goals and Objectives:

**Objectives:**
* Understand the basic infrastructure security concerns of serverless
  * Open S3 buckets
  * Wide scoping of resources in IAM policies
  * Wide scoping of IAM actions

**Goals:**
* Properly write an S3 policy so that bucket data is not publicly accessible
* Remove braod access to multiple AWS resources (eg. all dynamoDB tables)
* Create Lambda function roles on a per function basis with the limited actions required


## Infrastructure Security
This lab will primarily deal with AWS access controls. In particular, we will make use of [AWS IAM](https://aws.amazon.com/iam/) to properly secure our application infrastructure. Incorrect or far too broad IAM policies lead to many of the security issues that organizations see. This module will illustrate common mistakes people make and demonstrate best practices. We'll also see an S3 bucket policy used. This is similar to IAM but applies only to S3 buckets

### Access Controls / AWS IAM
The AWS IAM service is the service in AWS that controls access to resources and services in AWS.  We've touched on IAM in other workshop modules as we've granted Lambda functions the ability to perform certain actions on other AWS resources. For example, add an item to a DynamoDB table. So far we've shown you mostly best practices. In this workshop we'll explore some best practices by demonstrating the poorer practices people often employ and how to fix them.

IAM entities can mostly be broken down into users, groups, roles, and policies. We're going to ignore IAM users and groups since they're not relevant to us in this workshop. With our serverless applications we should be most concered with the following.

* _IAM Policy:_ A collection of IAM permission statements. These statements determine what actions can be performed against what resoruces. A policy may consist of multiple statements. IAM policies can be attached to an IAM role, group, or user. (Best practice is to not attach policies to users.) 
* _IAM Role:_ An IAM identity that is meant to be assumed by a person (eg. an IAM user) or AWS resource (eg. AWS EC2 instance or Lambda function). IAM roles do not have access keys or passwords. Instead by an identity assuming the role the entity is granted temporary security credentials. For out purposes in this workshop, we configure Lambda functions to assume an IAM Role that gives them permission to access other AWS resources.

With the way Serverless Framework handles IAM policy statements, policies, and roles by default; we'll first concern ourselves with individual IAM policy statements. Later we'll configure Serverless Framework to handle IAM roles differently.

#### Creating an IAM role and policy

Serverless Framework automatically creates an IAM role per service and configures each function to assume that Lambda role. (That's one IAM role in a *serverless.yml* file that all functions in that file will be configured to assume.) Let's demonstrate how we manage IAM with Serverless Framework through example.

We configure I am policy statements via the `provider.iamRoleStatements` property. For example, see the following.

```yaml
provider:
  # NOTE: other provider properties have been removed for clarity.
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:PutItem
      Resource:
        Ref: MyDdbTable
    - Effect: Allow
      Action:
        - sqs:SendMessage
      Resource:
        Fn::GetAtt:
          - MySqsDlq
          - Arn

```



There are two policy statements defined in that example. One statement grants the ability to put an item into a specific DynamoDB table.  The second allows send an SQS message to an SQS queue. Each policy statement defines an effect, ehich is to allow an action (as opposed to deny), an action (as defined by the IAM API), and an AWS resource ARN (we use CloudFormation functions to get the ARN of different resources in our application stack.)

These statements adhere to a general principle to follow when working with IAM, *grant as a few actions to as a few resources as possible.* Commonly this is refered to as "the principle of least privilege".  Not following that is the cause of many security issues in AWS. People don't follow best practices not just because they're lazy or "bad people", they do so because IAM can be complicated. If you don't know what DyanmoDB IAM permissions a function needs it's tempting to just write "_dynamodb:*_" and if you're not sure how to get the ARN of a resource through Cloudfromation it's tempting to set the resource to "_*_".

The following two links, especially the second one will help you with crafting IAM policies:

* [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
* [Actions, Resources, and Condition Keys for AWS Services](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html)

While we're here, we've mentioned that Serverless Framework creates a single role that all functions in the service will assume. What if we have functions for not just put, but also get, update, and delete? Every function is going to have those permissions. Even functions that maybe don't have to interact with DynamoDB at all. And what if we have two DynamoDB tables in a service?... You get the picture. We'll use a Serverless Framework plugin called [serverless-iam-roles-per-function](https://www.npmjs.com/package/serverless-iam-roles-per-function) to create multiple functions and allow us to better adhere to the principle of least privilege.

### S3 Bucket Access / S3 Bucket Policies

S3 bucket policies are similar to IAM policies but only applu to S3 buckets. They have their uses such as allowing public access to bucket objects when S3 is used for web hosting or for granting the ability for entities in another AWS account to access a bucket. (There are better ways to solve that.) Unfortunately these often get people into trouble. People create policies that grant access to S3 data by unauthorized people through policy mistakes or simple frusturation. We'll cover one such example and fix it.

<!--  Leave out for now

### Infrastructure Monitoring / AWS Config

Once infrastructure security issues have been detected and remediated there should be a process put in place to prevent the issue from happening again. We'll use [AWS Config](https://aws.amazon.com/config/) to monitor for and alert on issues.
-->

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

#### S3 bucket access in *wild-rydes-ride-data-lake*
The S3 bucket that ride data is deposited to in *wild-rydes-ride-data-lake* is open to the world and this needs to be fixed. Limit access to the bucket while allowing the _WriteRecord_ Lambda function to still write to the bucket.

1) Remove the S3 bucket policy resource
1) Add an IAM role policy statement that grants the *S3:PutObject* action to the _WriteRecord_ function.

<details>
<summary><strong>Hint 2</strong></summary>
<p>

* [S3 Bucket Policy CloudFormation Resources](FIXME)
* [Serverless Framework iamRoleStatements](FIXME)

</p>
</details>


<details>
<summary><strong>Answers</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -24,6 +24,17 @@ provider:
   cfnRole: "arn:aws:iam::${env:AWS_ACCOUNT}:role/CloudFormationDeployRole"
   environment:
     LOG_LEVEL: ${self:custom.log_level}
+  iamRoleStatements:
+    - Effect: "Allow"
+      Action:
+        - "S3:PutObject"
+      Resource:
+        Fn::Join:
+          - '/'
+          - - Fn::GetAtt:
+              - WildRydesDataLakeBucket
+              - Arn
+            - '*'
   stackTags:
     x-service: wild-rydes-ride-data-lake
     x-stack: ${self:service}-${self:provider.stage}
@@ -48,24 +59,3 @@ resources:

     WildRydesDataLakeBucket:
       Type: "AWS::S3::Bucket"
-
-    WildRydesDataLakeBucketPolicy:
-      Type: AWS::S3::BucketPolicy
-      Properties:
-        Bucket:
-          Ref: WildRydesDataLakeBucket
-        PolicyDocument:
-          Statement:
-            - Sid: PublicReadGetObject
-              Effect: Allow
-              Principal: '*'
-              Action:
-                - s3:GetObject
-              Resource:
-                Fn::Join:
-                  - "/"
-                  - - Fn::GetAtt:
-                      - WildRydesDataLakeBucket
-                      - Arn
-                    - "*"
```
</p>
</details>


#### Reduce broad DynamoDB access in *wild-rydes-ride-record*
Narrow DynamnoDB IAM privileges down to only the DynamoDB table *RideRecordTable*.


#### One IAM role per function in *wild-rydes-ride-record*
Serverless Framework, by default, creates a single IAM role for an entire service. That means every function has the same set of permissions and as a result Lambda functions have unnecessary access to other AWS resources. While we know of no ability to exploit this in our application, it violates a best practice.

Use the [serverless-iam-roles-per-function](https://www.npmjs.com/package/serverless-iam-roles-per-function) Serverless Framework plugin to give each function its own individual role. You can take the existing IAM role statements and apply them to the proper function.


### 3. Deploy AWS Config Rules to alert on open buckets and * access

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

