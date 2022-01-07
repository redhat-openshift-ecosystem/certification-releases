# Operator Certification CI Pipeline<br/>Instructions
## Get Help:
> Technology Partner Success Desk is a service  for all our technology partners where they can ask technical and non-technical questions pertaining to Red Hat offerings, programs, engagement processes, etc. If you run into any issues throughout these instructions please reach out to the Technology Partner Success Desk. 
>
> You can access the Success Desk by going to: [Red Hat Help Request](https://connect.redhat.com/support/technology-partner/#/). 

## Table of Contents
* [Before you start](#before-you-start)
  * [What you will need](#what-you-need)
  * [Preparing your Operator Bundle](#prepare-bundle)
* [Installation](#installation)
  * [Step 1 - Install OpenShift Pipelines Operator](#step1) 
  * [Step 2 - Configure the OpenShift CLI tool](#step2)
  * [Step 3 - Create an OpenShift Project (namespace) to work in](#step3)
  * [Step 4 - Add the Kubeconfig secret](#step4)
  * [Step 5 - Import Red Hat Catalogs](#step5)
  * [Step 6 - Install the Certification Pipeline and dependencies into the cluster](#step6)
  * [Step 7 - Configuration Steps for Submitting Results](#step7)
* [Optional configuration](#optional-config)
  * [Optional Step - If using digest pinning (required for submission)](#digest-pinning-config)
  * [Optional Step - If using a private container registry](#private-registry)
* [Pipeline Execution (Development/Iteration)](#execute-pipeline)
  * [Minimal Pipeline Run](#minimal-pipeline-run)
  * [Pipeline Run with Image Digest Pinning](#img-digest-pipeline-run)
  * [Pipeline Run with a Private Container Registry](#private-registry-pipeline-run)
* [Submit Results to Red Hat](#submit-result)
  * [Submit results from Minimal Pipeline Run](#submit-result-minimal)
  * [Submit results with Image Digest Pinning](#submit-result-img-digest)
  * [Submit results with a private container registry](#submit-result-private-registy)
  * [Submit results with Image Digest Pinning and a private container registry](#submit-result-registy-and-pinning)

## <a id="before-you-start"></a>Before you start:

### <a id="what-you-need"></a>What you'll need before you start:
1. An OpenShift Cluster *(recommended version 4.8 or above)*
> Note: The CI Pipeline will make a Persistent Volume claim for a 5GB volume. If you are running an [OpenShift cluster on bare metal](https://docs.openshift.com/container-platform/4.9/installing/installing_bare_metal/installing-bare-metal.html), ensure you have configured [Dynamic Volume Provisioning](https://docs.openshift.com/container-platform/4.9/storage/dynamic-provisioning.html). If you do not have Dynamic Volume Provisioning configured, consider setting up a [Local Volume](https://docs.openshift.com/container-platform/4.9/storage/persistent_storage/persistent-storage-local.html). The Local Volume storage path must have the `container_file_t` SELinux label to avoid Permission Denied errors, i.e. `chcon -Rv -t container_file_t "storage_path(/.*)?"`.

2. Kubeconfig file for a user with **cluster admin privileges**
4. The contents of your Operator Bundle
5. [Install](https://docs.openshift.com/container-platform/4.8/cli_reference/openshift_cli/getting-started-cli.html#installing-openshift-cli) `oc`, the OpenShift CLI tool (tested with version 4.7.13)
6. [Install](https://tekton.dev/docs/cli/) `tkn`, the Tekton CLI tool (tested with version 0.19.1)
7. [Install](https://git-scm.com/downloads) `git`, the Git CLI tool (tested with 2.32.0)

### <a id="prepare-bundle"></a>Prepare your Operator Bundle before you start

#### Bundle structure 
The certification pipeline expects you to have the source files for your Operator bundle. The Operator bundle consists of a specific directory structure. Details about the [expected structure are documented here](https://github.com/redhat-openshift-ecosystem/certified-operators). The high level structure is as follows:

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

Please Note: If you are targeting your Operator for Red Hat Marketplace distribution, you must include the following annotations in your clusterserviceversion.yaml:
``` bash
marketplace.openshift.io/remote-workflow: https://marketplace.redhat.com/en-us/operators/
{package_name}/pricing?utm_source=openshift_console
 
marketplace.openshift.io/support-workflow:https://marketplace.redhat.com/en-us/operators/\{package_name}/support?utm_source=openshift_console
```
If you are updating an existing Red Hat Marketplace Operator bundle in the [RHM Operator repository](https://github.com/redhat-openshift-ecosystem/redhat-marketplace-operators), your package name must be consistent with the existing folder name you see for your Operator. For Marketplace bundles, you will need to manually add `-rhmp` to your package name. Previously, this was done automatically and therefore will not impact customer upgrades when manually changed. 

*ci.yaml*: This file should include your Red Hat Technology Partner project id and the organization target for this operator. 

You can find instructions on where to find your project id: [here](https://github.com/redhat-openshift-ecosystem/certification-releases/blob/main/4.9/ga/operator-cert-workflow.md#step-a---get-project-id). 
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
* Log into GitHub and fork the upstream repo: https://github.com/redhat-openshift-ecosystem/certified-operators
* git clone your fork of the certified-operators repo
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

This GitHub personal access token should have the `Repo` scope added and all the sub-scopes under `Repo`.  Scopes can be added from the GitHub user interface. 

#### <a id="container-api-key"></a>Add Red Hat Container API access key
This API access key is specifically related to your unique partner account for Red Hat Connect portal. Instructions to obtain your API key can be found: [here](https://github.com/redhat-openshift-ecosystem/certification-releases/blob/main/4.9/ga/operator-cert-workflow.md#step-b---get-api-key). 
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
By default the Pipeline will use the OpenShift Container Registry running in the cluster. 

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
GIT_REPO_URL=<Git URL to your certified-operators fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=main \
  --param bundle_path=$BUNDLE_PATH \
  --param env=prod \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --showlog
```
After running this command you will be prompted for several additional parameters. Accept all the defaults. 

The following is set as default and doesn't need to be explicitly included, but can be overridden if your kubeconfig secret is created under a different name.
```bash
--param kubeconfig_secret_name=kubeconfig \
--param kubeconfig_secret_key=kubeconfig
```

> #### Troubleshooting Tip
>
> If you see a Permission Denied error try the GITHUB `HTTPS URL` instead of the `SSH URL`. 

## <a id="img-digest-pipeline-run"></a>Pipeline Run with Image Digest Pinning
* Execute the [Configuration Steps for Digest Pinning](#digest-pinning-config)

```bash
GIT_REPO_URL=<Git URL to your certified-operators fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=main \
  --param bundle_path=$BUNDLE_PATH \
  --param env=prod \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
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
GIT_REPO_URL=<Git URL to your certified-operators fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
REGISTRY=<your image registry.  ie: quay.io>
IMAGE_NAMESPACE=<namespace in the container registry>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=main \
  --param bundle_path=$BUNDLE_PATH \
  --param env=prod \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --param registry=$REGISTRY \
  --param image_namespace=$IMAGE_NAMESPACE \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
  --workspace name=registry-credentials,secret=registry-dockerconfig-secret \
  --showlog

```

# <a id="submit-result"></a>Submit Results to Red Hat
* Execute the [Configuration Steps for Submitting Results](#step7)

In order to submit results add the following `--param`'s where `$UPSTREAM_REPO_NAME` is equal to the repo where the Pull Request will be submitted. Typically this is a Red Hat Certification repo but you can use a repo of your own for testing.
```bash
--param upstream_repo_name=$UPSTREAM_REPO_NAME #Repo where Pull Request (PR) will be opened
```

```bash
--param submit=true
```

The following is set as default and doesn't need to be explicitly included, but can be overridden if your Pyxis secret is created under a different name.
```bash
--param pyxis_api_key_secret_name=pyxis-api-secret \
--param pyxis_api_key_secret_key=pyxis-api-key
```

## <a id="submit-result-minimal"></a>Submit results from Minimal Pipeline Run
```bash
GIT_REPO_URL=<Git URL to your certified-operators fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=main \
  --param bundle_path=$BUNDLE_PATH \
  --param upstream_repo_name=redhat-openshift-ecosystem/certified-operators \
  --param submit=true \
  --param env=prod \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --showlog
```

## <a id="submit-result-img-digest"></a>Submit results with Image Digest Pinning
* Execute the [Configuration Steps for Submitting Results](#step7)
* Execute the [Configuration Steps for Digest Pinning](#digest-pinning-config)

```bash
GIT_REPO_URL=<Git URL to your certified-operators fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=main \
  --param bundle_path=$BUNDLE_PATH \
  --param env=prod \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --param upstream_repo_name=redhat-openshift-ecosystem/certified-operators \
  --param submit=true \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
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
GIT_REPO_URL=<Git URL to your certified-operators fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
REGISTRY=<your image registry.  ie: quay.io>
IMAGE_NAMESPACE=<namespace in the container registry>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=main \
  --param bundle_path=$BUNDLE_PATH \
  --param env=prod \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --param registry=$REGISTRY \
  --param image_namespace=$IMAGE_NAMESPACE \
  --param upstream_repo_name=redhat-openshift-ecosystem/certified-operators \
  --param submit=true \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
  --workspace name=registry-credentials,secret=registry-dockerconfig-secret \
  --showlog
```

## <a id="submit-result-registy-and-pinning"></a>Submit results with Image Digest Pinning and a private container registry
* Execute the [Configuration Steps for Submitting Results](#step7)
* Execute the [Configuration Steps for Digest Pinning](#digest-pinning-config)
* Execute the [Configuration Steps for Private Registries](#private-registry)

```bash
GIT_REPO_URL=<Git URL to your certified-operators fork >
BUNDLE_PATH=<path to the bundle in the Git Repo> (ie: operators/my-operator/1.2.8)
GIT_USERNAME=<your github username>
GIT_EMAIL=<your github email address>
REGISTRY=<your image registry.  ie: quay.io>
IMAGE_NAMESPACE=<namespace in the container registry>
```

```bash
tkn pipeline start operator-ci-pipeline \
  --param git_repo_url=$GIT_REPO_URL \
  --param git_branch=main \
  --param bundle_path=$BUNDLE_PATH \
  --param env=prod \
  --param pin_digests=true \
  --param git_username=$GIT_USERNAME \
  --param git_email=$GIT_EMAIL \
  --param upstream_repo_name=redhat-openshift-ecosystem/certified-operators \
  --param registry=$REGISTRY \
  --param image_namespace=$IMAGE_NAMESPACE \
  --param submit=true \
  --workspace name=pipeline,volumeClaimTemplateFile=templates/workspace-template.yml \
  --workspace name=ssh-dir,secret=github-ssh-credentials \
  --workspace name=registry-credentials,secret=registry-dockerconfig-secret \
  --showlog
```
