# Serverless Infrastructure Security

In this module we'll add a new feature to Wild Rydes. However, this new feature has a variety of cloud infrastructure security issues that require fixing. We'll find and fix these issues typically by fixing AWS IAM issues.

## Goals and Objectives:

**Objectives:**
* Understand the cloud infrastructure security concerns of serverless.

**Goals:**
* Evaluate a new application and fix the cloud infrastructure security issues with it.

## Access Controls

This lab will primarily deal with AWS access controls. In particular, we will make use of [AWS IAM](https://aws.amazon.com/iam/) to properly secure our application infrastructure. Incorrect or far too broad IAM policies lead to many of the security issues that organizations see. This module will illustrate common mistakes people make and demonstrate best practices.

Terminology:

* _IAM Policy:_ A collection of IAM permission statements. These are tyipically attached to an IAM Role, Group, or User. (Best practice is to not attach policies to users.)
* _IAM Role:_ A collection of IAM policies. We attach IAM Roles to Lambda functions to give them permission to access other AWS resources.

IAM Permissions Reference:

* https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_actions-resources-contextkeys.html

## Instructions

### 1. Review and fix Security issues

This section has a number of security issues to be found and fixed. For now, stick to securing access to AWS resources.

#### Reduced function privileges / role per function
Serverless Framework, by default, creates a single IAM role for an entire service. That means every function has the same set of permissions and as a result Lambda functions have unnecessary access to other AWS resources. While we know of no ability to exploit this in our application, it violates a best practice.

Use the [serverless-iam-roles-per-function](https://www.npmjs.com/package/serverless-iam-roles-per-function) Serverless Framework plugin to give each function its own individual; role. You can take the existing IAM role statements and apply them to the proper function.

#### S3 bucket access
The S3 bucket that feedback is deposited in is open to the world and this needs to be fixed. Limit bucket access to:

* The _SendToS3Bucket_ Lambda function

<details>
<summary><strong>Hint</strong></summary>
<p>

Remove the S3 bucket policy resource and update the _SendToS3Bucket_ IAM role.

</p>
</details>

#### Reduced resource access

The _DispatchFeedback_ and _SendToS3Bucket_ functions have extensive IAM permissions. Reduce them only to the operations they need in order to function.

<details>
<summary><strong>Hint</strong></summary>
<p>

* [SNS IAM Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonsns.html)
* [S3 IAM Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazons3.html)

</p>
</details>


### 2. Deploy Wild Rydes Feedback

Deploy the new _wild-rydes-feedback_ service.

```
$ cd $WORKSHOP
$ git clone https://github.com/ServerlessOpsIO/wild-rydes-feedback.git
$ cd wild-rydes-ride-fleet
$ npm install
$ sls deploy -v
```

### 3. Deploy Updated Wild Rydes

If you haven't already checked out the _wild-rydes_ Github repository then do so now.

```
$ cd $WORKSHOP
$ git clone https://github.com/ServerlessOpsIO/wild-rydes.git
```

Deploy the new _wild-rydes_ GitHub branch. This branch has an update to the website frontend so users can enter in feedback that will be sent to the _wild-rydes-feedback_ service.

```
$ cd wild-rydes-ride-fleet
$ git checkout security
$ npm install
$ sls deploy -v
```

### 4. Use the new feedback service

Navigate to the newly updated application. If you can't remember the location of the website, do the following steps and get the _StaticSiteS3BucketWebsiteURL_ in _Stack Outputs_:

```
$ cd $WORKSHOP/wild-rydes
$ sls info -v
```

Navigate to the site in your browser and go to the ride request map. In the upper right corner you should see a new free text box. Enter a message in the box and click the _Submit_ button. (Be polite and considerate of your fellow participants)

Finally, you should see your message in the Gitter chatroom associated with this workshop. Click on the badge to join the room. Be sure to look for your message. [![Join the chat at https://gitter.im/ServerlessOpsIO/serverlessops-workshops](https://badges.gitter.im/ServerlessOpsIO/serverlessops-workshops-testing.svg)](https://gitter.im/ServerlessOpsIO/serverlessops-workshops-testing?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


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
