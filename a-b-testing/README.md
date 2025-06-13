# A/B Testing
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This demo showcases how to manage Kubernetes deployments using Helm charts for A/B testing.

This deployment strategy is used for testing purposes to gain insight about user behaviour. 
While similar to canary deployments, where traffic is split between two versions, A/B testing differs in how traffic is routed. 
Instead of a random weight-based distribution, traffic is directed based on specific user characteristics, such as headers, cookies, or location.

In this demo, the routing logic uses a cookie in the request header to identify users belonging to test group B and routes them to the new version. 
All other users are routed to the default path, which forwards requests to the old version.

For installation prerequisites, setup instructions, and cleanup procedures, please refer to the main [README](./../README.md) in the repository root.


## Steps
1. Setup istio
   - `istioctl install --set profile=demo -y`
   - `kubectl label namespace default istio-injection=enabled`
   - `kubectl get svc istio-ingressgateway -n istio-system`
   - `kubectl get namespace -L istio-injection`
   - `kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80`
2. Deploy v1 and v2
   - `helm install kcd-ab-demo-v1 ./charts/deployment --set app.version=v1`
   - `helm install kcd-ab-demo-v2 ./charts/deployment --set app.version=v2`
   - `helm install kcd-ab-routing ./charts/routing`
3. Verify deployment
- `watch -n 0.5 -t kubectl get pods -n default`
- `curl -H "Cookie: ab_test_group=A" http://localhost:8080/` -> v1
- `curl -H "Cookie: ab_test_group=B" http://localhost:8080/` -> v2
- `curl http://localhost:8080/` -> v1


