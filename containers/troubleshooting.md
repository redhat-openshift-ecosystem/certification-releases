# Container Certification Troubleshooting

## Table of Contents

### Certification Tests

* [RunAsNonRoot](#runasnonroot)
* [BasedOnUBI](#basedonubi)
* [HasModifiedFiles](#hasmodifiedfiles)
* [HasLicense](#haslicense)
* [HasUniqueTag](#hasuniquetag)
* [LayerCountAcceptable](#layercountacceptable)
* [HasNoProhibitedPackages](#hasnoprohibitedpackages)
* [HasRequiredLabel](#hasrequiredlabel)
* [Vulnerability Scan After Submission](#vulnerabilityscanner)

### Common Issues
* [Authorization Errors](#authorizationerrors)
* [File Not Found Errors](#filenotfound)
* [Calling Remote API Errors](#callingremoteapierrors)
## <a id="runasnonroot"></a>RunAsNonRoot
The certification tooling is verifying that the USER is declared in the dockerfile and is not one of the following:
- USER root
- USER 0
- no user specified
  
If you are an infrastructure partner, and your container needs root privileges, you must navigate to the 'Settings' tab of your certification project. Scroll down, and select 'Privileged' under the 'Host level access' field. This will allow the certification tooling to omit the RunAsNonRoot test and is submect to Red Hat review. This must be selected before the preflight tool is used.
  
## <a id="basedonubi"></a>BasedOnUBI
This test verifies that the container is using either UBI or RHEL as the base image. This will fail if you are using another linux distribution or scratch. Open a support case if you have questions about this policy.

## <a id="hasmodifiedfiles"></a>HasModifiedFiles
This test checks that no files installed via RPM in the base Red Hat layer are modified. Files that are able to be modified by end users, like config files, are the only modified files allowed.
  
## <a id="haslicense"></a>HasLicense
Your dockerfile must include a licenses directory and must include one or more license files. If the directory is empty or non-existent, this test will fail. The license should be for the software product itself.
  
## <a id="hasuniquetag"></a>HasUniqueTag
The test is confirming that the registry has one or more tags other than 'latest'. This test will also fail if your chosen registry does not expose the '/tags/list' endpoint.
  
## <a id="layercountacceptable"></a>LayerCountAcceptable
 The test confirms you have less than 40 layers.
  
## <a id="hasnoprohibitedpackages"></a>HasNoProhibitedPackages
 The tool checks that you have no prohibited packages in your container, including RHEL kernel packages.
  
## <a id="hasrequiredlabel"></a>HasRequiredLabel
See [our policy guide](https://access.redhat.com/documentation/en-us/red_hat_openshift_certification/4.9/html-single/red_hat_openshift_software_certification_policy_guide/index#:~:text=Chapter%C2%A02.%C2%A0Requirements%20for%20container%20images) for the required labels. Double check each label is included in the dockerfile. 
  
## <a id="vulnerabilityscanner"></a>Vulnerability Scan After Submission
The vulnerability scanning mechanism, powered by the Clair vulnerability scanner, runs after you submit your test results to Partner Connect. You must receive a grade of A to be able to publish the image. If you receive a grade of B or lower:
- Make sure you are pulling the latest UBI version

## <a id="authorizationerrors"></a>Authorization Errors
  <i>"The logs are saying I am not authorized"</i>

Possible causes:
  - You are not logged into your image registry 
  - You are using an API key for the wrong account: make sure the API key matches the account that you are using for the certification project.
  - You are resubmitting an image with the same tag that is already published: this unauthorized error comes from pyxis, our internal infrastructure.

## <a id="filenotfound"></a>File Not Found Errors
Possible causes:
  - You are not specifying the full path of the docker config file: please make sure you are using the full path, not the relative path. The file should only include the credentials for the image being certified.

## <a id="callingremoteapierrors"></a>Calling Remote API Errors
Possible causes:
  - You are using the old opsid in --certification-project-id, you could find the project id in project link  https://connect.redhat.com/projects/1234567890/overview
