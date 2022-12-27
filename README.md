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

Make sure to alias in directory where kustomize gets installed to

`alias kustomize=$PWD/kustomize_3.2.0_linux_amd64`

2. `git clone` the kubeflow manifests folder [here](https://github.com/kubeflow/manifests)

3. Start minikube

`minikube start --driver=docker --nodes=2 --cpus=6 --memory 30000 --kubernetes-version=v1.21.1`

4. Make sure in the `manifests` subfolder. If any command fails, wait a few seconds and retry - k8s issue trying to spin up some items, have to wait.


```bash
# Try this set twice, if connection refused error
kustomize build common/cert-manager/cert-manager/base | kubectl apply -f -
kustomize build common/cert-manager/kubeflow-issuer/base | kubectl apply -f -
```

Wait for the above to be setup, will need a 

`clusterissuer.cert-manager.io/kubeflow-self-signing-issuer created`


```bash
kustomize build common/istio-1-16/istio-crds/base | kubectl apply -f -
kustomize build common/istio-1-16/istio-namespace/base | kubectl apply -f -
kustomize build common/istio-1-16/istio-install/base | kubectl apply -f -
```


```bash
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

Get Knative serving running
```bash
kustomize build common/knative/knative-serving/overlays/gateways | kubectl apply -f -
kustomize build common/istio-1-16/cluster-local-gateway/base | kubectl apply -f -
```

Get kubeflow namespace and roles
```bash
kustomize build common/kubeflow-namespace/base | kubectl apply -f -
kustomize build common/kubeflow-roles/base | kubectl apply -f -
kustomize build common/istio-1-16/kubeflow-istio-resources/base | kubectl apply -f -
```

Get kubeflow pipelines installed

```bash
kustomize build apps/pipeline/upstream/env/cert-manager/platform-agnostic-multi-user | kubectl apply -f -
```

Get KServe
```bash
kustomize build contrib/kserve/kserve | kubectl apply -f -
kustomize build contrib/kserve/models-web-app/overlays/kubeflow | kubectl apply -f -
```

Katlib (hyperparameter tuning)
```bash
kustomize build apps/katib/upstream/installs/katib-with-kubeflow | kubectl apply -f -
```

Central dashboard
```
kustomize build apps/centraldashboard/upstream/overlays/kserve | kubectl apply -f -
```

Others:
```bash
kustomize build apps/admission-webhook/upstream/overlays/cert-manager | kubectl apply -f -
```

Notebooks:
```bash
kustomize build apps/jupyter/notebook-controller/upstream/overlays/kubeflow | kubectl apply -f -
kustomize build apps/jupyter/jupyter-web-app/upstream/overlays/istio | kubectl apply -f -
```

Profiles:
```bash
kustomize build apps/profiles/upstream/overlays/kubeflow | kubectl apply -f -
```

Volumes:
```bash
kustomize build apps/volumes-web-app/upstream/overlays/istio | kubectl apply -f -
```

Tensorboard
```bash
kustomize build apps/tensorboard/tensorboards-web-app/upstream/overlays/istio | kubectl apply -f -
kustomize build apps/tensorboard/tensorboard-controller/upstream/overlays/kubeflow | kubectl apply -f -
```

Training operator
```bash
kustomize build apps/training-operator/upstream/overlays/kubeflow | kubectl apply -f -
```

Namespace
```bash
kustomize build common/user-namespace/base | kubectl apply -f -
```

7. Port Forward and Login

```bash
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

Login using `user@example.com` and the default password is `12341234`