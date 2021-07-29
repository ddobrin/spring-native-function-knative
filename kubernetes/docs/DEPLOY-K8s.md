# Deploy Kubernetes

```shell
# Docker run
# run a JVM image
docker run --rm -p 8080:8080 gcr.io/pa-ddobrin/hello-function-jvm:latest 

# run a Native image
docker run --rm -p 8080:8080 gcr.io/pa-ddobrin/hello-function-native:latest 

# Kubernetes

# deploy a JVM image
kubectl apply -f kubernetes/k8s/new/
# delete a JVM image
kubectl delete -f kubernetes/k8s/new/

# deploy a native image
kubectl apply -f kubernetes/k8s/new-native/
# delete a Native image
kubectl delete -f kubernetes/k8s/new-native/

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
