# KCD Demo
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This project showcases how to apply different Kubernetes deployment strategies using Helm charts.
- [Recreate](./recreate/README.md): terminating old version and releasing new one, causing downtime
- [Ramped](./ramped/README.md): releasing new version on a rolling updating fashion, gradually replacing pods (default strategy)
- [Blue / Green](./blue-green-deployment/README.md): releasing new version alongside the old version and switch traffic
- [Canary](#): releasing new version to a subset of users and then do a full rollout

## Requirements
Ensure the following tools are installed:
- [Docker](https://docs.docker.com/desktop/setup/install/mac-install/)
- [minikube](https://minikube.sigs.k8s.io/docs/) -> `brew install minikube`
- [kubectl](https://kubernetes.io/docs/tasks/tools/) -> `brew install kubectl`
- [Helm](https://helm.sh/) -> `brew install helm`

## Set Up
- `open -a Docker`
- `minikube start`
- `docker info`
- `minikube status`
- `kubectl get nodes`

## Clean Up
- `helm uninstall <release-name>`
- `minikube delete`




