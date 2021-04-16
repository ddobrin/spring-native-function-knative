This sample app provides a simple `Hello` web app based on Spring Boot and Spring Cloud Functions.
It provides multiple deployment options and common Knative use-cases which developers are looking for.

`Currently Tracked Versions:`
* Spring Boot 2.4.5 - April 15, 2021
* Spring Native 0.9.2 (Spring Native Beta) - April 16, 2021
* OpenJDK version "11.0.10"
* OpenJDK 64-Bit Server VM GraalVM CE 21.0.0.2 (build 11.0.10+8-jvmci-21.0-b06, mixed mode, sharing)

This repo addresses the following topics:
1. Build 
  * Build a JVM / Native app image with the Spring Boot plugin and GraalVM
  * Build a JVM / Native Docker image with Java and Java Native Paketo Buildpacks
* CI/CD integration - Build a JVM / Native Docker image with kpack / Tanzu Build Service
2. Deploy 
  * Run locally / Kubernetes / Knative / Tanzu Serveless
3. Install Tanzu Serverless 
4. Serverless use-cases:
  * [x] Deployment of containers with the KNative(kn) CLI 
  * [x] Scale-to-zero, automatically
  * [x] Allow versioning of deployments and snapshots (deployed codes and configurations)
  * [x] Executing a particular version of a function
  * [x] Blue-Green and Canary deployments
    * It can be done in K8s
    * How to do B/G with the KNative(kn) CLI
      * Dynamic traffic splitting
      * Use Octant UI Plugin
  * [x] Dynamic resource configurations (memory, CPU cycles, concurrency, etc)
  * [x] Load-testing functions
  * [x] Delete a deployed service

Build Options:
* JVM application, leveraging OpenJDK
* Native Application, leveraging GraalVM

Deployment Models:
* Standalone web app
* Kubernetes Deployment and Service
* Knative Service

Source code tree:
```
src
├── main
│   ├── java
│   │   └── com
│   │       └── example
│   │           └── hello
│   │               └── SpringNativeFunctionKnativeApplication.java
│   └── resources
│       ├── application.properties
│       ├── static
│       └── templates
└── test
    └── java
        └── com
            └── example
                └── hello
                    └── SpringNativeFunctionKnativeApplicationTests.java

# The function used in this app is available in SpringNativeFunctionKnativeApplication.java
```

# Build

Building the code with the Spring Boot Maven wrapper leverages the following Maven profiles:
* `native-image` - build a Spring Native image leveraging GraalVM
* `jvm-image` - build a Spring JVM-based image leveraging OpenJDK

Building an executable application with the GraalVM compiler leverages the following Maven profile and requires the installation of the GraalVM and the native-image builder utility:
* `native`

## Build code as a JVM app using the Spring Boot Maven plugin with embedded Netty HTTP server
```bash 
# build and run code using
$ ./mvnw clean package spring-boot:run

# test locally
$ curl -w'\n' -H 'Content-Type: text/plain' localhost:8080 -d "from a Function"
```
## Build code as a Native JVM app using the GraalVM compiler with embedded Netty HTTP server
```bash 
# switch to the GraalVM JDK for this build
# ex, when using SDKman
$ sdk use java 21.0.0.r11-grl

# build and run code using
$ ./mvnw clean package -Pnative

# start the native executable
$ ./target/hello-function

# test locally
$ curl -w'\n' -H 'Content-Type: text/plain' localhost:8080 -d "from a Function"
```

## Build code as a JVM image using the Spring Boot Maven plugin and Java Paketo Buildpacks
```bash 
# build image with default configuration
$ ./mvnw clean spring-boot:build-image

# build image with the CNB Paketo buildpack of your choice
$ ./mvnw clean spring-boot:build-image -Pjvm-image

# start Docker image
$ docker run -p 8080:8080 hello-function-jvm:0.0.1

# test Docker image locally
$ curl -w'\n' -H 'Content-Type: text/plain' localhost:8080 -d "from a Function"
```

## Build code as a Spring Native image using the Spring Boot Maven plugin and the Java Native Paketo Buildpacks
```bash 
# build image with the CNB Paketo buildpack of your choice
$ ./mvnw clean spring-boot:build-image -Pnative-image

# start Docker image
$ docker run -p 8080:8080 hello-function-native:0.0.1

# test Docker image locally
$ curl -w'\n' -H 'Content-Type: text/plain' localhost:8080 -d "from a Function"
```
# CI/CD integration - Build a JVM / Native Docker image with kpack / Tanzu Build Service

To build an image with Java or Java Native Paketo Buildpacks with kpack or the Tanzu Build Service, you can use the commands listed below.

To start, install the tools as follows:
* `kpack CLI` - https://github.com/vmware-tanzu/kpack-cli 
  * kpack commands - https://github.com/vmware-tanzu/kpack-cli/blob/master/docs/kp.md 
* `kpack` - https://github.com/pivotal/kpack 
* Tanzu Build Service - https://network.pivotal.io/products/build-service/ 
  * Build Service Documentation - https://docs.pivotal.io/build-service/1-1/ 

## Building JVM Docker images
To build the JVM image with the Java Paketo Buildpack, please run:
```shell
$ kp image save hello-function-jvm \ 
    --tag <your-repo-prefix>/hello-function:jvm \ 
    --git https://github.com/ddobrin/spring-native-function-knative.git \
    --git-revision main \
    --cluster-builder base \ 
    --env BP_JVM_VERSION=11 \
    --env BP_MAVEN_BUILD_ARGUMENTS="-Dmaven.test.skip=true package spring-boot:repackage" \
    --wait 

* your-repo-prefix - prefix for your Container Registry. Ex. Docker-desktop hello-function:jvm, GCR gcr.io/pa-ddobrin/hello-function:jvm
* tag - image tag
* git - repo location 
* local-path - to build from a local download of the repo, replace "git" with "local-path"
        --local-path ~/spring-native-function-knative
* git-revision - the code branch in Git
* cluster-builder - the Paketo builder used to build the image
* BP_JVM_VERSION - Java version to build for, accepts 8, 11
* wait - if you wish to observe the build taking place
* BP_MAVEN_BUILD_ARGUMENTS - kpack/TBS works declaratively in K8s, therefore requires instructions for the `repackaging` goal to be triggered; local machine is imperative and `package` in pom.xml is sufficient. 
```

## Building Java Native Docker images
To build the JVM image with the Java Native Paketo Buildpack, please run:
```shell
$ kp image save hello-function-native \ 
    --tag <your-repo-prefix>/hello-function:native \ 
    --git https://github.com/ddobrin/spring-native-function-knative.git \
    --git-revision main \
    --cluster-builder tiny \ 
    --env BP_BOOT_NATIVE_IMAGE=1 \
    --env BP_JVM_VERSION=11 \
    --env BP_MAVEN_BUILD_ARGUMENTS="-Dmaven.test.skip=true package spring-boot:repackage" \
    --env BP_BOOT_NATIVE_IMAGE_BUILD_ARGUMENTS="-Dspring.spel.ignore=true -Dspring.xml.ignore=true -Dspring.native.remove-yaml-support=true --enable-all-security-services" \
    --wait 

* your-repo-prefix - prefix for your Container Registry. Ex. Docker-desktop hello-function:native, GCR gcr.io/pa-ddobrin/hello-function:native 
* tag - image tag
* git - repo location 
* local-path - to build from a local download of the repo, replace "git" with "local-path"
        --local-path ~/spring-native-function-knative
* git-revision - the code branch in Git
* cluster-builder - the Paketo builder used to build the image
* BP_BOOT_NATIVE_IMAGE - set to true builds a Spring Native image
* BP_JVM_VERSION - Java version to build for, accepts 8, 11
* wait - if you wish to observe the build taking place
* BP_MAVEN_BUILD_ARGUMENTS - kpack/TBS works declaratively in K8s, therefore requires instructions for the `repackaging` goal to be triggered; local machine is imperative and `package` in pom.xml is sufficient. 
* BP_BOOT_NATIVE_IMAGE_BUILD_ARGUMENTS - optimization arguments for the Native image to minimize image size
```

# Deploy

## Kubernetes Deployment and Service

<img src="https://kubernetes.io/images/kubernetes-horizontal-color.png"
     alt="Kubernetes" width="400" />

You can containerize this template app and deploy it as a Deployment and Service on Kubernetes.

For general guidance, see the [Spring Boot Kubernetes](https://spring.io/guides/gs/spring-boot-kubernetes/) Guide for details.

## Knative Service

<img src="https://avatars3.githubusercontent.com/u/35583233?s=280&v=4"
     alt="Knative" width="100" />

You can containerize this template app and deploy it as a Knative Service.

For general guidance, see the [Hello World - Spring Boot Java](https://knative.dev/docs/serving/samples/hello-world/helloworld-java-spring/) sample for details.

# Install Tanzu Serverless

Info URL: https://tanzu.vmware.com/serverless
Download: 
* log in to the Tanzu Network - https://network.pivotal.io/ 
* download Tanzu Serverless 
* follow the README.md#install for installation in your preferred K8s cluster

Pre-requisites:
* Kubernetes
    * Requires Kubernetes 1.17+
* Required command line tools
    * kubectl (Version 1.18 or newer)
    * kapp (Version 0.34.0 or newer)
    * ytt (Version 0.30.0 or newer)
    * yq (Version 3.4.1 )
    * kn
* Octant
    * install Octant - optional (https://github.com/vmware-tanzu/octant)
    * install Octant Plugin for Knative (https://github.com/vmware-tanzu/octant-plugin-for-knative)
* DNS
    * install Magic DNS - optional, easy to set up (https://knative.dev/v0.18-docs/install/any-kubernetes-cluster/)


### Summarized installation - subject to change
```shell
# Ensure that your Kubernetes client targets the intended cluster:
$ kubectl cluster-info

# Create a Kubernetes registry credentials secret in order to pull images:
# Your Tanzu Network login credentials
$ TANZU_LOGIN=user@example.com
$ TANZU_PASSWORD=password

# create secret
$ export SECRET_NAME=registry-credentials
$ export SECRET_NS=hello-function
$ kubectl create secret docker-registry ${SECRET_NAME} -n ${SECRET_NS} \
    --docker-server=registry.pivotal.io \
    --docker-username=${TANZU_LOGIN} --docker-password=${TANZU_PASSWORD}

# From the `serverless` directory, run the installation script:
$ ./bin/install-serverless.sh
```

# Serverless use-cases

## Deployment of containers

To start deploying without having to build the images, they are already available in DockerHub:
```shell
# JVM image
$ docker pull triathlonguy/hello-function:jvm

# JVM image for B/G
$ docker pull triathlonguy/hello-function:blue
$ docker pull triathlonguy/hello-function:green

# Native JVM image
$ docker pull triathlonguy/hello-function:native
```

```shell
$ kubectl cluster-info

$ export WORKLOAD_NS=hello-function
$ kubectl create namespace ${WORKLOAD_NS}

# copy the registry credentials secret in namespace
# use yq v3!
$ kubectl get secret ${SECRET_NAME} -n ${SECRET_NS} -oyaml |  \
     yq d - 'metadata.creationTimestamp' | yq d - 'metadata.namespace' |  \
     yq d - 'metadata.resourceVersion' |  yq d - 'metadata.selfLink' | yq d - 'metadata.uid' |   kubectl apply -n $WORKLOAD_NS -f -

# Add the secret to the default service account in the namespace
$ kubectl patch serviceaccount -n ${WORKLOAD_NS} default -p '{"imagePullSecrets": [{"name": "'${SECRET_NAME}'"}]}'

# create service
$ kn service create hello-function -n hello-function --image triathlonguy/hello-function:jvm --env TARGET="from Serverless Test - Spring Function on JVM" --revision-name hello-function-v1

        Creating service 'hello-function' in namespace 'hello-function':
        0.178s The Route is still working to reflect the latest desired specification.
        0.195s Configuration "hello-function" is waiting for a Revision to become ready.
        20.967s ...
        21.077s Ingress has not yet been reconciled.
        21.094s Waiting for Envoys to receive Endpoints data.
        21.477s Waiting for load balancer to be ready
        21.706s Ready to serve.

Service 'hello-function' created to latest revision 'hello-function-v1'; it is available at URL:
http://hello-function.hello-function.35.184.97.2.xip.io

# get the external address for your ingress
kubectl get service envoy -n contour-external \
  --output 'jsonpath={.status.loadBalancer.ingress[0].ip}'
ex.: 35.184.97.2

# test the service
$ curl -w'\n' -H 'Content-Type: text/plain' http://hello-function.hello-function.35.184.97.2.xip.io -d "test"

# load test the service with Siege
# install on Mac with `brew install siege` 
$ siege -d1  -c50 -t10S  --content-type="text/plain" 'http://hello-function.hello-function.35.184.97.2.xip.io POST test'
```

## Automatic scale-to-zero
```shell
# load test the service with Siege
# install on Mac with `brew install siege` 
$ siege -d1  -c200 -t60S  --content-type="text/plain" 'http://hello-function.hello-function.35.184.97.2.xip.io POST from-my-function'

# observe the function instances scaling up and, after 60s of inactivity, terminate and scale all the way back to zero.
```

### Create a service revision
When creating a revision, if the `traffic` parameter is not specified, all traffic will be routed to the new revision, which automatically becomes the `@latest`.
Check the traffic allocation from the initial service creation with `revision-name=hello-function-v1`.

Note that the deployment is done automatically by KNative with zero-downtime, using a `blue-green deployment` pattern!
```shell
# revision hello-function-1 gets 100% of the traffic
$ kn service describe hello-function -n hello-function
        Name:       hello-function
        Namespace:  hello-function
        Age:        3m
        URL:        http://hello-function.hello-function.35.184.97.2.xip.io

        Revisions:  
          100%  @latest (hello-function-v1) [1] (3m)
                Image:  triathlonguy/hello-function:jvm (pinned to ef7bef)

        Conditions:  
          OK TYPE                   AGE REASON
          ++ Ready                   3m 
          ++ ConfigurationsReady     3m 
          ++ RoutesReady             3m 
```

To create a new revision, however maintain the traffic on the previous revision, set the `traffic` parameter and prevent the new deployment to use the new revision.
This allow a `blue-green` deployment type of testing, as shown in future paragraphs.
```shell
# create revision hello-function-v2
$ kn service update hello-function -n hello-function --image triathlonguy/hello-function:jvm --env TARGET="from Serverless Test - from revision 2 of Spring Function on JVM" --revision-name hello-function-v2 --traffic @latest=0,hello-function-v1=100

Updating Service 'hello-function' in namespace 'hello-function':

  0.065s The Route is still working to reflect the latest desired specification.
  0.126s Revision "hello-function-v2" is not yet ready.
  5.596s ...
  5.679s Ingress has not yet been reconciled.
  5.758s ...
  5.911s unsuccessfully observed a new generation
  6.002s Waiting for Envoys to receive Endpoints data.
  7.735s Waiting for load balancer to be ready
  7.891s Ready to serve.

Service 'hello-function' updated to latest revision 'hello-function-v2' is available at URL:
http://hello-function.hello-function.35.184.97.2.xip.io
```

KN CLI allows us to `describe` the service and indicate that traffic still goes to the previous revision, while the new one gets zero traffic
```shell
$ kn service describe hello-function -n hello-function
Name:       hello-function
Namespace:  hello-function
Age:        12m
URL:        http://hello-function.hello-function.35.184.97.2.xip.io

Revisions:  
     +  hello-function-v2 (current @latest) [2] (1m)
        Image:  triathlonguy/hello-function:jvm (pinned to ef7bef)
  100%  hello-function-v1 [1] (12m)
        Image:  triathlonguy/hello-function:jvm (pinned to ef7bef)

Conditions:  
  OK TYPE                   AGE REASON
  ++ Ready                   1m 
  ++ ConfigurationsReady     1m 
  ++ RoutesReady             1m 
```

### Executing a specific revision of a function

To access a specific revision of a service, routing can be set up by assigning a tag to the revision (see https://knative.dev/docs/serving/using-subroutes/).

A tag applied to a route leads to an address for the specific traffic target to be created.
You can access that specific revision by prefixing `tag-` to the route.

For example, let's update revision `hello-function-v2` and assign the tag `candidate` to the revision. 
This allows us to test this candidate revision before sending any traffic to it.
```shell
$ kn service update hello-function -n hello-function  --tag hello-function-v2=candidate

$ kubectl get svc  -n hello-function 
NAME                                              TYPE           CLUSTER-IP    EXTERNAL-IP                                PORT(S)                             AGE
candidate-hello-function                          ExternalName   <none>        envoy.contour-internal.svc.cluster.local   80/TCP                              85s
...

$ curl -w'\n' -H 'Content-Type: text/plain' http://candidate-hello-function.hello-function.35.184.97.2.xip.io -d "test"

Hello from Serverless Test - from revision 2 of Spring Function on JVM
```

## Blue-Green and Canary deployments

### It can be done in K8s
Blue-green deployment can be done in K8s, as shown in the following Tanzu Developer Center Blog Post: [Declarative Deployments in Kubernetes: What Options Do I Have?](https://github.com/ddobrin/declarative-deployments-k8s#5)

### How to do B/G with the KNative(kn) CLI
When creating a revision, if the `traffic` parameter is not specified, all traffic will be routed to the new revision, which automatically becomes the `@latest`.
Check the traffic allocation from the initial service creation with `revision-name=hello-function-v1`.

To avoid the automatic traffic re-routing, a new revision should be created `without traffic routed to it` and `while specifying tag` (see above for both), in order to allow testing of the `green` version, and changing `blue` to `green` at a later time, after testing.

Let's reset the test by deploying a new image, as revision 3 and label it as the `blue` version, stable, and with traffic routed to it.
`green` will follow :
```shell
$ kn service update hello-function -n hello-function --image triathlonguy/hello-function:blue --env TARGET="from Serverless Test - from revision BLUE of Spring Function on JVM" --revision-name hello-function-blue --traffic @latest=100 --tag hello-function-blue=stable

## testing the BLUE revision
$ curl -w'\n' -H 'Content-Type: text/plain' http://stable-hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision BLUE of Spring Function on JVM

## deploy the GREEN revision, with tag green-candidate, and image tag green
$ kn service update hello-function -n hello-function --image triathlonguy/hello-function:green --env TARGET="from Serverless Test - from revision GREEN of Spring Function on JVM" --revision-name hello-function-green --traffic @latest=0,hello-function-blue=100 --tag hello-function-green=green-candidate

## testing the GREEN revision
$ curl -w'\n' -H 'Content-Type: text/plain' http://green-candidate-hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision GREEN of Spring Function on JVM

## Switch traffic from BLUE to GREEN after testing of GREEN has been completed

# first, we untag the stable tag from the BLUE revision 
# function route still points to BLUE
$ kn service update hello-function -n hello-function --untag stable

$ curl -w'\n' -H 'Content-Type: text/plain' http://hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision BLUE of Spring Function on JVM

## switch traffic to GREEN and assign green as STABLE
kn service update hello-function -n hello-function --tag hello-function-green=stable --traffic hello-function-green=100

## traffic points to the new route
$ curl -w'\n' -H 'Content-Type: text/plain' http://hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision GREEN of Spring Function on JVM

## testing by revision, the STABLE tag points to the GREEN revision
$ curl -w'\n' -H 'Content-Type: text/plain' http://stable-hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision GREEN of Spring Function on JVM
```

### Canary deployment 
For canary deployments, when deploying a new revision, traffic can be set to a small percentage, tested, then the canary can become the new version
```shell
# deploy the canary version
$ kn service update hello-function -n hello-function --image triathlonguy/hello-function:jvm --env TARGET="from Serverless Test - from revision CANARY of Spring Function on JVM" --revision-name hello-function-canary --traffic @latest=10,hello-function-green=90 --tag hello-function-canary=canary

# testing will respect the traffic percentage set above
$ curl -w'\n' -H 'Content-Type: text/plain' http://hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision GREEN of Spring Function on JVM
$ curl -w'\n' -H 'Content-Type: text/plain' http://hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision CANARY of Spring Function on JVM
$ curl -w'\n' -H 'Content-Type: text/plain' http://hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision GREEN of Spring Function on JVM
...

# traffic can be routed 100% to the canary revision when testing is complete
kn service update hello-function -n hello-function --traffic @latest=100

# traffic is routed only to the latest revision, which was canary
$ curl -w'\n' -H 'Content-Type: text/plain' http://hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision CANARY of Spring Function on JVM
```

Revisions can be listed as follows:
```shell
# KNative
$ kn service describe hello-function -n hello-function
Name:       hello-function
Namespace:  hello-function
Age:        2h
URL:        http://hello-function.hello-function.35.184.97.2.xip.io

Revisions:  
     +  hello-function-canary (current @latest) #canary [5] (26m)
        Image:  triathlonguy/hello-function:jvm (pinned to ef7bef)
  100%  @latest (hello-function-canary) [5] (26m)
        Image:  triathlonguy/hello-function:jvm (pinned to ef7bef)
     +  hello-function-green #green-candidate [4] (45m)
        Image:  triathlonguy/hello-function:green (pinned to ef7bef)
     +  hello-function-green #stable [4] (45m)
        Image:  triathlonguy/hello-function:green (pinned to ef7bef)
     +  hello-function-v2 #candidate [2] (1h)
        Image:  triathlonguy/hello-function:jvm (pinned to ef7bef)

Conditions:  
  OK TYPE                   AGE REASON
  ++ Ready                   4m 
  ++ ConfigurationsReady    26m 
  ++ RoutesReady             4m 

# Kubernetes
$ kubectl get deploy -n hello-function
NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
hello-function-blue-deployment                       0/0     0            0           52m
hello-function-canary-deployment                     0/0     0            0           25m
hello-function-green-deployment                      0/0     0            0           45m
hello-function-native-hello-function-v1-deployment   0/0     0            0           4h10m
hello-function-v1-deployment                         0/0     0            0           126m
hello-function-v2-deployment                         0/0     0            0           115m
```

# Setting requests and limits dynamically
```shell
# create the service with requests and limits
$ kn service create hello-limits -n hello-function --image triathlonguy/hello-function:jvm --env TARGET="from Serverless Test - with limits" --revision-name hello-limits-v1 --request memory=200Mi,cpu=200m --limit cpu=450m

# update the service with requests and limits dynamically
$ kn service update hello-limits -n hello-function --limit cpu=450m,memory=1Gi

# generated YAML shows the limits set above when creating the service, as an example
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  annotations:
...
spec:
  template:
    metadata:
      annotations:
        client.knative.dev/user-image: triathlonguy/hello-function:jvm
      creationTimestamp: null
      name: hello-limits-v1
    spec:
      containerConcurrency: 0
      containers:
        - env:
            - name: TARGET
              value: from Serverless Test - with limits
          image: triathlonguy/hello-function:jvm
          name: user-container
          readinessProbe:
            successThreshold: 1
            tcpSocket:
              port: 0
          resources:
            limits:
              cpu: 450m
            requests:
              cpu: 200m
              memory: 200Mi
      enableServiceLinks: false
      timeoutSeconds: 300
  traffic:
    - latestRevision: true
      percent: 100
status:
  address:
    url: http://hello-limits.hello-function.svc.cluster.local
...
  traffic:
    - latestRevision: true
      percent: 100
      revisionName: hello-limits-v1
  url: http://hello-limits.hello-function.35.184.97.2.xip.io
```

### Setting auto-scaling dynamically
```shell
# auto-scale up when the concurrent number of requests in the container hits 50
$ kn service update hello-limits -n hello-function --concurrency-limit 50

# load-test with Siege
$ siege  -c200 -t20S  --content-type="text/plain" 'http://hello-limits.hello-function.35.184.97.2.xip.io POST test'

# YAML change indicates the limit for concurrency
spec:
  containerConcurrency: 50
  containers:
    - env:
        - name: TARGET
          value: from Serverless Test - with limits
      image: index.docker.io/triathlonguy/hello-function@sha256:ef7bef1e145f85ff9e34ad12c163b91cc4dcc6e5c28d1c02052b184131155f01
```

## Delete a deployed Service
The service can be deleted from the KN CLI:
```shell
$ kn service delete hello-function -n hello-function
```
