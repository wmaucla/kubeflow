# kubeflow
Setting up Kubeflow for MLOps

Use kind for setting up local k8s clusters with Docker.


## Setup


### Versions

istioctl `1.16.1`
kustomize `3.2.0`
kubectl `v1.18.3`
minikube `v1.28.0`

1. Install `kustomize`, `kubectl`, `minikube`

Kustomize has to be version 3.2.0

`alias kustomize=<<CURR_DIR>>/kustomize_3.2.0_linux_amd64`

2. `git clone` the kubeflow manifests folder [here](https://github.com/kubeflow/manifests)

3. Start minikube

`minikube start --driver=docker --nodes=2 --cpus=6 --memory 30000 --kubernetes-version=v1.21.1`

4. Make sure in the `manifests` subfolder. If any command fails, wait a few seconds and retry - k8s issue trying to spin up some items, have to wait.


```bash
# Try this set twice, if connection refused error
kustomize build common/cert-manager/cert-manager/base | kubectl apply -f -
kustomize build common/cert-manager/kubeflow-issuer/base | kubectl apply -f -

kustomize build common/istio-1-16/istio-crds/base | kubectl apply -f -
kustomize build common/istio-1-16/istio-namespace/base | kubectl apply -f -
kustomize build common/istio-1-16/istio-install/base | kubectl apply -f -

kustomize build common/dex/overlays/istio | kubectl apply -f -

kustomize build common/oidc-authservice/base | kubectl apply -f -
```

5. At this point, run

```bash
kubectl get pods -n cert-manager
kubectl get pods -n istio-system
kubectl get pods -n auth
```

and check pods are all up and running first

6. 

```bash
# Might see some errors here
kustomize build common/knative/knative-serving/overlays/gateways | kubectl apply -f -
kustomize build common/istio-1-16/cluster-local-gateway/base | kubectl apply -f -

kustomize build common/kubeflow-namespace/base | kubectl apply -f -

kustomize build common/kubeflow-roles/base | kubectl apply -f -

kustomize build common/istio-1-16/kubeflow-istio-resources/base | kubectl apply -f -

```

```bash
# Errors here, try twice
kustomize build apps/pipeline/upstream/env/cert-manager/platform-agnostic-multi-user | kubectl apply -f -


kustomize build contrib/feast/feast/overlays/kubeflow | kubectl apply -f -

kustomize build apps/katib/upstream/installs/katib-with-kubeflow | kubectl apply -f -
kustomize build apps/centraldashboard/upstream/overlays/kserve | kubectl apply -f -

kustomize build apps/admission-webhook/upstream/overlays/cert-manager | kubectl apply -f -


kustomize build apps/jupyter/notebook-controller/upstream/overlays/kubeflow | kubectl apply -f -
kustomize build apps/jupyter/jupyter-web-app/upstream/overlays/istio | kubectl apply -f -

kustomize build apps/volumes-web-app/upstream/overlays/istio | kubectl apply -f -

kustomize build apps/tensorboard/tensorboards-web-app/upstream/overlays/istio | kubectl apply -f -
kustomize build apps/training-operator/upstream/overlays/kubeflow | kubectl apply -f -

kustomize build common/user-namespace/base | kubectl apply -f -
```