## Introduction

This is a demo of a very simple pipeline running in OpenShift. Many thanks to my colleague Martin Sauve for creating this example, I've only tweaked it a bit and automated the installation.

This pipeline uses the [bgdemo](https://github.com/gnunn1/bgdemo) project as the source application.

### Automated Installation

This demo includes an ansible playbook to automate the installation. Switch to the ```ansible``` folder and update vars.yml to reflect your OpenShift installation. Then simply run the ```pipeline.yml``` playbook:

```ansible-playbook pipeline.yml```

### Manual Installation

Follow the steps below to manually create the demo.

#### Create CICD Project and Jenkins

In OpenShift, create a cicd project:

```
oc new-project cicd
```

For testing purpose, if persistent storage is not available on your OpenShift ##environment, you can use the ephemeral template.

```
oc new-app jenkins-ephemeral
```

OR

```
oc new-app jenkins-persistent
```

#### Create Environments

Create the development environment for the project:

```
oc new-project dev --display-name="Flower Development"
```

Create the testing environment for the project:

```
oc new-project test --display-name="Flower Testing"
```

Login to OpenShift with admin credentials:

```
oc login -u admin
```

When we created the jenkins application, it will create a jenkins service account. We need to grant access to the jenkins service account to the dev and test project

```
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n dev
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n test
```

The test project needs to be able to pull images from the development environment

```
oc policy add-role-to-group system:image-puller system:serviceaccounts:test -n dev
```

#### Create Application

The pipeline in this project builds and deploys an application called ```myapp```. We need to create the build and deployment config in the development environment that will be called by jenkins

I have used the OpenShift console to create a simple app. In my case, I am using a php app (https://github.com/gnunn1/bgdemo).  Any app should work.

In the console, go to the development project - add to project - php (or other), configure the php app (name, git repo). The pipeline is currently configured to work with the name ```myapp```. Feel free to adapt to your needs.

IMPORTANT - go to the advanced configuration and disabled all builds and deployments triggers. We want jenkins to control that.


#### Create Test DC

We also need to create a deployment configuration in the test project for ```myapp```:

```
oc create deploymentconfig flower --image=<<RegistryServiceIP>>:5000/dev/flower:promoteToQA -n test
```

Note you can get the Registry Service IP and Port by running:

```
oc get svc -n default
```

And taking the IP and port for the docker-registry service.

IMPORTANT - If you don't update the ```<<RegistryServiceIP>>``` before creating the DC, you will need to edit the created dc to add the fully qualified image name.

#### Expose myapp in test

```
oc expose dc flower --port=8080
oc expose svc flower
```

#### Create the pipeline

```
oc create -f https://raw.githubusercontent.com/gnunn1/pipeline/master/pipeline.yaml -n cicd
```
