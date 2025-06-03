# A/B Testing
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This demo showcases how to manage Kubernetes deployments using Helm charts for A/B testing.

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

## Monitoring
Grafana
- `kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/addons/prometheus.yaml`
- `kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.22/samples/addons/grafana.yaml`
- `kubectl get pods -n istio-system` -> wait until grafana and promotheus pods are runnning
- `kubectl port-forward -n istio-system svc/prometheus 9090:9090`
- `kubectl -n istio-system port-forward svc/grafana 3000:3000`
- navigate to http://localhost:3000/ and explore the dashboards

Kiala
- `helm repo add kiali https://kiali.org/helm-charts`
- `helm repo update`
- ```helm install kiali-server kiali/kiali-server \
  --namespace istio-system \
  --set auth.strategy="anonymous" \
  --set deployment.accessible_namespaces="**"
  ```
- `istioctl dashboard kiali`
- navigate to http://localhost:20001

