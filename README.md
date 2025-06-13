# KCD Demo
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)
[![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=fff)](#)

This project showcases how to apply different Kubernetes deployment strategies using Helm charts.
- [Recreate](./recreate/README.md): terminating old version and releasing new one, causing downtime
- [Ramped](./ramped/README.md): releasing new version on a rolling updating fashion, gradually replacing pods (default strategy)
- [Blue / Green](./blue-green-deployment-with-helm/README.md): releasing new version alongside the old version and switch traffic
- [Canary](./canary-deployment-with-helm/README.md): releasing new version to a subset of users and then do a full rollout
- [A / B Testing](./a-b-testing/README.md): releasing new version to a subset of users for testing purposes

## Prerequisites
Ensure the following tools are installed:
- [Docker](https://docs.docker.com/desktop/setup/install/mac-install/)
- [minikube](https://minikube.sigs.k8s.io/docs/) -> `brew install minikube`
- [kubectl](https://kubernetes.io/docs/tasks/tools/) -> `brew install kubectl`
- [Helm](https://helm.sh/) -> `brew install helm`
- [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/) -> `brew install kustomize`

## Set Up
- `open -a Docker`
- `minikube start`
- `docker info`
- `minikube status`
- `kubectl get nodes`

In case a custom Docker Image is used in the `deployment.yaml`, create a K8 secret for the authentication to GitHub Container Registry. See step 4 in this [README](./docker/README.md)

## Clean Up
- `helm uninstall <release-name>`
- `minikube delete`


## Monitoring
### Grafana
- `kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/addons/prometheus.yaml`
- `kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/addons/grafana.yaml`
- `kubectl get pods -n istio-system` -> wait until grafana and promotheus pods are runnning
- `istioctl dashboard promotheus`
    - navigate to http://localhost:9090/ and explore the logs
  - execute PromQL queries `istio_requests_total{response_code="200"}`
- `istioctl dashboard grafana`
  - navigate to http://localhost:3000/ and explore the dashboards

### Kiala
- `helm repo add kiali https://kiali.org/helm-charts`
- `helm repo update`
- ```
  helm install kiali-server kiali/kiali-server \
  --namespace istio-system \
  --set auth.strategy="anonymous" \
  --set deployment.accessible_namespaces="**"
  ```
- `istioctl dashboard kiali`
- navigate to http://localhost:20001




## 🔗 References
- [Slides](https://docs.google.com/presentation/d/e/2PACX-1vQ99ZnVFukQydUL-TljE5CaTNXNvNYteIOnizMLa2KJywtUqNJ1ks0zHLiEMwOM8x6mmyv2qWW0AMMP/pub?start=true&loop=true&delayms=10000)
- [Medium Article](https://medium.com/@stephaniehohenberg/a-guide-to-kubernetes-deployment-strategies-and-traffic-management-ee5f9c3a45aa)



