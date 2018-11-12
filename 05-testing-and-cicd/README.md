<!--
We could create a new service that initially fails. We should test how long this module takes people. If only around an hour we should have people fix the problem.
-->

# Serverless CI/CD & Testing

This workshop demonstrates a typical software testing and CI/CD workflow. We'll work with the existing Wild Rydes services to add unit, integration, and load tests. Once tests have been run locally we'll setup a CI/CD pipeline.

The inability to fully run serverless applications locally makes local testing important for efficiency purposes. You cannot run your application locally in a container or VM and the deployment process to AWS can add distracting interupts and delays.

While there are attempts to recreate AWS services as locally runnable services, this workshop takes an opinionated view that avoids them. The pay per-use model of serverless means any engineer should be able to deploy their service in AWS when they need it.

* [Unit Testing](./01-unit-testing)
* [Integration Testing](./02-integration-testing)
* [Load Testing](./03-load-testing)
* [CI/CD](./04-cicd)

After the workshop the participant should be able to carry out the basic software and system testing lifecycle.

## Tech Stack

This workshop does not introduce any new services to the Wild Rydes application. However, software testing frameworks and a CI/CD platform will be introduced.

Software Testing
* [Pytest](https://docs.pytest.org/en/latest/): Python testing framework
* [Moto](http://docs.getmoto.org/en/latest/): Python module for mocking AWS services
* [Artillery](https://artillery.io/): Load testing framework

CI/CD
* [TravisCI](https://travis-ci.org/dashboard) - Ci/CD SaaS platform

## Pytest

Pytest is a Python testing framework...
