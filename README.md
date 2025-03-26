# AAP Execution Environment

## Command line usage

```
ansible-builder build --container-runtime docker -t aaphub.aib.pri/ee-custom-rhel9
```


## Creating you own EE image

1. Clone this repository

2. Edit `Jenkinsfile` and change following variables as needed:

```
  environment {
    REGISTRY_CREDS = credentials('automation-hub')
    AUTOMATION_HUB_TOKEN = credentials('aaphub-dev')
    AUTOMATION_HUB_URL = 'https://aaphub-dev.aib.pri/api/galaxy/'
    REGISTRY = 'aaphub-test.aib.pri'
    IMAGE = 'ee-custom-rhel9'
  }
```

3. Change repository url to point to your own repository:

```
    stage('Clone') {
      steps {
        git branch: 'master', url: 'ssh://git@gitstash.aib.pri/itso/ee-custom-rhel9.git'
        script {
            env.GIT_COMMIT_EMAIL=sh(returnStdout: true, script: "git log --no-merges -s --format='%ae' -1")
            env.CI_VERSION = sh(returnStdout: true, script: "${WORKSPACE}/.ci/calculate-pipeline-build-number.sh")
            currentBuild.displayName = "${CI_VERSION}"
        }
      }
    }
```

4. Edit `execution-environment.yml` file and put required <mark>Collection</mark>, <mark>Python modules</mark>, <mark>System RPMs</mark> as in example below:

```
dependencies:
  ansible_core:
    package_pip: ansible-core
  ansible_runner:
    package_pip: ansible-runner
  galaxy:
    collections:
      - community.general
  python:
    - pyvmomi
    - infoblox-client
  system:
    - openssh-clients [platform:redhat]
```

5. Commit your changes to your repository.

6. Create new Jenkins pipeline.

## Examples of `execution-enviroment.yml` file

https://github.com/ansible/ansible-builder/tree/devel/docs/scenario_guides
