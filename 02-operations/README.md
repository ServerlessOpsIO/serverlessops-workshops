# Serverless Operations
This workshop demonstrates typical problems in the operation of serverless systems and responses to these issues. We'll be adding more complexity to the Wild Rydes application and in turn cause bugs to appear in the application. These will be demonstrated by performance issues seen by the user and failures on the backend.

* [Failure](./01-failure)
* [Performance](./02-performance)
* [Architecture](./03-architecture)

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
* [wild-rydes-fleet-dispatch](https://github.com/ServerlessOpsIO/wild-rydes-fleet-dispatch)
  * Dispatches unicorn from fleet
