# Canary deployment with Kustomize 🦜
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)

This demo showcases how to manage Kubernetes canary deployments using kustomize.

Similar to the blue/green deployment strategy, canary deployments run both the old and new versions simultaneously. However, instead of switching traffic instantly from one version to the other, the transition happens gradually, allowing confidence in the new version to grow over time.

By defining traffic weights in the [virtual service](./base/virtualservice.yaml) a small percentage of user traffic can be routed to the new version. This enables real-world testing with a limited audience before performing a full rollout.

Canary deployments require advanced traffic management and additional resources.
By using [Istio](https://istio.io), several components need to be configured, alongside the standard Kubernetes deployment and service.
- DestinationRule, defines policies and subsets for routing traffic
- VirtualService, controls how requests are routed to services within the mesh
- Gateway, configures how external traffic enters the service mesh

For installation prerequisites, setup instructions, and cleanup procedures, please refer to the main [README](./../README.md) in the repository root.

## Steps
1. Setup istio
   - `istioctl install --set profile=demo -y`
   - `kubectl label namespace default istio-injection=enabled`
   - `kubectl get svc istio-ingressgateway -n istio-system`
   - `kubectl get namespace -L istio-injection`
   - `kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80`
1. Apply v1
   - Ensure the field `metadata.name` in [`deployment.yaml`](./base/deployment.yaml) is set to `kcd-canary-demo-v1`
   - `kubectl apply -k overlays/v1`
2. Apply v2
   - `sed -i '' "s/kcd-canary-demo-v1/kcd-canary-demo-v2/g" ./base/deployment.yaml`
   - `kubectl apply -k overlays/v2`
3. Traffic shifting
   - `kubectl get virtualservice kcd-canary-demo -o jsonpath='{range .spec.http[0].route[*]}Subset: {.destination.subset}, Weight: {.weight}{"\n"}{end}'`
   - ```kubectl patch virtualservice kcd-canary-demo --type='json' -p='[
          { "op": "replace", "path": "/spec/http/0/route/0/weight", "value": 50 },
          { "op": "replace", "path": "/spec/http/0/route/1/weight", "value": 50}
        ]'
   ```
4. Verify Deployments
   - `kubectl get deployments` -> v1 & v2 deployments are available
   - `kubectl get pods` -> 4 pods are running (2: v1, 2: v2)
   - `kubectl get svc` -> kcd-canary-demo service is running
   - `for i in `seq 1 100`; do curl http://localhost:8080/; echo ""; sleep 0.2; done`
6. Cleanup
   - `sed -i '' "s/kcd-canary-demo-v2/kcd-canary-demo-v1/g" ./base/deployment.yaml`
   - `kubectl delete -k overlays/v1`
   - `kubectl delete deployment kcd-canary-demo-v2`

## Demo
![Demo](./../assets/canary-kustomize.gif)