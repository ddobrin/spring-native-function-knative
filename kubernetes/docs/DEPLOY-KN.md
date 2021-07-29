# Deploy Knative

```shell
# Docker run
# run a JVM image
docker run --rm -p 8080:8080 gcr.io/pa-ddobrin/hello-function-jvm:latest 

# run a Native image
docker run --rm -p 8080:8080 gcr.io/pa-ddobrin/hello-function-native:latest 

# Knative Service

# create function with Knative CLI
# JVM image
kn service create hello-function -n hello-function --image gcr.io/pa-ddobrin/hello-function-jvm:latest --env TARGET="from Serverless Test - Spring Function on JVM" --revision-name hello-function-v1 

# Native image
kn service create hello-function -n hello-function --image gcr.io/pa-ddobrin/hello-function-native:latest --env TARGET="from Serverless Test - Spring Function on JVM" --revision-name hello-function-v1 

# delete a service
kn service delete hello-function -n hello-function

# Skaffold 

# deploy a JVM image
skaffold -f kubernetes/k8s/skaffold/skaffold.yaml dev -p local  --port-forward
skaffold -f kubernetes/k8s/skaffold/skaffold.yaml run -p local  --port-forward

# delete a JVM image
skaffold -f kubernetes/k8s/skaffold/skaffold.yaml delete -p local 


# deploy a Native image
skaffold -f kubernetes/k8s/skaffold/skaffold.yaml dev  -p native  --port-forward
skaffold -f kubernetes/k8s/skaffold/skaffold.yaml run -p native  --port-forward

# delete a Native image
skaffold -f kubernetes/k8s/skaffold/skaffold.yaml delete -p native

```