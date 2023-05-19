# Jenkins Pipeline Test

This repository is to demonstrate a Jenkins Pipeline. Reading a YAML template, then updating with pipeline parameters to an output YAML.

## Setup

### Prerequisites

Plugins:

- [https://plugins.jenkins.io/pipeline-utility-steps](https://plugins.jenkins.io/pipeline-utility-steps)
- [https://plugins.jenkins.io/git](https://plugins.jenkins.io/git)

#### Credentials

Make a SSH key and add the public key to GitHub or wherever. Update the private key according to this - [https://stackoverflow.com/questions/10589976/permission-denied-public-key-during-fetch-from-github-with-jenkins-user-on-ubu](https://stackoverflow.com/questions/10589976/permission-denied-public-key-during-fetch-from-github-with-jenkins-user-on-ubu)

### Variables

Variables that are neccessary to be updated within the pipeline.

|Variable|Description|
|:--|:--|:--|
|BRANCH_NAME |main, dev, or whatever branch|
|CREDENTIAL_ID| This is the ID that is stored for the credential in Jenkins|
|REPO_URL| This is the repo url using git/ssh|

Updating the pipeline, based on the template file. Review the stucture of the YAML and update accordingly. Using the parameters that are setup with the pipeline as input. `params.cpu` is an example of a parameter that needs to be updated.

Example Template

```yaml
kind: ResourceQuota
apiVersion: v1
metadata:
  name: quota
  namespace: {{ namespace }}
  labels:
    quota-tier: {{ tier }}
  annotations:
    kubernetes.io/quota-tier: {{ tier }}
spec:
  hard:
    cpu: {{ cpu }}
    memory: {{ memory }}
    limits.cpu: {{ burstcpu }}
    limits.memory: {{ burstmem }}
  scopes:
    - NotTerminating
```

Example Pipeline

```groovy
... OMITTED ...
stage("Create New Resource Quota File") {
    steps {
        script {
            // Create config variable
            def config = readYaml file: "templates/resource_quota.yaml"

            // Update content from template; config variable
            config.metadata.namespace = params.namespace
            config.metadata.labels."quota-tier" = params.tier
            config.metadata.annotations."kubernetes.io/quota-tier" = params.tier
            config.spec.hard.cpu = params.cpu
            config.spec.hard.memory = params.memory
            config.spec.hard."limits.cpu" = params.burstcpu
            config.spec.hard."limits.memory" = params.burstmem
            
            // Write config variable updates to resource_quota.yaml file
            writeYaml file: "resource_quota.yaml", data: config

            // This isn't really necessary, just left it in for now
            def read = readYaml file: "resource_quota.yaml"
        }
      ... OMITTED ...
    }
}
```

### Pipeline

Remember to add the necessary parameters prior to executing the job.

|Name|Parameter Type | Default Value | Description|
|:--|:--|:--|:--|
|namespace|string||Enter the namespace|
|tier|choices <ul><li>small</li><li>medium</li><li>large</li><li>xlarge</li>| |Choose the tier size |
|cpu|string||Enter the CPU size|
|memory|string||Enter the Memory size|
|burstcpu|string||Enter the Burst CPU size|
|burstmem|string||Enter the Burst Memory size|

Review the [Pipeline](./Jenkinsfile) here. This example should be added to the Jenkins Pipeline job.
