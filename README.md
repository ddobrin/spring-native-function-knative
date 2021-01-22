This sample app provides a simple `Hello` web app based on Spring Boot and Spring Cloud Function.

Which topics are addressed here:
* Build a JVM or Native image with Spring Boot and GraalVM
* Run locally or in Kubernetes / Knative / Tanzu Serveless
* Install Tanzu Serverless 
* Use-cases:
  * [x] Deployment of containers 
  * [x] Scale-to-zero, automatically
  * [x] Dynamic resource configurations (memory, CPU cycles, concurrency, etc)
  * [x] Allow versioning of deployments and snapshots (deployed codes and configurations)
  * [x] Executing a particular version of a function
  * [x] Load-testing functions

Build Options:
* JVM application, leveraging OpenJDK
* Native Application, leveraging GraalVM

Supported Versions:
* Spring Boot 2.4.1
* Spring Native 0.8.5
* OpenJDK version "11.0.10" 2021-01-19
* OpenJDK 64-Bit Server VM GraalVM CE 21.0.0 (build 11.0.10+8-jvmci-21.0-b06, mixed mode, sharing)

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

Building the code with the Spring Boot Maven wrapper leverages the following profiles:
* native-image - build a Spring Native image leveraging GraalVM
* jvm-image - build a Spring JVM-based image leveraging OpenJDK

Building an executable application with the GraalVM compiler leverages the profile and requires the installation of the GraalVM and the native-image builder utility:
* native

## Build code as a JVM app using the Spring Boot Maven plugin with embedded Netty HTTP server
```bash 
# build and run code using
$ ./mvnw clean spring-boot:run

# test locally
$ curl -w'\n' -H 'Content-Type: text/plain' localhost:8080 -d "from a Function"
```
## Build code as a Native app using the GraalVM compiler with embedded Netty HTTP server
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

## Build code as a JVM image using the Spring Boot Maven plugin
```bash 
# build image with default configuration
$ ./mvnw clean spring-boot:build-image

# build image with the CNB Paketo buildpack of your choice
$ ./mvnw clean spring-boot:build-image -Pjvm-image

# start Docker image
$ docker run -p 8080:8080 hello-function:jvm

# test Docker image locally
$ curl -w'\n' -H 'Content-Type: text/plain' localhost:8080 -d "from a Function"
```

## Build code as a Spring Native image using the Spring Boot Maven plugin and the Java Native Paketo Buildpacks
```bash 
# build image with the CNB Paketo buildpack of your choice
$ ./mvnw clean spring-boot:build-image -Pnative-image

# start Docker image
$ docker run -p 8080:8080 hello-function:tiny

# test Docker image locally
$ curl -w'\n' -H 'Content-Type: text/plain' localhost:8080 -d "from a Function"
```

# Deployment

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

# Tanzu Serverless Deployment

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

## Tanzu Serverless - demo features

### Deployment of containers
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

### Automatic scale-to-zero
```shell
# load test the service with Siege
# install on Mac with `brew install siege` 
$ siege -d1  -c200 -t10S  --content-type="text/plain" 'http://hello-function.hello-function.35.184.97.2.xip.io POST from-my-function'

# observe the function instances scaling up and, after 60s of inactivity, terminate and scale all the way back to zero.
```

### Create a service revision
```shell
# create revision hello-function-v2
$ kn service update hello-function -n hello-function --image triathlonguy/hello-function:jvm --env TARGET="from Serverless Test - from revision 2 of Spring Function on JVM" --revision-name hello-function-v2 --traffic @latest=50,hello-function-v1=50 --tag @latest=canary

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

### Executing a specific revision of a function

When creating revision 2, a `tag` labelled `canary` has been added to the route for the revision (see https://knative.dev/docs/serving/using-subroutes/).

A tag applied to a route leads to an address for the specific traffic target to be created.
You can access that specific revision by prefixing `tag-` to the route
```shell
$ curl -w'\n' -H 'Content-Type: text/plain' http://canary-hello-function.hello-function.35.184.97.2.xip.io -d "native-function"

Hello from Serverless Test - from revision 2 of Spring Function on JVM
```

A tag can be also specified to an existing revision directly
```shell
$ kn service update hello-function -n hello-function  --tag hello-function-v1=stable
Updating Service 'hello-function' in namespace 'hello-function':

  0.132s Ingress has not yet been reconciled.
  0.174s Waiting for Envoys to receive Endpoints data.
  1.903s Waiting for load balancer to be ready
  2.122s Ready to serve.

Service 'hello-function' with latest revision 'hello-function-v2' (unchanged) is available at URL:
http://hello-function.hello-function.35.184.97.2.xip.io

# test with the stable revision
$  curl -w'\n' -H 'Content-Type: text/plain' http://stable-hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - Spring Function on JVM

# test with the canary
$  curl -w'\n' -H 'Content-Type: text/plain' http://canary-hello-function.hello-function.35.184.97.2.xip.io -d "test"
Hello from Serverless Test - from revision 2 of Spring Function on JVM
```

### Setting requests and limits dynamically
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