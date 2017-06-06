
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
oc new-project dev --display-name="ABC Development"
```

Create the testing environment for the project:

```
oc new-project test --display-name="ABC Testing"
```

Login to OpenShift with admin credentials:

```
oc login admin
```

When we created the jenkins application, it will create a jenkins service account. We need to grant access to the jenkins service account to the dev and test project

```
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n dev
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n test
```

The test project needs to be able to pull images from the development environment

oc policy add-role-to-group system:image-puller system:serviceaccounts:test -n dev

#### Create Application

The pipeline in this project builds and deploys an application called ```myapp```. We need to create the build and deployment config in the development environment that will be called by jenkins

I have used the OpenShift console to create a simple app. In my case, I am using a php app (https://github.com/masauve/bgdemo).  Any app should work.

In the console, go to the development project - add to project - php (or other), configure the php app (name, git repo). The pipeline is currently configured to work with the name ```myapp```. Feel free to adapt to your needs.

IMPORTANT - go to the advanced configuration and disabled all builds and deployments triggers. We want jenkins to control that.


#### Create Test DC

We also need to create a deployment configuration in the test project for ```myapp```:

```
oc create deploymentconfig myapp --image=<<RegistryServiceIP>>:5000/dev/myapp:promoteToQA -n testing
```

Note you can get the Registry Service IP and Port by running:

```
oc get svc -n default
```

And taking the IP and port for the docker-registry service.

IMPORTANT - If you don't update the ```<<RegistryServiceIP>>``` before creating the DC, you will need to edit the created dc to add the fully qualified image name.

#### Expose myapp in test

```
oc expose dc myapp --port=8080
oc expose svc myapp
```

#### Create the pipeline

```
oc create pipeline.yaml -n cicd
```
