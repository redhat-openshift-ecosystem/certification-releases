# Operator Certification CI Pipeline<br/>Beta Instructions

## Table of Contents
- [Operator Certification CI Pipeline<br/>Beta Instructions](#operator-certification-ci-pipelinebeta-instructions)
  - [Table of Contents](#table-of-contents)
  - [<a id="before-you-start"></a>Before you start:](#before-you-start)
    - [<a id="what-you-need"></a>What you'll need before you start:](#what-youll-need-before-you-start)
    - [<a id="prepare-bundle"></a>Prepare your Operator Bundle before you start](#prepare-your-operator-bundle-before-you-start)
      - [Bundle structure](#bundle-structure)
      - [Fork the upstream repo](#fork-the-upstream-repo)
  - [<a id="installation"></a>Installation](#installation)
    - [<a id="step1"></a>Step 1 - Install OpenShift Pipelines Operator](#step-1---install-openshift-pipelines-operator)
    - [<a id="step2"></a>Step 2 - Configure the OpenShift CLI tool (oc)](#step-2---configure-the-openshift-cli-tool-oc)
    - [<a id="step3"></a>Step 3 - Create an OpenShift Project (namespace) to work in](#step-3---create-an-openshift-project-namespace-to-work-in)
    - [<a id="step4"></a>Step 4 - Add the Kubeconfig secret](#step-4---add-the-kubeconfig-secret)
    - [<a id="step5"></a>Step 5 - Import Red Hat Catalogs](#step-5---import-red-hat-catalogs)
    - [<a id="step6"></a>Step 6 - Install the Certification Pipeline and dependencies into the cluster](#step-6---install-the-certification-pipeline-and-dependencies-into-the-cluster)
    - [<a id="step7"></a>Step 7 - Configuration Steps for Submitting Results](#step-7---configuration-steps-for-submitting-results)
      - [<a id="github-api-token"></a>Add a GitHub API Token for the repo where the PR will be created](#add-a-github-api-token-for-the-repo-where-the-pr-will-be-created)
      - [<a id="container-api-key"></a>Add Red Hat Container API access key](#add-red-hat-container-api-access-key)
- [<a id="optional-config"></a>Optional Configuration](#optional-configuration)
  - [<a id="digest-pinning-config"></a>Optional Step - If using digest pinning (required when submitting to Red Hat)](#optional-step---if-using-digest-pinning-required-when-submitting-to-red-hat)
    - [<a id="private-key"></a>Add private key to access GitHub repo with Operator Bundle source](#add-private-key-to-access-github-repo-with-operator-bundle-source)
  - [<a id="private-registry"></a>Optional Step - If using a private container registry](#optional-step---if-using-a-private-container-registry)
    - [<a id="container-registry-creds"></a>Add credentials for the Container Registry](#add-credentials-for-the-container-registry)
- [<a id="execute-pipeline"></a>Execute the Pipeline (Development Iterations)](#execute-the-pipeline-development-iterations)
  - [<a id="minimal-pipeline-run"></a>Minimal Pipeline Run](#minimal-pipeline-run)
  - [<a id="img-digest-pipeline-run"></a>Pipeline Run with Image Digest Pinning](#pipeline-run-with-image-digest-pinning)
  - [<a id="private-registry-pipeline-run"></a>Pipeline Run with a Private Container Registry](#pipeline-run-with-a-private-container-registry)
- [<a id="submit-result"></a>Submit Results to Red Hat](#submit-results-to-red-hat)
  - [<a id="submit-result-minimal"></a>Submit results from Minimal Pipeline Run](#submit-results-from-minimal-pipeline-run)
  - [<a id="submit-result-img-digest"></a>Submit results with Image Digest Pinning](#submit-results-with-image-digest-pinning)
  - [<a id="submit-result-private-registy"></a>Submit results with a private container registry](#submit-results-with-a-private-container-registry)
  - [<a id="submit-result-registy-and-pinning"></a>Submit results with Image Digest Pinning and a private container registry](#submit-results-with-image-digest-pinning-and-a-private-container-registry)


## <a id="before-you-start"></a>Before you start:

### <a id="what-you-need"></a>What you'll need before you start:
1. An OpenShift Cluster *(recommended version 4.8 or above)*
2. Kubeconfig file for a user with cluster admin privileges (usually located at ~/.kube/config If this does not exist login to OpenShift via oc login and it will be created)
3. The contents of your Operator Bundle
4. [Install](https://docs.openshift.com/container-platform/4.8/cli_reference/openshift_cli/getting-started-cli.html#installing-openshift-cli) `oc`, the OpenShift CLI tool (tested with version 4.7.13)
5. [Install](https://tekton.dev/docs/cli/) `tkn`, the Tekton CLI tool (tested with version 0.19.1)
6. [Install](https://git-scm.com/downloads) `git`, the Git CLI tool (tested with 2.32.0)

### <a id="prepare-bundle"></a>Prepare your Operator Bundle before you start

#### Bundle structure 
The certification pipeline expects you to have the source files for your Operator bundle. The Operator bundle consists of a specific directory structure. Details about the [expected structure are documented here](https://github.com/redhat-openshift-ecosystem/certified-operators-preprod). The high level structure is as follows:

```bash
├── config.yaml
├── operators
  └── my-operator
      ├── 1.4.6
      │   ├── manifests
      │   │   ├── cache.example.com_my-operators.yaml
      │   │   ├── my-operator-controller-manager-metrics-service_v1_service.yaml
      │   │   ├── my-operator-manager-config_v1_configmap.yaml
      │   │   ├── my-operator-metrics-reader_rbac.authorization.k8s.io_v1_clusterrole.yaml
      │   │   └── my-operator.clusterserviceversion.yaml
      │   └── metadata
      │       └── annotations.yaml
      └── ci.yaml
```

*config.yaml*: This file should include the organization you are targeting for distribution of your Operator. The value should be either `certified-operators` or `redhat-marketplace`. See the example below:
``` bash
organization: certified-operators
```

*ci.yaml*: This file should include your Red Hat Technology Partner project id and the organization target for this operator. You can find instructions on where to find your project id: [here](https://github.com/redhat-openshift-ecosystem/certification-releases/blob/main/4.9/beta/pipeline-overview.md#step-a---get-project-id). 
``` bash
cert_project_id: "<your partner project id>"
```

*annotations.yaml*: This file should include an OpenShift versions annotation. *(This should be added to any existing content)*
```bash
# OpenShift annotations.
com.redhat.openshift.versions: v4.6-v4.8
```
> Note: An example Operator Bundle can be found [here](https://github.com/opdev/simple-demo-pipeline/tree/main/operators/simple-demo-operator).

#### Fork the upstream repo
Once you have the contents of your Operator bundle structured properly, 
* Log into GitHub and fork the upstream repo: https://github.com/redhat-openshift-ecosystem/certified-operators-preprod
* git clone your fork of the certified-operators-preprod repo
* Add the contents of your Operator bundle to `operators` directory in your fork


## <a id="installation"></a>Installation

### <a id="step1"></a>Step 1 - Install OpenShift Pipelines Operator
* Log into your cluster's OpenShift Console with cluster admin privileges
* Use the left hand menu to navigate to *Operators*
* In the *Operators* submenu click on *OperatorHub*
* Use the Filter/Search box to filter on *OpenShift Pipelines*
* Click the *Red Hat OpenShift Pipelines* tile
* In the flyout menu to the right click the *Install* button near the top
* On the next screen "Install Operator" scroll to the bottom of the page and click *Install*

### <a id="step2"></a>Step 2 - Configure the OpenShift CLI tool (oc)
* Open a terminal window
* Set the `KUBECONFIG` environment variable
```bash
export KUBECONFIG=/path/to/your/cluster/kubeconfig
```

Example:

```shell
export KUBECONFIG=~/.kube/config
```

> *This kubeconfig will be used to deploy the Operator under test and run the certification checks.*

### <a id="step3"></a>Step 3 - Create an OpenShift Project (namespace) to work in
```bash
oc adm new-project <my-project-name> # create the project
oc project <my-project-name> # switch into the project
```
> #### Troubleshooting Tip
>
> There are known issues running the pipeline in the `default` project/namespace. We recommend creating a project for the pipeline.

### <a id="step4"></a>Step 4 - Add the Kubeconfig secret
```bash
  oc create secret generic kubeconfig --from-file=kubeconfig=$KUBECONFIG
```
> *This kubeconfig will be used to deploy the Operator under test and run the certification checks.*

### <a id="step5"></a>Step 5 - Import Red Hat Catalogs
```bash
oc import-image certified-operator-index \
  --from=registry.redhat.io/redhat/certified-operator-index \
  --reference-policy local \
  --scheduled \
  --confirm \
  --all
```

```bash
oc import-image redhat-marketplace-index \
  --from=registry.redhat.io/redhat/redhat-marketplace-index \
  --reference-policy local \
  --scheduled \
  --confirm \
  --all
```

### <a id="step6"></a>Step 6 - Install the Certification Pipeline and dependencies into the cluster
```bash
git clone https://github.com/redhat-openshift-ecosystem/operator-pipelines
cd operator-pipelines
oc apply -R -f ansible/roles/operator-pipeline/templates/openshift/pipelines
oc apply -R -f ansible/roles/operator-pipeline/templates/openshift/tasks
```
### <a id="step7"></a>Step 7 - Configuration Steps for Submitting Results

#### <a id="github-api-token"></a>Add a GitHub API Token for the repo where the PR will be created

Upon completion the Pipeline can automatically open a Pull Request to submit your Operator to Red Hat. To enable this functionally, add a GitHub API Token and use `--param submit=true` when running the pipeline. 

```bash
oc create secret generic github-api-token --from-literal GITHUB_TOKEN=<github token>
```

#### <a id="container-api-key"></a>Add Red Hat Container API access key
You will receive a specific API access key to use for this step.
```bash
oc create secret generic pyxis-api-secret --from-literal pyxis_api_key=< API KEY >
```

# <a id="optional-config"></a>Optional Configuration

## <a id="digest-pinning-config"></a>Optional Step - If using digest pinning (required when submitting to Red Hat)
The pipeline offers functionality that will automatically replace all image tags in your Bundle with Image Digest SHAs.This allows the pipeline to ensure it is using a pinned version of all images. The pipeline commits the pinned version of your Bundle to your GitHub repo as a new branch.  In order to do this a private key with access to GitHub needs to be added to your cluster as a secret. 

### <a id="private-key"></a>Add private key to access GitHub repo with Operator Bundle source
Base64 encode a private key with access to GitHub repo containing the Bundle
```bash
base64 /path/to/private/key
```

Create a secret that contains the base64 encoded private key
```bash
cat << EOF > ssh-secret.yml
kind: Secret
apiVersion: v1
metadata:
  name: github-ssh-credentials
data:
  id_rsa: |
        <base64 encoded private key>
EOF
```

Add the secret to the cluster
```bash
oc create -f ssh-secret.yml
```



## <a id="private-registry"></a>Optional Step - If using a private container registry
By default the Pipeline will use the OpenShift Container Registry running in the cluster.  If you would like to use a private registry you will need to add credentials for that registry.

### <a id="container-registry-creds"></a>Add credentials for the Container Registry
The Pipeline will automatically build your Operator Bundle Image as well as a Bundle Image Index for testing and verification.  

By default these images will be created in the container registry on the cluster but if you would like to leverage an external private registry you can provide the credentials by adding a secret to the cluster. 

```bash
oc create secret docker-registry registry-dockerconfig-secret \
    --docker-server=quay.io \
    --docker-username=<registry username> \
    --docker-password=<registry password> \
    --docker-email=<registry email>
```





# <a id="execute-pipeline"></a>Execute the Pipeline (Development Iterations)
There are multiple ways to execute the Pipeline.  Below are several examples but parameters and workspaces can be removed or added per your requirements. 

## <a id="minimal-pipeline-run"></a>Minimal Pipeline Run

```bash
GIT_REPO_URL=<Git URL to your certified-operators-preprod fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=stage \
  --param bundle_path=$BUNDLE_PATH \
  --param env=stage \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=kubeconfig,secret=kubeconfig \
  --showlog
```
After running this command you will be prompted for several additional parameters. Accept all the defaults. 

> #### Troubleshooting Tip
>
> If you see a Permission Denied error try the GITHUB `HTTPS URL` instead of the `SSH URL`. 

## <a id="img-digest-pipeline-run"></a>Pipeline Run with Image Digest Pinning
* Execute the [Configuration Steps for Digest Pinning](#digest-pinning-config)

```bash
GIT_REPO_URL=<Git URL to your certified-operators-preprod fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=stage \
  --param bundle_path=$BUNDLE_PATH \
  --param env=stage \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=kubeconfig,secret=kubeconfig \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
  --showlog
```
> #### Troubleshooting Tip
>
> ##### Error: could not read Username for `https://github.com` 
> try using the SSH GitHub URL in `--param git_repo_url`

## <a id="private-registry-pipeline-run"></a>Pipeline Run with a Private Container Registry
* Execute the [Configuration Steps for Private Registries](#private-registry)
```bash
GIT_REPO_URL=<Git URL to your certified-operators-preprod fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
REGISTRY=<your image registry.  ie: quay.io>
IMAGE_NAMESPACE=<namespace in the container registry>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=stage \
  --param bundle_path=$BUNDLE_PATH \
  --param env=stage \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --param registry=$REGISTRY \
  --param image_namespace=$IMAGE_NAMESPACE \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=kubeconfig,secret=kubeconfig \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
  --workspace name=registry-credentials,secret=registry-dockerconfig-secret \
  --showlog \

```

# <a id="submit-result"></a>Submit Results to Red Hat
* Execute the [Configuration Steps for Submitting Results](#step7)

In order to submit results add the following `--param`'s and `--workspace` where `$UPSTREAM_REPO_NAME` is equal to the repo where the Pull Request will be submitted. Typically this is a Red Hat Certification repo but you can use a repo of your own for testing.
```bash
--param upstream_repo_name=$UPSTREAM_REPO_NAME #Repo where Pull Request (PR) will be opened
```

```bash
--param submit=true
```

```bash
--workspace name=pyxis-api-key,secret=pyxis-api-secret
```

## <a id="submit-result-minimal"></a>Submit results from Minimal Pipeline Run
```bash
GIT_REPO_URL=<Git URL to your certified-operators-preprod fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=stage \
  --param bundle_path=$BUNDLE_PATH \
  --param upstream_repo_name=redhat-openshift-ecosystem/certified-operators-preprod \
  --param submit=true \
  --param env=stage \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=kubeconfig,secret=kubeconfig \
  --workspace name=pyxis-api-key,secret=pyxis-api-secret \
  --showlog
```

## <a id="submit-result-img-digest"></a>Submit results with Image Digest Pinning
* Execute the [Configuration Steps for Submitting Results](#step7)
* Execute the [Configuration Steps for Digest Pinning](#digest-pinning-config)

```bash
GIT_REPO_URL=<Git URL to your certified-operators-preprod fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=stage \
  --param bundle_path=$BUNDLE_PATH \
  --param env=stage \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --param upstream_repo_name=redhat-openshift-ecosystem/certified-operators-preprod \
  --param submit=true \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=kubeconfig,secret=kubeconfig \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
  --workspace name=pyxis-api-key,secret=pyxis-api-secret \
  --showlog
```
> #### Troubleshooting Tip
>
> ##### Error: could not read Username for `https://github.com` 
> try using the SSH GitHub URL in `--param git_repo_url`

## <a id="submit-result-private-registy"></a>Submit results with a private container registry
* Execute the [Configuration Steps for Submitting Results](#step7)
* Execute the [Configuration Steps for Private Registries](#private-registry)
```bash
GIT_REPO_URL=<Git URL to your certified-operators-preprod fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
REGISTRY=<your image registry.  ie: quay.io>
IMAGE_NAMESPACE=<namespace in the container registry>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=stage \
  --param bundle_path=$BUNDLE_PATH \
  --param env=stage \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --param registry=$REGISTRY \
  --param image_namespace=$IMAGE_NAMESPACE \
  --param upstream_repo_name=redhat-openshift-ecosystem/certified-operators-preprod \
  --param submit=true \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=kubeconfig,secret=kubeconfig \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
  --workspace name=registry-credentials,secret=registry-dockerconfig-secret \
  --workspace name=pyxis-api-key,secret=pyxis-api-secret \
  --showlog
```

## <a id="submit-result-registy-and-pinning"></a>Submit results with Image Digest Pinning and a private container registry
* Execute the [Configuration Steps for Submitting Results](#step7)
* Execute the [Configuration Steps for Digest Pinning](#digest-pinning-config)
* Execute the [Configuration Steps for Private Registries](#private-registry)

```bash
GIT_REPO_URL=<Git URL to your certified-operators-preprod fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
REGISTRY=<your image registry.  ie: quay.io>
IMAGE_NAMESPACE=<namespace in the container registry>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=stage \
  --param bundle_path=$BUNDLE_PATH \
  --param env=stage \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --param upstream_repo_name=redhat-openshift-ecosystem/certified-operators-preprod \
  --param registry=$REGISTRY \
  --param image_namespace=$IMAGE_NAMESPACE \
  --param submit=true \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=kubeconfig,secret=kubeconfig \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
  --workspace name=registry-credentials,secret=registry-dockerconfig-secret \
  --workspace name=pyxis-api-key,secret=pyxis-api-secret \
  --showlog
```
