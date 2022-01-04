# Troubleshooting the Operator Cert Pipeline

## Table of Contents

### [Manual Pull Request (Red Hat Tested)](#manual)
* [set-github-status-pending](#set-github-status-pending)
* [set-env](#set-env)
* [checkout](#checkout)
* [validate-pr-title](#validate-pr-title)
* [get-bundle-path](#get-bundle-path)
* [bundle-path-validation](#bundle-path-validation)
* [certification-project-check](#certification-project-check)
* [content-hash](#content-hash)
* [create-support-link-for-pr](#create-support-link-for-pr)
* [get-cert-project-related-data](#get-cert-project-related-data)
* [submission-validation](#submission-validation)
* [merge-registry-credentials](#merge-registry-credentials)
* [update-cert-project-status](#update-cert-project-status)
* [reserve-operator-name](#reserve-operator-name)
* [get-supported-versions](#get-supported-versions)
* [annotations-validation](#annotations-validation)
* [digest-pinning](#digest-pinning)
* [verify-changed-directories](#verify-changed-directories)
* [yaml-lint](#yaml-lint)
* [verify-pinned-digest](#verify-pinned-digest)
* [dockerfile-creation](#dockerfile-creation)
* [build-bundle](#build-bundle)
* [generate-index](#generate-index)
* [make-bundle-repo-public](#make-bundle-repo-public)
* [build-index](#build-index)
* [make-index-repo-public](#make-index-repo-public)
* [get-ci-results-attempt](#get-ci-results-attempt)
* [preflight-trigger](#preflight-trigger)
* [upload-artifacts](#upload-artifacts])
* [get-ci-results](#get-ci-results)
* [link-pull-request](#link-pull-request)
* [verify-ci-results](#verify-ci-results)
* [query-publishing-checklist](#query-publishing-checklist)
* [merge-pr](#merge-pr)

### [Automated Pull Request (Tested on Partner Premise)](#automated)
* [Use the latest Pipeline code](#latest-code)
* [Fork the Production GitHub repo](#fork-prod)
* [Get a clean kubeconfig](#clean-kubeconfig)
* [Digest pinning fatal: could not read Username](#gh-username)
* [Support multiple container registries / Registry access errors](#multiple-registries)
* [Cannot find CSV](#cannot-find-csv)
* [Get a Red Hat Registry Service Account token](#fork-prod)
* [Verify Pinned Digest Failed](#pin-failed)


# <a id="manual"></a>Manual Pull Request *(Red Hat Tested)*

## <a id="set-github-status-pending"></a>set-github-status-pending
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="set-env"></a>set-env
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="checkout"></a>checkout
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="validate-pr-title"></a>validate-pr-title
### <a id="pr-title"></a>Pull Request Title
When creating a pull request manually the title of your pull request must follow a predefined format. 

| Prefix | Package Name | Version |
|--------|--------------|---------|
| The word `operator` | Operator package name | Version in parenthesis. A `v` prefix is suggested.|

> Note: The version in your PR Title must match the version directory in your Operator Bundle.  
> If you use the `v` prefix in your PR title it must also be used when naming your version directory. 

### Examples
`operator simple-demo-operator (v0.0.0)`

`operator hello-world-certified (v.1.2.3)`

`operator my-operator (3.2.1)`



## <a id="get-bundle-path"></a>get-bundle-path
Failures at this step are uncommon. If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="bundle-path-validation"></a>bundle-path-validation
Failures at this step are uncommon. Please reference [this step](#validate-pr-title) to ensure your package name is consistent with your PR title. Also, please ensure your package name matches your Operator folder name. If the package name is consistent and you still experience a failure or error at this step, contact Red Hat Support.

## <a id="certification-project-check"></a>certification-project-check
This step confirms that your `ci.yaml` contains a `cert_project_id` value. 

## <a id="content-hash"></a>content-hash
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="create-support-link-for-pr"></a>create-support-link-for-pr
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="get-cert-project-related-data"></a>get-cert-project-related-data
Make sure the `cert_project_id` in your `ci.yaml` file is formatted correctly and references the right project in your Partner Connect account. 

`cert_project_id: 6804256accf2367227abc887612ffc5567`

> Note: Do not include the `ospid-` prefix and remove any dashes/hyphens 

Make sure you have included the Authorized GitHub Usernames in Connect
See more detail [here](#auth-gh-users).

## <a id="submission-validation"></a>submission-validation
You may only have one open Pull Request at a time. Attempting to open a second Pull request while one is still open will result in a failure here. 

### <a id="auth-gh-users"></a>Authorized GitHub Usernames
Any GitHub `username` or GitHub `organization` used to submit a pull request must be entered in the GitHub Authorized Users field on the Project settings page in connect.redhat.com.

| Fork URL | User or Org |
|----------|-------------|
| `https://github.com/my-github-user/certified-operators.git` | my-github-user |
| `https://github.com/my-github-organization/certified-operators.git` | my-github-organization |
| `https://github.com/my-github-user/redhat-marketplace-operators.git` | my-github-user |
| `https://github.com/my-github-organization/redhat-marketplace-operators.git` | my-github-organization |

Once you have your GitHub username or organization identified follow the instructions below to add it to Connect

1. Navigate to [connect.redhat.com](https://connect.redhat.com/)
2. Click the `Login` button
3. Click the `Log in for technology partners` button
4. Click `Product Certification` > `Manage certification projects`
5. Click on the Project link for your Operator Bundle Image
6. Click on the `settings` tab
7. Add your GitHub users/organizations to the `Authorized GitHub user accounts` field. 

![Auth GH Users](assets/AuthGHUsers.png)

## <a id="merge-registry-credentials"></a>merge-registry-credentials
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="update-cert-project-status"></a>update-cert-project-status
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="reserve-operator-name"></a>reserve-operator-name
This step will make sure that the package name of our Operator belongs to your `cert_project_id`. This step will fail if another `cert_project_id` has laid claim to the package name used in this Pull Request. 

## <a id="get-supported-versions"></a>get-supported-versions
Failures at this step may be caused by several issues:

> If you also have a failure at the `annotations-validation` step, you should [resolve](#annotation-validation) the `annotation-validation` issues first and then retry. It's possible that this will resolve failures at this step as well. 

* Your `annotations.yaml` file needs to include an `com.redhat.openshift.versions` annotation indicating the OpenShift versions supported by your Operator.  
* The filename for your `clusterserviceversion.yaml` must be prefixed with your Operator's package name. 
* The value of your `operators.operatorframework.io.bundle.package.v1` annotation in the `annotations.yaml` must equal your Operator's package name. 

## <a id="annotations-validation"></a>annotations-validation
The version used in your Pull Request title will be the version directory used to look up your `annotations.yaml` file. A common mistake is to use a `v` in the version of your PR title without also using a `v` prefix for the version folder in your Operator Bundle. 

| Pull Request Title | Version directory name |
|------------------------------------------|----------|
| `operator simple-demo-operator (v1.2.3)` | `v1.2.3` |
| `operator simple-demo-operator (1.2.3)`  | `1.2.3`  |

Also please ensure to double check that your package naming remains consistent throughout the metadata code, particularly in the annotations.yaml file.


## <a id="digest-pinning"></a>digest-pinning

> If you also have a failure at the `annotations-validation` step, you should [resolve](#annotation-validation) the `annotation-validation` issues first and then retry. It's possible that this will resolve failures at this step as well. 

### <a id="pinning"></a>
All images referenced in your Operator Bundle must reference SHA digest and not tags. The existence of tags in your bundle will cause a certification failure Replace all image tags with image digests. 

| Unpinned Example | Pinned Example |
|----------|--------|
| `quay.io/my_repo/my_image:v1.0.0` | `quay.io/my_repo/my_image@sha256:fd8d827d4d345ec327cb92d30086a17a2e08ba9c3163db4a25bfe2512123fd6a` |

In addition your clusterserviceversion.yaml must include a `relatedImages` section. This section should implement a format similar to the one below. 

```
...
spec:
  relatedImages: 
    - name: etcd-operator 
      image: quay.io/etcd-operator/operator@sha256:d134a9865524c29fcf75bbc4469013bc38d8a15cb5f41acfddb6b9e492f556e4 
    - name: etcd-image
      image: quay.io/etcd-operator/etcd@sha256:13348c15263bd8838ec1d5fc4550ede9860fcbb0f843e48cbccec07810eebb68
...
```

## <a id="verify-changed-directories"></a>verify-changed-directories
Your Pull Request should only add files and not modify any files that have already been merged.  Make sure files changed reside in a single version directory that matches the version used in the title of your Pull Request

## <a id="yaml-lint"></a>yaml-lint
> If you also have a failure at the `annotations-validation` step, you should [resolve](#annotation-validation) the `annotation-validation` issues first and then retry. It's possible that this will resolve failures at this step as well. 

**Warnings** at this step should be addressed if possible but won't result in a failure.  
**Errors** at this step will need to be addressed.  Often errors center around unexpected whitespace at the end of lines or missing newlines at the end of your `yaml` files. 

## <a id="verify-pinned-digest"></a>verify-pinned-digest
See [Digest Pinning](#digest-pinning)

This step checks to ensure that all your container images are using SHA digests instead of tags.  
This step also checks for the existence of a `spec.relatedImages` section in your Cluster Service Version (CSV). 

More information on formatting `clusterserviceversion.yaml` files can be found [here](https://docs.openshift.com/container-platform/4.9/operators/operator_sdk/osdk-generating-csvs.html).

## <a id="dockerfile-creation"></a>dockerfile-creation
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="build-bundle"></a>build-bundle
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="generate-index"></a>generate-index
Failures at this step are uncommon.  

If you are updating your Operator to be compatible with OpenShift v4.9, and your Operator was previously removed from the 4.9 Operator index, be sure you are following the guidance outlined [here](https://connect.redhat.com/blog/updating-operators-openshift-49). A common cause of error is to use 'replaces' to replace a version of your Operator that was removed from the index.

If you still experience a failure or error at this step, contact Red Hat Support.

## <a id="make-bundle-repo-public"></a>make-bundle-repo-public
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="build-index"></a>build-index
Failures at this step are uncommon and if they do occur they are often transient.  

Please click the `Close pull request` button in GitHub then click the `Reopen pull request` button.
Closing and re-opening your Pull request will restart the Pipeline. If your PR fails at this step twice in a row please contact Red Hat Support

## <a id="make-index-repo-public"></a>make-index-repo-public
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="get-ci-results-attempt"></a>get-ci-results-attempt
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="preflight-trigger"></a>preflight-trigger
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

> Note: There is a known issue if you Operator only supports OpenShift 4.7 or below. In this case we recommend using the CI Pipeline.

## <a id="upload-artifacts"></a>upload-artifacts
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="get-ci-results"></a>get-ci-results
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="link-pull-request"></a>link-pull-request
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.

## <a id="verify-ci-results"></a>verify-ci-results
Issues at this step typically point to failures in the Preflight checks which confirm your Operators adherence to the certification policy. There may be additional logs available at connect.redhat.com listed on the Test Results page for your Project. 

Failures here may be caused by multiple issues
* In your CSV (clusterserviceversion.yaml) ensure that your `alm-examples` annotation is correct. Look out for incorrect spacing or tabs which may inadvertently include other annotations under `alm-examples`
* In `alm-examples` make sure all your CRs have a `spec` block.
* Failure here may indicate that we were unable to deploy your Operator using the Operator Lifecycle Manager (OLM)
* Ensure all of your related images are certified and published

## <a id="query-publishing-checklist"></a>query-publishing-checklist
Failures here usually point to an incomplete Checklist item.  Please login to connect.redhat.com and complete all the items listed under the Project publishing checklist. 

## <a id="merge-pr"></a>merge-pr
Failures at this step are uncommon.  If you do experience a failure or error at this step, contact Red Hat Support.










<hr>



## <a id="pkg-name"></a>Package Name
Your Operator's package name must be used consistently in three areas

| Package |
|--------------|
| Name of your folder under the `operators` directory in your fork |
| Value of the `operators.operatorframework.io.bundle.package.v1` annotation in `annotations.yaml` |
| Prefix of the filename for your `clusterserviceversion.yaml` file |



# <a id="automated"></a>Automated Pull Request *(Tested on Partner Premise)*

## <a id="latest-code"></a>Make sure you are using the latest version of the Pipeline
As the Pipeline is updated with fixes and enhancements you want to make sure you are using the latest version. 

### Soft reload of the Pipeline using `oc apply`
```bash
git clone https://github.com/redhat-openshift-ecosystem/operator-pipelines
cd operator-pipelines
oc apply -R -f ansible/roles/operator-pipeline/templates/openshift/pipelines
oc apply -R -f ansible/roles/operator-pipeline/templates/openshift/tasks
```

### Hard reload of the Pipeline using `oc delete` and `oc create`
```bash
git clone https://github.com/redhat-openshift-ecosystem/operator-pipelines
cd operator-pipelines
oc delete -R -f ansible/roles/operator-pipeline/templates/openshift/pipelines
oc delete -R -f ansible/roles/operator-pipeline/templates/openshift/tasks
oc create -R -f ansible/roles/operator-pipeline/templates/openshift/pipelines
oc create -R -f ansible/roles/operator-pipeline/templates/openshift/tasks
```

## <a id="fork-prod"></a>Make sure we are using Production
Partners who were a part of the alpha testing and beta testing may still be using GitHub forks of the `preprod` repos.  Make sure you are now using the production repo for your GitHub fork. 

> Prod Repo is https://github.com/redhat-openshift-ecosystem/certified-operators

In addition your `tkn` command should reference the production environments

```
... any other params
--param git_repo_url=<your fork of the prod repo>
--param git_branch=main
--param env=prod
--param upstream_repo_name=redhat-openshift-ecosystem/certified-operators
... any other params

```





## <a id="clean-kubeconfig"></a>Get a clean kubeconfig
The Kubeconfig you are using may contain multiple contexts with certs for multiple clusters. If you try to use that kubeconfig as is it may cause issues on your OpenShift cluster. To make sure you are using a clean kubeconfig targeting your Openshift CI Cluster use the script below. 

```bash
oc config view --flatten > new-kubeconfig
export KUBECONFIG=new-kubeconfig
oc delete secret kubeconfig
oc create secret generic kubeconfig --from-file=kubeconfig=$KUBECONFIG
```

## <a id="gh-username"></a>Digest pinning `fatal: could not read Username for 'https://github.com': No such device or address`
> Use GitHub SSH URL or set .gitconfig to enforce SSH

## <a id="multiple-registries"></a>Add access to multiple container registries. 
If you are leveraging multiple container registries you will likely need to provide credentials for each. The sample script below can be modified to accommodate X number of registries

This issue is most often seen in the Digest Pinning task and the Build Index task. If you are seeing errors indicating registry access issues, the following might resolve it. 

```bash
#! /bin/bash

CRED1=$(echo -n "myuser1:mypasswd1" | base64)
CRED2=$(echo -n "myuser2:mypasswd2" | base64)
CRED3=$(echo -n "myuser3:mypasswd3" | base64)


cat << EOF > auth.yml
{
        "registry1": {
                "auth": $CRED1
        },
        "registry2": {
                "auth": $CRED2
        },
        "registry3": {
                "auth": $CRED3
        }

}
EOF

CONFIGJSON=$(base64 ./auth.yml)

cat << EOF > regcred.yml
apiVersion: v1
kind: Secret
metadata:
  name: registry-dockerconfig-secret
data:
  .dockerconfigjson: $CONFIGJSON
type: kubernetes.io/dockerconfigjson
EOF

#########################################
## If adding to a local OpenShift Cluster
# oc create -f regcred.yml
##########################################
# If adding to connect.redhat.com 
# copy the contents of the regcred.yml to the 
# OpenShift Object YAML field on the Project settings page
```

## <a id="cannot-find-csv"></a>Cannot find CSV file
Make sure the `package` annotation in your `annotations.yaml` file matches the prefix of the clusterserviceversion.yaml filename.  

For example if my Operator is called `simple-demo-operator` then the following should be set to:
1. metadata/annotations.yaml
```bash
...
operators.operatorframework.io.bundle.package.v1: simple-demo-operator**
...
```

2. manifests/**simple-demo-operator**.clusterserviceversion.yaml

## Get a Red Hat registry service account token
### Instructions

https://access.redhat.com/RegistryAuthentication#creating-registry-service-accounts-6

### Create a Registry Service Account
https://access.redhat.com/terms-based-registry/

Upon creating a Registry Service Account you will be given a username similar to the one below

`username`: `123456789:my-sa-acctname`

And a token.

`eyJhbGc.................`

With the username and the token as your password you can follow the instructions for [supporting multiple registries](#multiple-registries). 


## <a id="pin-failed"></a>Verify Pinned Digest Step Fails 
Digest pinning will create a pinned version of your CSV that uses SHA Digest for all images. Your manually pinned CSV must match exactly, as tested by `git diff --stat`, what is created by the Digest Pinning tool. Sometimes the Digest Pinning Tool may add duplicate entries using different names in the relatedImages section. 

If you are using the CI Pipeline adding `--param pin_digest=true` will avoid this issue.  If you are submitting a PR manually you may hit this issue. 

## 404 Error when attempting to open a PR with the CI Pipeline
* Make sure you have a secret named github-api-secret that contains a GitHub personal access token
* Make sure the GitHub personal access token has the `Repo` scope selected, which should also select all scopes under `Repo`
``` 
repo Full control of private repositories
        repo:status Access commit status
        repo_deployment Access deployment status
        public_repo Access public repositories
        repo:invite Access repository invitations
        security_events Read and write security events
 ```
