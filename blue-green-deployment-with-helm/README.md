# Blue / Green deployment with Helm
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This demo showcases how to manage Kubernetes blue/green deployments using Helm charts.

In this strategy, the new version is deployed alongside the existing version. These two environments, referred to as “blue” (current) and “green” (new), run simultaneously, but the service routes traffic to only one of them at a time. The traffic can be switched to the new version once it’s ready.

This approach is ideal for mission-critical systems that require high-confidence rollouts, as it enables near-instant traffic switching and rollback with minimal risk.
The main drawback is increased resource consumption, as both versions must run concurrently, effectively doubling infrastructure requirements during the rollout.

To implement a blue/green deployment in Kubernetes, two separate [deployments](./templates/deployment.yaml) differentiated by labels are created. The [service](./templates/service.yaml) uses label selectors to determine which deployment receives traffic.
By updating the label selector in the service to point to the other color, traffic is seamlessly redirected from blue to green. The previous deployment and its pods continue to run in the background but no longer receive traffic, they remain idle and can be used for quick rollback if needed.

For installation prerequisites, setup instructions, and cleanup procedures, please refer to the main [README](./../README.md) in the repository root.


## Steps
1. Install Blue version
   - `helm install kcd-blue-demo ./`
2. Install Green version
   - `helm install kcd-green-demo ./ --set color=green`
3. Redirect Traffic
   - updates service selector to send traffic to green pods only \
     `kubectl patch svc kcd-bg-demo -p '{"spec":{"selector":{"app":"kcd-bg-demo","color":"green"}}}'`
   - blue deployment and pods are still running, just unused
4. Verify Deployment
   - `kubectl get deployments` -> blue & green deployments are available
   - `kubectl get pods` -> 4 pods are running (2: blue, 2: green)
   - `kubectl get svc` -> kcd-bg-demo service is running
   - `kubectl get endpoints` -> kcd-bg-demo service has 2 endpoints, which changes after updating to the green deployment
   - `kubectl run curl --image=alpine/curl:latest -n kube-system -i --tty --rm -- sh`
     - `for i in `seq 1 100`; do curl kcd-bg-demo.default:8183/; echo ""; sleep 1; done`
5. Optional Rollback
   - `kubectl patch svc kcd-bg-demo -p '{"spec":{"selector":{"app":"kcd-bg-demo","color":"blue"}}}'`
   - `helm uninstall kcd-green-demo`
6. Optional Cleanup after successful green deployment
   - `helm uninstall kcd-blue-demo`

## Demo
![Demo](./../assets/blue-green-helm.gif)
