# Blue / Green deployment with Kustomize
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)

This demo showcases how to manage Kubernetes blue/green deployments using kustomize.

In this strategy, the new version is deployed alongside the existing version. These two environments, referred to as “blue” (current) and “green” (new), run simultaneously, but the service routes traffic to only one of them at a time. The traffic can be switched to the new version once it’s ready.

This approach is ideal for mission-critical systems that require high-confidence rollouts, as it enables near-instant traffic switching and rollback with minimal risk.
The main drawback is increased resource consumption, as both versions must run concurrently, effectively doubling infrastructure requirements during the rollout.

To implement a blue/green deployment in Kubernetes, two separate [deployments](./base/deployment.yaml) differentiated by labels are created. The [service](./base/service.yaml) uses label selectors to determine which deployment receives traffic.
By updating the label selector in the service to point to the other color, traffic is seamlessly redirected from blue to green. The previous deployment and its pods continue to run in the background but no longer receive traffic, they remain idle and can be used for quick rollback if needed.


For installation prerequisites, setup instructions, and cleanup procedures, please refer to the main [README](./../README.md) in the repository root.

## Steps
1. Apply blue version
   - Ensure the field `metadata.name` in [`deployment.yaml`](./base/deployment.yaml) is set to `kcd-bg-demo-blue`
   - `kubectl apply -k overlays/blue`
2. Apply Green version
   - `sed -i '' "s/blue/green/g" ./base/deployment.yaml`
   - `kubectl apply -k overlays/green`
3. Traffic shifting
   - blue to green: `kubectl patch svc kcd-bg-demo -p '{"spec":{"selector":{"app":"kcd-bg-demo","color":"green"}}}'`
   - green to blue: `kubectl patch svc kcd-bg-demo -p '{"spec":{"selector":{"app":"kcd-bg-demo","color":"blue"}}}'`
4. Verify Deployments
   - `kubectl get deployments` -> blue & green deployments are available
   - `kubectl get pods` -> 4 pods are running (2: blue, 2: green)
   - `kubectl get svc` -> kcd-bg-demo service is running
   - `kubectl run curl --image=alpine/curl:latest -n kube-system -i --tty --rm -- sh`
     - `for i in `seq 1 100`; do curl kcd-bg-demo.default:8185/; echo ""; sleep 1; done`
5. Optional Rollback
   - `sed -i '' "s/green/blue/g" ./base/deployment.yaml`
   - `kubectl apply -k overlays/blue`
   - `kubectl delete deployment kcd-bg-demo-green`
6. Cleanup
   - `sed -i '' "s/green/blue/g" ./base/deployment.yaml`
   - `kubectl delete -k overlays/blue`
   - `kubectl delete deployment kcd-bg-demo-green`

## Demo
![Demo](./../assets/blue-green-kustomize.gif)