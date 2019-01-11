# Serverless Security
<!-- Some ideas for Python vulns
https://www.kevinlondon.com/2015/08/15/dangerous-python-functions-pt2.html
-->

This workshop demonstrates typical security issues  around serverless infrastructure and applications. We will introduce a new service with security issues and handle some existing security issues in Wild Rydes.

The modules in this workshop are:
* [Application Security](./01-application/)
* [Infrastructure Security](./02-infrastructure/)

After the workshop the participant should be familiar with common serverless security issues, what to look for, and how to remediate them.

## Security Concepts Covered
This workshop's modules will cover the following security concepts.

* API access controls
  * Configure an API Gateway authorizer to controll access to the API
* Secrets management and storage
  * Secure the API key in SSM Parameter Store, encrypted using KMS, and have a Lambda function fetch the key when invoked.
* Permissions scope and pinciple of least access
  * Rescope IAM polices that allow a too broad a range of actions on too broad a range of resoruces
* Insecure application depenencies
  * Find and update an application dependency with a reported vulnerability.
* Security monitoring and remediation
  * Use AWS Config create a monitoring rule for a previously encvountered security issue.