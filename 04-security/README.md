# Serverless Security
<!--
https://www.kevinlondon.com/2015/08/15/dangerous-python-functions-pt2.html
* YAML deserialization!
* maybe it grabs an API token from an environment file.

* Introduce an insecure service and secure it.
    * Maybe that chatbot?
    * Maybe a log shipper?
1) Infrastructure & access
    * S3 access?
    * IAM statements and roles
        * role per function
        * no '*' permissions
2) Application
    * Code?
        * event injection?
        * bad deps.
        * token handling / secrets management.
            * Don't leave hard coded in a file
            * Use param store instead.
        * hard coded token
    * use OWASP top 10
3) API Access
    * Cognito!
        * add to frontend
        * Add to ride-fleet

* Introduce a chatbot?
    * Bot sends help requests to a slack channel....
        * APIG + Lambda + DynamoDB + DDB stream or SNS + Lambda / Slack (or gitter!)
        * APIG + Lambda + SNS
            * Lambda + Gitter
            * Lambda + S3 (+ Lambda + Lex?)
    * Need Cognito so people aren't openly talking to Slack
    * Can we do an event injection?
-->

This workshop demonstrates typical security issues and implications around serverless infrastructure. We'll be introduced to a simple service that will be added to Wild Rydes application and we'll assess common security issues and how to remediate them. It will cover the following topics:

* [Application Security](./01-application/)
* [Infrastructure Security](./02-infrastructure/)
* [Authentication And Authorization](./03-authn-and-authz/)

After the workshop the participant should be familiar with common serverless security issues, what to look for, and how to remediate them.

## Tech Stack

This workshop will introduce a new service into the Wild Rydes application. The new service is called wild-rydes-feedback. This service, combined with an update to wild-rydes, will allow users to deliver feedback on the application. Feedback will be directed to two different services. The first service is a [Gitter](https://gitter.im/home) chat room. This allows us to monitor feedback on the site in real time. The second service is an S3 bucket. At a later point we plan on building a sentiment analysis pipeline. But for now, we're just interested in collecting feedback.

AWS Services used:
* [API Gateway](https://aws.amazon.com/api-gateway/)
* [Lambda](https://aws.amazon.com/lambda/)
* [SNS](https://aws.amazon.com/sns/)
* [Cognito](https://aws.amazon.com/cognito/)
* [SSM Parameter Store](https://aws.amazon.com/systems-manager/features/#Parameter_Store)
* [IAM](https://aws.amazon.com/iam/)

