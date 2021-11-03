# Troubleshooting the Operator Cert Pipeline

## Table of Contents
* [Use the latest Pipeline code](#latest-code)
* [Fork the Production GitHub repo](#fork-prod)
* [Get a clean kubeconfig](#clean-kubeconfig)
* [Digest pinning fatal: could not read Username](#gh-username)
* [Support multiple container registries / Registry access errors](#multiple-registries)
* [Cannot find CSV](#cannot-find-csv)
* [Get a Red Hat Registry Service Account token](#fork-prod)

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