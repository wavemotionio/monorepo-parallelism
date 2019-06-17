# monorepo-parallelism
Parallelism in a [Rush](https://rushjs.io/pages/intro/welcome/) Monorepo on [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/pipelines/)

## Multiple jobs
Azure DevOps hosted agents can be defined in an [azure-pipelines.yml](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema) file. A complex pipeline can contain [stages](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/stages?view=azure-devops&tabs=yaml) and [jobs](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops&tabs=yaml). Stages and jobs run in parallel by default unless dependencies are specified.

**Pros:** 
    - Theoretically, we can scale horizontally by adding as many machines as we need.
**Cons:**
	- Hosted agents can be [expensive](https://docs.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs?view=azure-devops).
	- Jobs do not share resources so we must rebuild the the code or introduce a [blob storage](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/azure-file-copy?view=azure-devops), [containers](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/container-phases?view=azure-devops&tabs=yaml), or another cloud solution in order to avoid repeat steps.

## Rush bulk commands
Rush bulk commands can be run in parallel. Parallelim adheres to the dependency graph and execution can be controlled in relation to a package by using the `-t PROJECT1` (--to) and `-f PROJECT2` (--from) command line parameters.  Out-of-the-box commands, such as `rush build` will run in parallel by default. Custom bulk commands default [`enableParallelism` to false](https://rushjs.io/pages/configs/command_line_json/).

**Pros:** 
    - Vertically scale by cores on the host machine.
**Cons:**
	- Default hosted agents only have two cores and other resource limitations which, depending on your tests, can actually *slow  down* the process.
	- The ability to run the tests in parallel will depend a lot on the quality of (properly scoped, deterministic, independent) tests and configuring your tools such as [karma](https://karma-runner.github.io/latest/index.html) and [protractor](https://www.protractortest.org/#/). 

## Alternatives
Parallelism is only one of the many ways to speed up your pipelines.

### Containers
Docker containers are another great way to augment parallelism.  You can:
1. Increase efficiency and avoid running the same setup tasks simultaneously on multiple agents by building an image and then pulling the image when each dependent agent initializes.
1. Leverage docker-compose.yml to stand up highly specialized instances of server tools such as [selenium grid](https://github.com/SeleniumHQ/docker-selenium).

### Rush version policies
The packages within a monorepository might not always meet the same software lifecycle demands in which case we need ways to differentiate groups of packages that have either grown apart or consist of very isolated concerns.

[Version policies](https://rushjs.io/pages/configs/version_policies_json/) are a great way to isolate groups of packages. You can build `--to-version-policy VERSION_POLICY_NAME` or [--bump](https://rushjs.io/pages/commands/rush_version/) and [publish](https://rushjs.io/pages/commands/rush_publish/) `--version-policy POLICY` to focus a group of components.

example: [/rush.json](https://rushjs.io/pages/configs/rush_json/)
```
"projects": [
    {
      "packageName": "tslint-config",
      "projectFolder": "tools/tslint-config",
      "versionPolicyName": "universal"
    },
    {
      "packageName": "example-logger",
      "projectFolder": "componentlibrary/visual-indicators/example-logger",
      "versionPolicyName": "universal"
    },
    {
      "packageName": "example-form",
      "projectFolder": "componentlibrary/forms/example-form",
      "versionPolicyName": "angularjsForms"
    },
    {
      "packageName": "example-input-datetime",
      "projectFolder": "componentlibrary/forms/example-input-datetime",
      "versionPolicyName": "angularjsForms"
    }
```
