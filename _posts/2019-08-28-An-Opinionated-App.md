---
title: An Opinionated App
published: true
---

>
This post is a [copy of my linked-in article](https://www.linkedin
.com/pulse/opinionated-app-anand-sambamoorthy/?trackingId=jbl9trikOwRJeICbT001Qw%3D%3D) that I had published 
earlier before I had my GitHub hosted Jekyll platform.
>

The posting is about - how easy/fast can we develop a production class java based, git-managed (git-ops), continuously delivered, cloud-hosted-containerized service (micro)?

It turns out we can do it in 5 easy steps/commands - along with a few default choices to select/enter. This is as long as you are happy with chosen-frameworks opinions based on what you wanted rather than me-do-it-all-by-myself with a black-canvas!

## Tech Stack

* Spring boot - for application development (java based)
* GCP-Kubernetes (GKE) - for Cloud-Containers
* JX (Jenkins-x) - for Continuous Delivery. Note: This article is centered around this!
* GitHub (git) - for source control

Note: But for jx - you should be able to pick-chose equivalents that are more relevant e.g. AWS-EKS for cloud-containers, JHipster based application instead of spring and your own git provider instead of GitHub.

## Pre-Reqs

* GitHub Account
* GCP - <http://console.cloud.google.com> with containers GKE enabled 
(you can chose AWS, Azure as well, my free tier with AWS got exp!)

## The 5-listed commands (login to gcloud shell)

1. Create a [google-cloud](https://cloud.google.com/) project in the 
[console](https://console.cloud.google.com/)

```shell
# gcloud projects create <project-name>
% gcloud projects create k8s-my-apps-project
```

2. Install [jenkins-x](https://jenkins.io/projects/jenkins-x/) binary

```shell
% curl -L https://storage.googleapis.com/artifacts.jenkinsxio.appspot.com/binaries/cjxd/latest/jx-linux-amd64.tar.gz | tar xzv
% sudo mv jx /usr/local/bin ## or set your path to point to the extracted **/jx
```

3. Create [GKE](https://cloud.google.com/kubernetes-engine/) Cluster

```shell
# jx create cluster gke -n <cluster-name> -z <gcp-hosting-zone>

% jx create cluster gke -n k8s-my-apps-cluster -z us-west1-a --skip-login

# choose the GCP project e.g "k8s-my-apps-project" - select "Serverless Jenkins X Pipelines with Tekton". Use git and git-hub for pipeline (you might have to auth with api-token the first time)

# The above would have created a GKE Cluster with jenkins-x platform install along with 2 default environments "staging" and "production" with env-code checked to git-hub
```

4. Create a [Spring App (spring-boot)](https://spring.io/projects/spring-boot)

```shell
% jx create spring -d web -d actuator

# This will create a new spring-boot application along with a new repo in git-hub e.g. jx-helloworld-app, generate files Dockerfile, Charts (helm), Jenkinsfile (jenkins-x.yml) necessary for build-deploy

# Will also deploy the application in staging env - verify with "%jx get applications". Paste the URL displayed in a browser and you should get the spring-boot infamous 404; "WhiteLabel Error Page". You know this means it is working!
```

5. Make a change and move/promote-to-production

```shell
# Make a change - add a static web-page or a restful api. Use any editor add a static html page in the spring boot src/main/resources/static/jx.html - with some html content e.g. "Hello World!" 

# Commit the change to git-hub in master (for real - you need to use branch and pull-request rather than direct on master and jx does a pr-environement; that is for later)

# This will deploy the new app to staging again (automagically). 
# Observe the new deploy "%jx get activity -f jx-helloworld-app -w"
# Verify the URL and the version "%jx get applications"

# Move/Promote-To-Production 
# jx promote <application-name> --version <version> -- env <environment>

% jx promote jx-helloworld-app --version v0.0.2 --env production

# Verify the URL in prod. "%jx get applications"
# curl -s http://jx-helloworld-app.jx-production.xxx.xxx.xxx.xxx.nip.io/jx.html

```

In **summary**, if you have the pre-reqs and able to get this going you 
should have the application running in GKE in less than a couple of hours.

Any changes you want to make is done declaratively in source-control (GitHub) and the containerized platform is changed to reflect that! This is along with application provisioning as well say - increase replicas (pods) to 'n'; all controlled in source-control (yaml'ed)

All this is made possible with "Jenkins X" an open source project that offers automated CI/CD for cloud native applications on Kubernetes. In a gist it provides developers with a number of benefits - faster software delivery releases taking an opinionated view of the application dev & deploy, simplicity of installation and configuration and enhances the business sustainability over time. 


## Reference

* <https://jenkins.io/projects/jenkins-x/>
* <https://www.cloudbees.com/resource/whitepaper/cicd-cloud-native-applications-kubernetes>
     