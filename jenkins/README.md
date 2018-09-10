# Sample Jenkins Declarative Pipeline
This folder contains a sample Jenkinsfile using declarative syntax for running Jobs as part of a CICD Pipeline.

The build process is broken down into `Stages`.

Environment variables are declared at the top of the file. Their values are adjusted at build time depending on which `Branch` of your code is being built.

For example, we adjust Environment values for our Production builds when the `master branch` is being built.

In the case of sensitive information, we pull the values from Jenkins Secrets rather than expose them inside the Jenkinsfile. This helps protect values for passwords or API keys that we don't want pushed into our code repository.

## Feature Branch Builds ##
---
Successful builds of feature branches are pushed to Google Container Registry (GCR) where they can be pulled down and run locally for testing or other uses.


## Master Branch Builds ##
---
Successful builds of `master` are tagged and pushed to GCR.

## Deployment ##
---
Deployment is handled by Spinnaker.

Examples of our Spinnaker Pipelines can be found in the `spinnaker` directory of this project.

### Deployment Overview ###
Spinnaker Pipeline Triggers detect updated Docker containers pushed to GCR by the Jenkins build.

A Pipeline executes the following steps:
* Pull down the newest container
    * NOTE: Do NOT rely on the __latest__ tag. For branches other than `master` the __latest__ tag represents the __most recent build__ which is NOT necessarily the container tagged __latest__.
    * We've found it more reliable to tag containers sequentially, starting with some value `N`. That way we can reliably dependon the __newest__ container being tagged `N+1`
* Deploy the container to a Temporary Pod
* Execute Smoke Tests against the Temporary Pod
    * IFF the Smoke Tests are Successful, we promote the Temporary Pod to Production
* Tear Down the Temporary Pod