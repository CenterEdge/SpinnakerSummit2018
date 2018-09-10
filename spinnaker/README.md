# Sample Spinnaker Pipeline Templates

Example Pipeline Template and Config files.

## Templates ##
---
The `templates` folder contains a Spinnaker Pipeline Template.

In order for a Pipeline Template to be referenced by Spinnaker, they must be placed in a location accessible to Spinnaker.

For example, a public Github repository or Amazon Web Services S3.

The Pipeline Template has the following features:
* Variables
* Kubernetes Secrets
    * Environment variables with values pulled from Kubernetes secrets
* Multiple Stages
* Conditional Stage(s)
* Jenkins Smoke Test Stage
    * Executes a Jenkins Job and waits for a result

## Configs ##
---
The `configs` folder contains multiple configuration files that reference the template file.

Each `config` represents a Pipeline definition in Spinnaker.

Multiple `config` files can reference the same `template` and customize its behavior using `variables`.

## Deployment Overview ##
---
Spinnaker Pipeline Triggers detect updated Docker containers pushed to GCR by the Jenkins build.

A Pipeline executes the following steps:
* Pull down the newest container
    * NOTE: Do NOT rely on the __latest__ tag. For branches other than `master` the __latest__ tag represents the __most recent build__ which is NOT necessarily the container tagged __latest__.
    * We've found it more reliable to tag containers sequentially, starting with some value `N`. That way we can reliably dependon the __newest__ container being tagged `N+1`
* Deploy the container to a Temporary Pod
* Execute Smoke Tests against the Temporary Pod
    * IFF the Smoke Tests are Successful, we promote the Temporary Pod to Production
* Tear Down the Temporary Pod

