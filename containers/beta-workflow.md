# Container Certification

## *This currently is only applicable to containers for OpenShift Container Platform and Red Hat Enterprise Linux*

## Part One: Setting up your certification project in the UAT environment

### 1. Log in to the Partner Connect [UAT environment](https://connect.uat.redhat.com)
a. Use the credentials for your production [Partner Connect](https://connect.redhat.com) technology partner account.

### 2. Create your certification project
- Navigate to "Product certification" -> "Manage certification projects"
- Select "Create Project"
- Select "Red Hat OpenShift" or "Red Hat Enterprise Linux" as the desired platform 
- Select "Container Image" as what you want to certify

### 3. Setup your project

- <b>Project Name</b>: This is internal and will not be visible to users.

- <b>OS Content Type</b>: This will be Red Hat’s Universal Base Image or Red Hat Enterprise Linux. If you are using a different base image, this will not be able to be certified and you will need to migrate to a compatible base image type.

- <b>Distribution Method</b>: This will not have much impact for the beta phase since your certification will not be going into production. However, please select from the options based on where you intend to host your image.

<b>Note</b>: We recommend for beta testing, if possible, to use a "Non-Red Hat container registry" due to UAT limitations and to not use the "Red Hat Marketplace only" selection. If you need the "Red Hat container registry and marketplace if applicable" option to utilize our internal registry, you must use the special instructions below.

### 4. Complete project checklist

<b>Note</b>: During beta testing in the UAT (user acceptance testing) environment this part of the workflow will be inconsequential. However, please review and walk through the steps to ensure everything is clear and functional for you. 

*The export control questionnaire is only required when distributing through Red Hat’s container registry. Additionally, you will not need to complete this step during beta.

### 5. Submit your container image for verification

*Prerequisites
- You must use a RHEL system
- You must use podman to log into the registry your image is hosted in and have the location of the authentication file that is automatically created ([see here](https://docs.podman.io/en/latest/markdown/podman-login.1.html#authfile-path))
- Connect project must be set up (does not have to be complete, but must be started)
- You will need a pyxis API key ([found here](https://connect.uat.redhat.com/account/api-keys))

<b>Non-Red Hat container registry</b>

a. Build your container

[Building an image with Podman](https://docs.podman.io/en/latest/markdown/podman-build.1.html) 

<b>Note</b>: You are not required to build images with podman; this is just a reference.

b. Upload your container to a registry (any registry of your choice)

- Your registry may be private or public

c. Download the [Preflight certification utility](https://github.com/redhat-openshift-ecosystem/openshift-preflight/releases)

d. Run Preflight (without submitting results)

<b>Running container policy checks against a container iteratively until all tests pass:</b>
```
set $ PFLT_PYXIS_HOST=catalog.uat.redhat.com/api/containers

preflight check container registry.example.org/your-namespace/your-image:sometag \
--pyxis-api-token=<your-api-token> \
--certification-project-id=<cert-project-id> 
```  

<b>Red Hat Container registry and marketplace if applicable [only use if necessary]</b>

Please do not follow the instructions in the UI of the UAT environment for this distribution method. Use the following instructions. Beta testing has a dependency on the production version of the Partner Connect portal. Take care, as utilizing this method has the potential to impact your production container projects.

<b>Prerequisite</b>: Create and set up a container certification project at [connect.redhat.com](https://connect.redhat.com) in your Partner Connect portal technology partner account set to distribute to the Red Hat container registry. You will use some information from this project to complete your UAT testing.

a. Build your container

[Building an image with Podman](https://docs.podman.io/en/latest/markdown/podman-build.1.html) 

<b>Note</b>: You are not required to build images with podman; this is just a reference.

b. Upload your container to the internal Red Hat registry
- If possible, please use a container that <b>already exists</b> in the production Red Hat registry. Otherwise, please be aware that your container will be going through an additional certification scan prior to utilizing the preflight tool.

In order to test the certification of your container, you must first push it to Red Hat's inbound certification registry (the production version, not UAT). Once your image has been pushed to the inbound registry, it will automatically be scanned and the result of the certification test will be available for you to view. This automated scanning only remains in place during UAT for internally hosted images and will be subsequently removed in favor of the new preflight tool.

The following commands will allow you to login to the registry, tag your image and push your image:

<b>Container Registry Login:</b>
The inbound certification registry requires authorization. The following command will allow you to authenticate to the registry. You will be prompted for a password. Use the registry key that is provided to you in any of your production container project set to distribute to the Red Hat container registry. 

*You will find this registry key in the "Upload image manually" page of your Red Hat hosted container project.

```
podman login -u unused scan.connect.redhat.com
```

<b>Tag your container</b>:
In order to push your container, you must first tag it so it is associated to the certification registry. The podman tag command will do this. You must replace the following parameters:
- [image-id]: The container IMAGE ID for the image you want to submit. This IMAGE ID can be displayed using the podman images command.
- [image-name]: Assign any name to your container. This name wil not be used for publishing.
- [tag]: A version identifier for this image. If this image is published, this tag will be published and used to uniquely identify this image. The tag cannot be empty or the string "latest".

```
podman tag [image-id] scan.connect.redhat.com/ospid-[production-project-ID]/[image-name]:[tag]
```
<b>Note</b>: Do not use the UAT project ID - use your production project ID from [connect.redhat.com](https://connect.redhat.com)

<b>Push your container</b>
This command will send your container to the certification registry. Replace the [image-name] and [tag] parameters and use the same replacement values as used in the docker tag command.

```
podman push scan.connect.redhat.com/ospid-[production-project-ID]/[image-name]:[tag]
```
c. Download the Preflight certification utility https://github.com/redhat-openshift-ecosystem/openshift-preflight/releases

d. Run Preflight (without submitting results)

Running container policy checks against a container iteratively until all tests pass:
```
set $ PFLT_PYXIS_HOST=catalog.uat.redhat.com/api/containers
 
preflight check container registry.connect.redhat.com/your-namespace/your-image:sometag \
--pyxis-api-token=<your-api-token> \
--certification-project-id=<cert-project-id> 
```

### 6. Review log information and make adjustments as needed
  
  - Review troubleshooting information [here](https://github.com/redhat-openshift-ecosystem/certification-releases/blob/main/containers/troubleshooting.md) (this is an in-progress troubleshooting document)
  
### 7. Please submit a support ticket if needed [here](https://connect.redhat.com/support/technology-partner/#/case/new). Please put "Container Certification - Beta" as the subject.
  
  <b>Note</b> If you experience a bug related to preflight or the Partner Connect UAT environment, or if you have a suggestion for a feature improvement or contribution, please open a GitHub issue [here](https://github.com/redhat-openshift-ecosystem/openshift-preflight/issues). Be sure to review open issues to avoid duplication.
  
### 8. Submit Results
  
  <b>Running container policy checks against a container that passes all tests needed to submit to Red Hat:</b>
  ```
  preflight check container registry.example.org/your-namespace/your-image:sometag \
  --submit \
  --pyxis-api-token=<your-api-token> \
  --certification-project-id=<your-project-id> \
  --docker-config=/path/to/your/dockerconfig
  ```

Once you submit your test results, you will be able to view them in the certification project UI within Connect at this URL: 
```
https://connect.uat.redhat.com/projects/{ProjectID}/images/{ImageID}/scan-results
```
<b>Note</b>: You must go directly to this link to view your test results. The “images” tab in the UAT environment UI will not display during our beta phase. Therefore, you will not be able to navigate to the image test results without using the direct URL.

### 9. We need your input to improve our tooling and partner experience! Please share your feedback [here](https://forms.gle/Wyo9eEe1EDUqdCAy7)

