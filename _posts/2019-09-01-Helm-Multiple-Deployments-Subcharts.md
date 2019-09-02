---
title: "Kubernetes-Helm Deploying Multiple Applications(containers) by Subcharts"
published: true
tags: helm, kubernetes, packaging, multiple containers
---

This is my first real post in the newly setup [my-github pages]
(https://asamba.github.io). 

This post is stemming with my play-work with kubernetes where I wanted to install multiple apps (containers/services) with a single command. The rescue came in the way of [Helm](https://helm.sh/) - A package manager for Kubernetes. This will describe how to install/manage multiple apps (containers/services). You might want to reference [Using Helm and Kubernetes](https://www.baeldung.com/kubernetes-helm) for a single application deployment.
 
### Applications Topology

| Sno | App Type | App Name |
| -------- | ------- | ------- | 
| 1 | Parent | greeting-app |
| 2 | Child-1 | helloworld |
| 3 | Child-2 | hiworld |


### Step-1: Create the Services

#### Create Child Service - helloworld 
* A simple spring-boot service - you can add a RestController /hello that returns "Hello World" or you can create a static page /resources/static/hello.html that returns "Hello World!"
* Create a docker image and push that to the docker-hub

```console
% mvn package
% docker build -t asamba/helloworld:v1 
% docker push asamba/helloworld:v1
# replace asamba with your own dockerhub repo
```

#### Create Child Service - hiworld 
* A simple spring-boot service - you can add a RestController /hi that returns "Hi World" or you can create a static page /resources/static/hi.html that returns "Hi World!"
* Create a docker image and push that to the docker-hub

```console
% mvn package
% docker build -t asamba/hiworld:v1 
% docker push asamba/hiworld:v1
```

### Step-2: Create the Charts

#### Create Subchart for Child application - helloworld & hiworld
* Generate sample chart using ```helm create helloworld``` - this should create a folder helloworld with sub-folders/files inside. 
* Delete templates/tests, ingress.yaml and remove the ingress references in NOTES.txt
* Edit the values.yaml file to look like following

```yaml
replicaCount: 1

image:
  repository: asamba/helloworld
  tag: v1
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 8080

containerPort:
  port: 8080

```

* Verify the chart with `helm lint helloworld` and ensure there are no errors. You can also check the actual yamls by `helm template helloworld

* Generate sample chart for hiworld similar to the above and the values.yaml updated and test with `helm lint hiworld` 

#### Create Chart for Parent application - greeting
* Generate sample chart using `helm create greetings` 
* Delete all the files unders templates/ folder
* Create a new file `requirements.yaml` under greetings/ with the below content

```yaml
dependencies:
  - name: helloworld
  - name: hiworld

```

### Step-3: Deploy the app

`helm install --name greetings-app greetings`

* This should deploy the greetings-app with the other 2 child applications with the associated containers/services.

### You can package the application with `helm package` as described in the helm documentation or the tutorial at [Using Helm and Kubernetes](https://www.baeldung.com/kubernetes-helm)
 
### Step-2 (Alternative): Another way to create the dependent-child applications are as below
* Package your child applications and host in a http server; best with github and create the requirements.yaml as below (sample)

```yaml
dependencies:
  - name: helloworld-app
    repository: https://raw.githubusercontent.com/asamba/myhelmcharts/master/
    version: 0.1.0
    tags:
      - helloworld
  - name: hi-app
    repository: https://raw.githubusercontent.com/asamba/myhelmcharts/master/
    version: 0.1.0
    tags:
      - hiworld
```


Note: the above method is the preferred method for creating dependent applications say- you want to create a database for a service.

In summary - Helm is awesome and helps you to package applications and manage (install, delete, upgrade) with a single command. No more multiple files need to be run in seq like `kubectl apply -f hiworld-deployment.yaml` `kubectl apply -f hiworld-service.yaml` for each application you want to maintain along with common ones like `ingress.yaml` etc.

### Source Code Reference

* [Charts](https://github.com/asamba/greetings-charts) 
* [HelloWorld Source Code](https://github.com/asamba/helloworld)
* [HiWorld Source Code](https://github.com/asamba/hiworld)
 