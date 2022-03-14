# Container Certification

## Part One: Setting up your certification project in the UAT environment

### 1. Log in to the Partner Connect [UAT environment](connect.uat.redhat.com)
a. Use the credentials for your production [Partner Connect](connect.redhat.com) technology partner account.

### 2. Create your certification project
- Navigate to "Product certification" -> "Manage certification projects"
- Select "Create Project"
- Select "Linux Containers" as the platform 
- Select "Container Image" as what you want to certify

### 3. Setup your project

- <b>Project Name</b>: This is internal and will not be visible to users.

- <b>OS Content Type</b>: This will be Red Hat’s Universal Base Image or Red Hat Enterprise Linux. If you are using a different base image, this will not be able to be certified and you will need to migrate to a compatible base image type.

- <b>Distribution Method</b>: This will not have much impact for the beta phase since your certification will not be going into production. However, please select from the options based on where you intend to host your image.

### 4. Complete project checklist

<b>Note</b>: During beta testing in the UAT (user acceptance testing)  environment this part of the workflow will be inconsequential. However, please review and walk through the steps to ensure everything is clear and functional for you. 

The export control questionnaire is only required when distributing through Red Hat’s container registry. Additionally, you will not need to complete this step during beta.

### 5. Test your container image

*Prerequisites
- You must use a RHEL system
- You must use podman to log into the registry your image is hosted in and have the location of the authentication file that is automatically created ([see here](https://docs.podman.io/en/latest/markdown/podman-login.1.html#authfile-path))
- Connect project must be set up (does not have to be complete, but must be started)
- You will need a pyxis API key ([found here](https://connect.uat.redhat.com/account/api-keys))

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

