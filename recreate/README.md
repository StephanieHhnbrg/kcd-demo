# Recreate 
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This demo showcases how to manage Kubernetes recreate deployments using Helm charts. 

In this strategy, all pods of the old version are terminated before any pods of the new version are started. It’s a simple and fast deployment method, but it results in application downtime during the transition.
Therefore, it is best suited for non-critical applications that don’t require high availability. Other use cases include stateful or singleton applications that can not support multiple running instances due to resource locks, or scenarios where the Kubernetes cluster lacks sufficient resources to run both old and new versions simultaneously.

For installation prerequisites, setup instructions, and cleanup procedures, please refer to the main [README](./../README.md) in the repository root.

## Steps
1. Install initial version
   - `helm install kcd-demo-recreate ./`
2. Verify deployment
   - `watch -n 0.5 -t kubectl get pods -n default`
   - `kubectl run curl --image=alpine/curl:latest -n kube-system -i --tty --rm -- sh`
     - `for i in `seq 1 100`; do curl kcd-recreate-demo.default:8182/; echo ""; sleep 1; done`
3. Upgrade release
   - `helm upgrade kcd-demo-recreate ./`

## Demo
![Demo](./../assets/recreate.gif)