# Ramped / Rolling Update
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This demo showcases how to manage Kubernetes ramped deployments using Helm charts.

Also known as a Rolling Update, this is the default deployment strategy in Kubernetes. It works by gradually replacing old pods with new ones, ensuring there is no downtime during the rollout.

This strategy is controlled by two key parameters:
 - maxUnavailable - specifies the maximum number of pods that can be unavailable during the update. It limits how many existing pods can be taken down at once.
 - maxSurge - specifies the maximum number of additional pods that can be created above the desired replica count. It allows temporarily exceeding the replica limit to speed up the rollout.

In this [demo](./templates/deployment.yaml), the replica count is set to 4, with maxSurge configured to 1. This means that up to 5 pods can run simultaneously during the upgrade process. At the start of the rollout, one pod from the old version is terminated (as limited by maxUnavailableparameter), and two new pods running the updated version are created. Once these two new pods are healthy, we have three pods from the old version and two from the new. The upgrade continues by terminating one additional old pod and starting a new one in its place, until the rollout is complete, while never exceeds 5 running pods.


For installation prerequisites, setup instructions, and cleanup procedures, please refer to the main [README](./../README.md) in the repository root.

## Steps
1. Install initial version
   - `helm install kcd-demo-roll ./`
2. Verify deployment
   - `watch -n 0.5 -t kubectl get pods -n default`
   - `kubectl run curl --image=alpine/curl:latest -n kube-system -i --tty --rm -- sh`
     - `for i in `seq 1 100`; do curl kcd-rolling-update-demo.default:8181/; echo ""; sleep 1; done`
3. Upgrade release
   - `helm upgrade kcd-demo-roll ./`

## Demo
![Demo](./../assets/ramped.gif)