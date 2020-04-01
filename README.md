# OpenShift Developer Experience Demo

The purpose of this repository is to document a quick, yet thorough,
demonstration of the development capabilities of OpenShift.

* [Source-to-Image (S2I) from Git repository](#S2I-from-Git-Repository)
* [Source-to-Image (S2I) from Containerfile](#S2I-from-Containerfile)
* [Blue-Green Deployment](#Blue-Green-Deployment)
* [Jenkins Pipeline Deployment](#Jenkins-Pipeline-Deployment)
* [Wrap-up Presentation](#Wrap-Up-Presentation)

Before starting, make sure you have cloned a copy of this repository locally.
All the YAML deployment files referenced are in the repo.

```bash
git clone https://github.com/techjw/cicd-demo.git
cd cicd-demo
```


## S2I from Git Repository
Demonstrate build/deploy of the Duck Hunt application using S2I from source code.
The application requires nodejs 12, so create an ImageStream for ubi8/nodejs-12

```bash
oc create -f imagestream-nodejs12.yaml
oc new-project demo-s2i-git
oc new-app openshift/nodejs-12~https://github.com/techjw/DuckHunt-JS.git
```

* Show all resources via command-line (`oc get all -n demo-s2i-git`)
* Show all resources via console


## S2I from Git Containerfile
Demonstrate build/deploy of a simple application using S2I to build from a Containerfile.
Also, be sure to explain Containerfile == Dockerfile.

```bash
oc new-project demo-s2i-container
oc new-app https://github.com/techjw/s2i-containerfile.git
oc expose svc s2i-containerfile
oc get route s2i-containerfile
```

* Show all resources via command-line (`oc get all -n demo-s2i-git`)
* Show all resources via console

#### Update index.php and rebuild
```bash
oc start-build s2i-containerfile --follow=true
```

* Show the Git Webhook in the BuildConfig console page


## Blue-Green Deployment
Setup Blue-Green deployment for 2 versions of an application.

#### Deploy BLUE application
```bash
oc new-project bg-demo
oc new-app nginx-example~https://github.com/techjw/bluegreen-demo.git#blue --name=nginx-blue
oc expose svc nginx-blue
oc get route nginx-blue
```

#### Create BlueGreen route
```bash
oc expose svc nginx-blue --name=nginx-bluegreen
oc get route nginx-bluegreen
```

#### Deploy GREEN application, switch traffic manually
```bash
oc new-app nginx-example~https://github.com/techjw/bluegreen-demo.git#green --name=nginx-green
oc expose svc nginx-green
oc get route nginx-green

oc patch route/nginx-bluegreen -p '{"spec":{"to":{"name":"nginx-green"}}}'
# Check that application is GREEN in browser

oc patch route/nginx-bluegreen -p '{"spec":{"to":{"name":"nginx-blue"}}}'
# Check application is BLUE in browser
```

#### Load balance between BLUE and GREEN
```bash
oc annotate route/nginx-bluegreen haproxy.router.openshift.io/balance=roundrobin
oc annotate route/nginx-bluegreen haproxy.router.openshift.io/disable_cookies=true

oc set route-backends nginx-bluegreen nginx-blue=50 nginx-green=50
# Check application in the browser, refresh and watch it switch back-and-forth.
```


## Jenkins Pipeline Deployment

Stand up an ephemeral instance of Jenkins with the OpenShift template and
demonstrate use of an inline Jenkinsfile pipeline strategy to deploy an
application (nodejs + mongodb).


```bash
oc new-project demo-jenkins-pipeline
oc get template -n openshift | grep -i jenkins
oc new-app jenkins-ephemeral
oc create -f nodejs-jenkins-pipeline.yaml
```

* Talk about the available templates used for Jenkins deploy
    * A Jenkins server is required for JenkinsPipeline strategy
* Show the BuildConfig pipeline interface in the console
* Introduce Tekton / OpenShift Pipelines


## Wrap-Up Presentation

Start with a quick tour of the Developer view, then run through the
following presentation containing slides that introduce more advanced features.

[OpenShift Developer Experience](https://docs.google.com/presentation/d/1py6p1nA1Sz4FzsUZBS0IIIsuQKxfWP174A3z-w_Zr10/edit?usp=sharing)

* CodeReady Workspaces
* CodeReady Containers
* `odo` Developer CLI
* OpenShift Pipelines (Tekton)
* OpenShift Serverless (KNative)


## Recognition

Thanks and credit to those it is due.
The following sources were used in helping compile this demo.

* Juanjo Floristan - [ocp-build-demos](https://gitlab.com/jfloristan/ocp-build-demos)
* OpenShift/Origin - [nodejs-sample-pipeline.yaml](https://github.com/openshift/origin/blob/master/examples/jenkins/pipeline/nodejs-sample-pipeline.yaml)
* Vadim Rutkovsky - [DuckHunt-JS](https://github.com/vrutkovs/DuckHunt-JS)
    * Matt Surabian - [DuckHunt-JS](https://github.com/MattSurabian/DuckHunt-JS)
