# Serverless Operations
This workshop demonstrates typical problems in the operation of serverless systems and responses to these issues.

The modules in this workshop are:

* [Failure](./01-failure)
* [Performance](./02-reliability)
* [Monitoring](./03-monitoring)
* [Performance](./04-performance)

After the workshop the participant should be familiar with common serverless operational concerns and how to remediate them.

## Operations Concepts Covered

* Application failures issues
  * Find a fault in a service that causes the application to failure for a user.
  * Use metrics and logs to diagnose the issue
* Function Instrumentation
  * Add instrumentation to a function to help diagnose where issues lay.
* Architectural decision making
  * Refactor a service for optimal performance and cost.

## Tech Stack

This workshop will build off the services in the [Up And Running workshop](../01-up-and-running/) and will introduce new services as a part of Wild Rydes and a third-party service for monitoring the performance of a Lambda function.

3rd Party Services used:

* [Thundra](https://www.thundra.io/)
  * Provides monitoring, metrics, and observability
* [Artillery](https://artillery.io/)
  * Load testing tool to help us trigger and find issues.

Our tech stack is as follows.

* [wild-rydes](https://github.com/ServerlessOpsIO/wild-rydes)
  * Front end website and ride request backend
* [wild-rydes-ride-fleet](https://github.com/ServerlessOpsIO/wild-rydes-ride-fleet)
  * Front end website and ride request backend
* [wild-rydes-ride-record](https://github.com/ServerlessOpsIO/wild-rydes-ride-record)
  * Front end website and ride request backend
* [wild-rydes-pricing](https://github.com/ServerlessOpsIO/wild-rydes-pricing)
  * Backend service to determine surge or discount pricing
* [wild-rydes-fleet-location](https://github.com/ServerlessOpsIO/wild-rydes-fleet-location)
  * Locates dispatched member of fleet
