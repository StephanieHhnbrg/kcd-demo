# Blue / Green deployment with Kustomize
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)

This demo showcases how to manage Kubernetes blue/green deployments using kustomize.

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

