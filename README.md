
#In OpenShift, create a cicd project:

oc new-project cicd

#deploy the jenkins template:

#It is recommended to use persistent storage to persist Jenkins pipeline configuration.
#For testing purpose, if persistent storage is not available on your OpenShift #environment, you can use the ephemeral template.

oc create -f jenkins-persistent-template.json

# Add the following to your OpenShift master configuration (it is a YAML file, spaces and indentation are important):

jenkinsPipelineConfig:
  autoProvisionEnabled: true
  parameters: null
  serviceName: jenkins
  templateName: jenkins-persistent
  templateNamespace: cicd

## Restart OpenShift master

# create the development environment for the project:

oc new-project development --display-name="Project ABC development environment"

# create the testing environment for the project:

oc new-project testing --display-name="Project ABC testing environment"


# login to OpenShift with admin credentials:

oc login admin

#When starting a pipeline build for the first time, the jenkins-persistent template will be provisionned. This will create a jenkins service account. We need to grant access to the jenkins service account to the dev and test project

oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n development
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n testing

#The testing project needs to be able to pull images from the development environment

oc policy add-role-to-group system:image-puller system:serviceaccounts:testing -n development

#the pipeline in this project builds and deploys an application called flower-app. We need to create the build and deployment config in the development environment that will be called by jenkins

#I have used the OpenShift console to create a simple app. In my case, I am using a php app (https://github.com/masauve/bgdemo).  Any app should work.

#In the console, go to the development project - add to project - php (or other), configure the php app (name, git repo). The pipeline is currently configured to work with the name flower-app. Feel free to adapt to your needs.

#IMPORTANT - go to the advanced configuration and disabled all builds and deployments triggers. We want jenkins to control that.


# We also need to create a deployment configuration in the testing project for flower-app

oc create deploymentconfig flower-app --image=<<RegistryServiceIP>>:5000/development/flower-app:promoteToQA -n testing

# IMPORTANT - You will need to edit the created dc to add the fully qualified image name.

oc expose dc myapp --port=8080

oc expose svc myapp
