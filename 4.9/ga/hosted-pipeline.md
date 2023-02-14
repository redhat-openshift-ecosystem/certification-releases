# Operator Certification Hosted Pipeline<br/>Instructions

## Get Help:
> Technology Partner Success Desk is a service  for all our technology partners where they can ask technical and non-technical questions pertaining to Red Hat offerings, programs, engagement processes, etc. If you run into any issues throughout these instructions please reach out to the Technology Partner Success Desk. 
>
> You can access the Success Desk by going to: [Red Hat Help Request](https://connect.redhat.com/support/technology-partner/#/). 


## Table of Contents
* [Before you start](#before-you-start)
* [Step 1: Fork the Upstream Repo](#fork)
* [Step 2: Add your Operator Bundle](#add-bundle)
* [Step 3: Create a Pull Request](#pull-request)
* [Reminders](#reminders)
 
 When certifying your Operator with the Red Hat Hosted Pipeline you will need to create a Pull Request against the Red Hat certification repository.  Instructions for doing so follow:

## <a id="before-you-start"></a>Before you start
* Complete the Project checklist in connect.redhat.com
* Add your GitHub username to the list of authorized GitHub users in connect.redhat.com under the Project settings page. 
* Add a docker config.json secret to connect.redhat.com under the Project settings page in the YAML field if you are using a private container registry

## <a id="fork"></a>Step 1: Fork the Upstream Repo
Based upon the Catalog(s) that you are targeting for distribution, fork the appropriate repo(s) from the table below:

| Catalog             | Upstream Repo |
|-------------------- |---------------------------------------------------------------------------|
| Certfied Catalog    | https://github.com/redhat-openshift-ecosystem/certified-operators |
| Red Hat Marketplace | https://github.com/redhat-openshift-ecosystem/redhat-marketplace-operators |

> If you intend to publish in multiple catalogs, you will need to fork each catalog and complete the certification once for each fork. 

If you are not familiar with creating a fork in GitHub [you can find instructions here](https://docs.github.com/en/get-started/quickstart/fork-a-repo). 

## <a id="add-bundle"></a>Step 2: Add your Operator Bundle
In the `operators` directory of your fork there are a series of subdirectories. 

### If you have certified this Operator before

Find the folder for your Operator under the `operators` directory. This is where you will place the contents of your Operator Bundle.

Please Note: Your package name must be consistent with the existing folder name you see for your Operator. For Marketplace bundles, you will need to manually add “-rhmp” to your package name. Previously, this was done automatically and therefore will not impact customer upgrades when manually changed. 

### If this is the first time this Operator has been certified

If this Operator does not already have a subdirectory under the `operators` parent directory then you will need to create one. Use the illustration above as an example of the expected directory structure. 

Create a new directory under `operators`.  The name of this directory should match your Operator's package name. In our example it is `my-operator` 

* Under the directory you created above, create a subdirectory representing the version number of the Operator. In our example it is `v1.4.6`

* Under the version directory, add a `manifests` folder containing all your OpenShift manifests including your `clusterserviceversion.yaml`

### All Operators

The example below illustrates the expected directory structure. 

```bash
├── config.yaml
├── operators
  └── my-operator
      ├── v1.4.6
      │   ├── manifests
      │   │   ├── cache.example.com_my-operators.yaml
      │   │   ├── my-operator-controller-manager-metrics-service_v1_service.yaml
      │   │   ├── my-operator-manager-config_v1_configmap.yaml
      │   │   ├── my-operator-metrics-reader_rbac.authorization.k8s.io_v1_clusterrole.yaml
      │   │   └── my-operator.clusterserviceversion.yaml
      │   └── metadata
      │       └── annotations.yaml
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


*ci.yaml*: This file should include your Red Hat Technology Partner project id and the organization target for this operator. 

You can find instructions on where to find your project id: [here](https://github.com/redhat-openshift-ecosystem/certification-releases/blob/main/4.9/ga/operator-cert-workflow.md#step-a---get-project-id). 
``` bash
cert_project_id: "<your partner project id>"
```

*annotations.yaml*: This file should include an OpenShift versions annotation. *(This should be added to any existing content)*
```bash
# OpenShift annotations.
com.redhat.openshift.versions: v4.9-v4.12
```

The `com.redhat.openshift.versions` field, part of the metadata in the operator bundle, is used to determine whether an operator is included in the certified catalog for a given OpenShift version. You must use it to indicate the version(s) of OpenShift supported by your operator.

Note that the letter 'v' must be used before the version, and spaces are not allowed.
The syntax is as follows:
* A single version indicates that the operator is supported on that version of OpenShift or later. The operator will be automatically added to the certified catalog for all subsequent OpenShift releases.
* A single version preceded by '=' indicates that the operator is supported ONLY on that version of OpenShift. For example, using `"=v4.10"` will add the operator to the certified catalog for OpenShift 4.10, but not for later (or earlier) OpenShift releases.
* A range can be used to indicate support only for OpenShift versions within that range. For example, using `"v4.9-v4.11"` will add the operator to the certified catalog for OpenShift 4.9, 4.10 and 4.11 but not for OpenShift 4.12.

For more details, please see [Managing OpenShift Versions](https://redhat-connect.gitbook.io/certified-operator-guide/ocp-deployment/operator-metadata/bundle-directory/managing-openshift-versions) in the Certified Operator Build Guide.

> Note: An example Operator Bundle can be found [here](https://github.com/opdev/simple-demo-pipeline/tree/main/operators/simple-demo-operator).

## <a id="pull-request"></a>Step 3: Create a Pull Request
The final step is to create a Pull Request against the targeted upstream repo. 

| Catalog             | Upstream Repo |
|-------------------- |---------------------------------------------------------------------------|
| Certfied Catalog    | https://github.com/redhat-openshift-ecosystem/certified-operators |
| Red Hat Marketplace | https://github.com/redhat-openshift-ecosystem/redhat-marketplace-operators |

> If you intend to publish in multiple catalogs, you will create a pull request for each target catalog. 

If you are not familiar with creating a pull request in GitHub you can [find instructions here](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork). 

#### Note
The title of your pull request must conform to the following format.  

```
operator my-operator (v1.4.6)
```

It should begin with the word `operator` followed by your Operator Package name, followed by the version number in parenthesis. 

Once your pull request is created it will trigger the Red Hat hosted pipeline and provide an update via a PR comment once it has failed or completed. 


## <a id="reminders"></a>Reminders
* You can re-trigger the Red Hat hosted pipeline by closing your PR and then reopening it. 
* You can only have one open PR at a time for a given Operator version
* Once a PR has been successfully merged it can not be changed.  You will need to bump the version of your Operator and open a new PR. 
* The Package name of your Operator, should be used as the directory name you created under `operators`.  This package name should match the `package` annotation in the `annotations.yaml` file. This package name should also match the prefix of the clusterserviceversion.yaml filename.  
* Your pull requests should only modify files in a single Operator version directory.  Do not attempt to combine updates to multiple versions nor updates across multiple Operators. 
* The version indicator used to name your version directory should match the version indicator used in the title of the pull request. 
* Image tags are not accepted only SHA digest.  [Replace all references to image tags](troubleshooting.md#pinning) with the corresponding SHA digest. 



