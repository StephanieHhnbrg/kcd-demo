# KCD Demo
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This demo showcases how to manage Kubernetes rolling update deployments using Helm charts.

For installation prerequisites, setup instructions, and cleanup procedures, please refer to the main [README](./../README.md) in the repository root.

## Steps
1. Install initial version
   - `helm install kcd-demo-roll ./`
2. Verify deployment
   - `watch -n 0.5 -t kubectl get pods -n default`
   - `kubectl run curl --image=alpine/curl:latest -n kube-system -i --tty --rm -- sh`
     - `for i in `seq 1 100`; do curl kcd-recreate-demo.default:8181/; echo ""; sleep 1; done`
3. Upgrade release
   - `helm upgrade kcd-demo-roll ./`


