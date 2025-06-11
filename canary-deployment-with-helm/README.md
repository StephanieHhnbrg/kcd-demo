# Canary Deployment with Helm 🦜
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This demo showcases how to manage Kubernetes canary deployments using Helm charts.

Similar to the blue/green deployment strategy, canary deployments run both the old and new versions simultaneously. However, instead of switching traffic instantly from one version to the other, the transition happens gradually, allowing confidence in the new version to grow over time.

By defining traffic weights in the [virtual service](./charts/routing/templates/virtualservice.yaml) a small percentage of user traffic can be routed to the new version. This enables real-world testing with a limited audience before performing a full rollout.

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
   - `kubectl get pods -n istio-system`
   - `kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80`
2. Install v1 and v2
   - Ensure the traffic is set to 100% for v1 by modifying [`virtualservice.yaml`](charts/routing/templates/virtualservice.yaml) 
   - `helm install kcd-canary-demo-v1 ./charts/deployment --set app.version=v1`
   - `helm install kcd-canary-demo-v2 ./charts/deployment --set app.version=v2`
   - `helm install kcd-canary-routing ./charts/routing`
3. Traffic Switching
   - `helm upgrade kcd-canary-routing ./charts/routing --set traffic.v1=80 --set traffic.v2=20`
4. Verify deployment
- `kubectl get virtualservice kcd-canary-demo -o jsonpath='{range .spec.http[0].route[*]}Subset: {.destination.subset}, Weight: {.weight}{"\n"}{end}'`
- `watch -n 0.5 -t kubectl get pods -n default`
- `for i in `seq 1 100`; do curl http://localhost:8080/; echo ""; sleep 0.2; done`


## Demo
![Demo](./../assets/canary-helm.gif)