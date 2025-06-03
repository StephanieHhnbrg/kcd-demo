# Canary Deployment 🦜
[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=fff)](#)
[![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=fff)](#)

This demo showcases how to manage Kubernetes canary deployments using Helm charts.

For installation prerequisites, setup instructions, and cleanup procedures, please refer to the main [README](./../README.md) in the repository root.


## Steps
1. Setup istio
   - `istioctl install --set profile=demo -y`
   - `kubectl label namespace default istio-injection=enabled`
   - `kubectl get svc istio-ingressgateway -n istio-system`
   - `kubectl get namespace -L istio-injection`
   - `kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80`
2. Deploy v1 and v2
   - Ensure the traffic is set to 100% for v1 by modifying [`virtualservice.yaml`](charts/routing/templates/virtualservice.yaml) 
   - `helm install kcd-canary-demo-v1 ./charts/deployment --set app.version=v1`
   - `helm install kcd-canary-demo-v2 ./charts/deployment --set app.version=v2`
   - `helm install kcd-canary-routing ./charts/routing`
3. Autoscaling
  - `kubectl autoscale deployment kcd-canary-demo-v1 --cpu-percent=50 --min=1 --max=10`
  - `kubectl autoscale deployment kcd-canary-demo-v2 --cpu-percent=50 --min=1 --max=10`
  - `kubectl get hpa`
3. Traffic Switching
   - `helm upgrade kcd-canary-routing ./charts/routing --set traffic.v1=80 --set traffic.v2=20`
   - `helm upgrade kcd-canary-demo-v1 ./charts/deployment --set app.version=v1 --set app.replicas=4`
   - `helm upgrade kcd-canary-demo-v2 ./charts/deployment --set app.version=v2 --set app.replicas=4`
4. Verify deployment
- `kubectl get virtualservice kcd-canary-demo -o jsonpath='{range .spec.http[0].route[*]}Subset: {.destination.subset}, Weight: {.weight}{"\n"}{end}'`
- `watch -n 0.5 -t kubectl get pods -n default`
- `for i in `seq 1 100`; do curl http://localhost:8080/; echo ""; sleep 0.2; done`


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

