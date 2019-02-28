# Serverless Reliability

In this module we're cover aspects of ensuring reliability of a serverless application. We'll do this by addressing an issue you may run across with DynamoDB, how that issue might cause silent failures, and finally how to mitigate those failures.

## Goals and Objectives

__Objectives:__

* Understand where and how a serverless event-driven application can fail

**Goals**:

* Increase reliability of ride event delivery by:
  * Implementing Lambda dead letter queues
  * Configuring DynamoDB for scalability

## Serverless Reliability
<!-- FIXME: Add diagram of current arch and future arch -->

We're now going to address the failures we saw in the first module.  In moving to an event-driven architecture for writing ride information to DynamoDB in the previous module, we hid the problems in our architecture from the user but we did solve them. This illustrates an issue in event-driven architectures. You may have failures down stream which need to be caught and handled because the triggering action has already finished. (eg. If recording the a ride fails we can't force the user to order a new one so we can trigger the process again.)

### DynamoDB Scaling

What we want to do in this module is first minimize the liklihood that DynamoDB writes fail. By default, DynamoDB tables use provisioned capacity for processing reads and writes. Capacity is measured in RCUs for reads and WCUs for writes. They are calcualted as follows:

* **Write capacity unit (WCU):** Each API call to write data to your table is a write request. For items up to 1 KB in size, one WCU can perform one *standard* write request per second. Items larger than 1 KB require additional WCUs. *Transactional* write requests require two WCUs to perform one write per second for items up to 1 KB. For example, a standard write request of a 1 KB item would require one WCU, a standard write request of a 3 KB item would require three WCUs, and a transactional write request of a 3 KB item would require six WCUs.
* **Replicated write capacity unit (rWCU):** When using DynamoDB global tables, your data is written automatically to multiple AWS Regions of your choice. Each write occurs in the local region as well as the replicated regions. As a best practice, you should provision for an rWCU in each replicated region, times two. For example, if you have deployed a global table in US East (Ohio) and EU (Frankfurt) and you perform 5 writes per second to your global table, you should provision at least 20 rWCUs (5 writes x 2 regions x 2).

To handle variable workloads DynamoDB provides two options. One is to configure auto-scaling for provisioned capacity. This is similar to EC2 auto-scaling. When WCUs or RCUs reach a certain threshold, additional capacity is provisioned. There is however a lag in when this njew capacity is available.

The second option is to configure on-demand capacity. While auto-scaling is less expensive than on-demand scaling, the delay in provisioning capacity can result in database operation failures which makes on-demand capacity ideal for non-predictable or spikey workloads.

In this module we'll configure DynamoDB to use on-dmeand scaling. 

### Lambda Dead Letter Queues

Because it's still possible for DynamoDB writes to fail (the cloud is still unpredictible) we will also setup a dead letter queue for the *PutRideRecordSns* Lambda function. If the Lambda function fails to successfully complete after its retries are exhausted, the event will deposited to an SQS queue. This allows us to retry processing the event at a later point.

## Instructions
### 1. Add dead letter queue to _PutRideRecordSns_ in _wild-rydes-ride-record_
Add a dead letter queue for the _PutRideRecordSns_ Lambda function. While SNS guarantees message delivery to our Lambda function, our function is not guaranteed to execute successfully. When a Lambda function is triggered by an SNS event and the Lambda function fails to complete successfully, [Lambda will retry the function up to twice more with delays in between invocations](https://docs.aws.amazon.com/lambda/latest/dg/retries-on-errors.html). If all three invocations fail we want to drop the event is either discarded, or, sent to an SNS topic or SQS queue if a [dead letter configuration](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-lambda-function-deadletterconfig.html) exists. We’ll add an SQS queue in this step and we'll do this by using a Serverless Framework plugin called [serverless-plugin-lambda-dead-letter](https://github.com/gmetzker/serverless-plugin-lambda-dead-letter).


#### Serverless Framework plugin
Install the Serverless Framework plugin serverless-plugin-lambda-dead-letter.
```
$ sls plugin install -n serverless-plugin-lambda-dead-letter
```
<details>
<summary><strong>Output</strong></summary>
<p>

```
Serverless: Installing plugin "serverless-plugin-lambda-dead-letter@latest" (this might take a few seconds...)
Serverless: Successfully installed "serverless-plugin-lambda-dead-letter@latest"
```
</p>
</details>

#### SQS Queue
Create an SQS queue. The SQS Queue's CloudFormation resource name should be *PutRideRecordSnsDlq*. You won't need to configure any properties on the CloudFormation resource.

Use the CloudFormation documentation linked below.
* [CloudFormation AWS::SQS::Queue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html)

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -93,6 +103,9 @@ resources:
               - ".amazonaws.com/${self:custom.stage}"
               - "${self:custom.sevrice_url_path_base}"

+    PutRideRecordSnsDlq:
+      Type: AWS::SQS::Queue
+
   Outputs:
     RideRecordUrl:
       Description: "URL of service"
```
</p>
</details>

#### IAM permissions
Give the *PutRideRecordSns* Lambda function the ability to write to the queue.  Do this by adding an additional IAM policy statement under the `provider.iamRoleStatements` key that allows the *sqs:SendMessage* action on the new SQS queue. You can look at the existing statement allowing writes to DynamoDB for guidance.

The following documentation resources will help you accomplish this.

* [Serverless Framework IAM Role Statements](https://serverless.com/framework/docs/providers/aws/guide/iam/)
* [SQS IAM Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonsqs.html)
* [CloudFormation AWS::SQS::Queue - Return Values](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html#aws-properties-sqs-queues-ref)

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -39,6 +39,13 @@ provider:
         Fn::GetAtt:
           - RideRecordTable
           - Arn
+    - Effect: Allow
+      Action:
+        - sqs:SendMessage
+      Resource:
+        Fn::GetAtt:
+          - PutRideRecordSnsDlq
+          - Arn

 functions:
   PutRideRecordHttp:
```
</p>
</details>

#### Lambda Function Update
Add the dead letter queue configuration to the *PutRideRecordSns* Lambda function. You do this by adding a `deadLetter.targetArn.GetResourceArn` key for the function and it’s value is the name of the SQS queue CloudFormation resource, *PutRideRecordSnsDlq*.

You can refer to the documentation for *serverless-plugin-lambda-dead-letter*.

* [serverless-plugin-lambda-dead-letter Method-3: Use a queue/topic created in the resources](https://github.com/gmetzker/serverless-plugin-lambda-dead-letter#method-3)

<details>
<summary><strong>Output</strong></summary>
<p>

```diff
@@ -62,6 +62,9 @@ functions:
     environment:
       DDB_TABLE_NAME:
         Ref: RideRecordTable
+    deadLetter:
+      targetArn:
+        GetResourceArn: PutRideRecordSnsDlq
     events:
       - sns: "${self:custom.wild_rydes_sns_topic_arn}"

```
</p>
</details>

Now if there’s a failure by your *PutRideRecordSns* Lambda function to process an event, the event won’t be lost and can be reprocessed later.



### 2. Add DynamoDB scaling in *wild-rydes-ride-record*
Add the ability to scale to your DynamoDB table by enabling on-demand pricing. There isn't enough data to properly size the read and write capacity on the table so this is the best choice for scaling the table. If we could predict load, then we would have chosen to implement DynamoDB auto-scaling.

To do this, remove the *ProvisionedThroughput* configuration on the *RideRecordTable* CloudFormation resource and add the *BillingMode* key with a value of *PAY_PER_REQUEST*.

Reference the CloudFormation documentation here:

* [CloudFormation AWS::DynamoDB::Table](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-dynamodb-table.html)

<details>
<summary><strong>Answer</strong></summary>
<p>

```diff
--- a/serverless.yml
+++ b/serverless.yml
@@ -65,9 +65,7 @@ resources:
         KeySchema:
           - AttributeName: ${self:custom.ddb_table_hash_key}
             KeyType: HASH
-        ProvisionedThroughput:
-          ReadCapacityUnits: 1
-          WriteCapacityUnits: 1
+        BillingMode: PAY_PER_REQUEST

     ServiceUrlSsmParam:
       Type: "AWS::SSM::Parameter"
```
</p>
</details>


## Q&A

### DynamoDB auto-scaling

Q. What are the pros and cons of DynamoDB table provisioned capacity with auto-scaling versus on-demand capacity?

<details>
<summary><strong>Answer</strong></summary>
<p>

Provisioned capacity with auto-scaling is cheaper than on-demand capacity but additional provisioned capacity takes time to become available.

</p>
</details>

Q. What sort of DynamoDB usage patterns are ideal provisioned capacity with auto-scaling? What sort of suage patterns are idea for on-demand capacity? *Think about uniformity of read and write access.*
<details>
<summary><strong>Answer</strong></summary>
<p>
On-demand capacity is ideal for unpredictable or spikey workloads. In these situations auto-scaling may not provision capacity fast enough to keep up with demand.

However, if there's little to no issue with latency of adding a record to DynamoDB, you could investigate letting write's fail, fall into the dead letter queue, and be processed after additional capacity has finished provisioning. But you should perform financial calculations to see if the savings of using provisioned capacity is worth it.

</p>
</details>

**EXTRA CREDIT** Q. You may find your traffic is predictible and also at a scale where the cost overhead of on-demand DynamoDB scaling makes configuring auto-scaling more appealing. Configure your DynamoDB table. Use the [serverless-dynamodb-autoscaling Serverless Framework plugin](https://github.com/sbstjn/serverless-dynamodb-autoscaling) to setup auto-scaling.

### Dead Letter Queues

**EXTRA CREDIT** Q. Write a function to process the dead letter queue messages so that their data makes its way into DynamoDB.