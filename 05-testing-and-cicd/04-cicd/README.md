# Serverless CI/CD

This workshop module is just a demonstration of a CI/CD pipeline. Each service in this workshop already has a [TravisCI](https://travis-ci.org) configuration. You won't be building a new pipeline, just observing an example pipeline.

## CI/CD Pipeline
The testing and deploy pipeline is broken up into multiple stages. This is to make determining where the process failed quicker. But it is done at the expense of a longer running pipeline as Travis needs to create a new stage environment, fetch the build artifact, and extract it.

Out pipeline adheres to the following constraints.

* A single artifact is used for all pipeline stages.
* Deployment environments are separated by accounts.

<!-- DIAGRAM -->

### Build Stage
The build stage runs unit tests and creates a deployable artifact that will be used for all subsequent stages. Tests are run for all changes to master and on PRs.

A build artifact is uploaded to S3. However, a build artifact is only created on changes to master such as after a PR merge.


### Deploy To Dev Stage
The build artifact is deployed to the development pipeline.

### Integration Test Stage
This stage runs integration tests against the newly deployed dev environment. This is its own stage because it differentiates between deployment errors (which would occur in the previous stage) versus application errors.

### Promote To Prod Stage
The build artifact is copied from the dev account to a bucket in the production account.

### Deploy To Prod Stage
The build artifact is deployed to the production environment.
