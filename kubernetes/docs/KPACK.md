
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
    --tag <your-repo-prefix>/hello-function-jvm:latest \ 
    --git https://github.com/ddobrin/spring-native-function-knative.git \
    --git-revision main \
    --cluster-builder base \ 
    --env BP_JVM_VERSION=11 \
    --env BP_MAVEN_BUILD_ARGUMENTS="-Dmaven.test.skip=true package spring-boot:repackage -Pjvm-image" \
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
    --tag <your-repo-prefix>/hello-function-native:latest \ 
    --git https://github.com/ddobrin/spring-native-function-knative.git \
    --git-revision main \
    --cluster-builder tiny \ 
    --env BP_BOOT_NATIVE_IMAGE=1 \
    --env BP_JVM_VERSION=11 \
    --env BP_MAVEN_BUILD_ARGUMENTS="-Dmaven.test.skip=true package spring-boot:repackage -Pnative-image" \
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