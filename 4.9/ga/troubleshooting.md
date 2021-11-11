# Troubleshooting the Operator Cert Pipeline

## Table of Contents


### [Manual Pull Request (Red Hat Tested)](#manual)
* [PR Title](#pr-title)
* [Authorized GitHub Usernames](#auth-gh-users)
* [Package Name](#pkg-name)
* [Digest Pinning](#pinning)
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

## <a id="pr-title"></a>Pull Request Title
When creating a pull request manually the title of your pull request must follow a predefined format. 

| Prefix | Package Name | Version |
|--------|--------------|---------|
| The word `operator` | Operator package name | Version in parenthesis. A `v` prefix is suggested|

### Examples
`operator simple-demo-operator (v0.0.0)`

`operator hello-world-certified (v.1.2.3)`

`operator my-operator (3.2.1)`

## <a id="auth-gh-users"></a>Authorized GitHub Usernames
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

## <a id="pkg-name"></a>Package Name
Your Operator's package name must be used consistently in three areas

| Package |
|--------------|
| Name of your folder under the `operators` directory in your fork |
| Value of the `operators.operatorframework.io.bundle.package.v1` annotation in `annotations.yaml` |
| Prefix of the filename for your `clusterserviceversion.yaml` file |

## <a id="pinning"></a>Digest Pinning
All images referenced in your Operator Bundle must reference SHA digest and not tags. The existance of tags in your bundle will cause a certification failure Replace all image tags with image digests. 

| Unpinned Example | Pinned Example |
|----------|--------|
| `quay.io/my_repo/my_image:v1.0.0` | `quay.io/my_repo/my_image@sha256:fd8d827d4d345ec327cb92d30086a17a2e08ba9c3163db4a25bfe2512123fd6a` |

# <a id="automated"></a>Automated Pull Request *(Tested on Partner Premise)*

## <a id="latest-code"></a>Make sure you are using the latest version of the Pipeline
As the Pipeline is updated with fixes and enhancements you want to make sure you are using the latest version. 

## <a id="fork-prod"></a>Make sure we are using Production
Partners who were apart of the alpha testing and beta testing may still be using GitHub forks of the `preprod` repos.  Make sure you are now using the production repo for your GitHub fork. 

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
If you are leveraging multiple container registries you will likely need to provide credentials for each. The sample script below can be modified to accommadate X number of registries

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

For example if my Operator is called `simple-demo-operator` then the follwing should be set to:
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

### Create a Registry Service Acount
https://access.redhat.com/terms-based-registry/

Upon creating a Registry Service Account you will be given a username similar to the one below

`username`: `123456789:my-sa-acctname`

And a token.

`eyJhbGc.................`

With the username and the token as your password you can follow the instructions for [supporting mupltiple registries](#multiple-registries). 


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
