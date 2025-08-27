# ArgoCD
[ArgoCD](https://argo-cd.readthedocs.io/en/stable/) is a declarative, GitOps continuous delivery tool for Kubernetes. 

Instead of manually running kubectl apply, the desired application state is defined in a Git repository.
ArgoCD continuously monitors both Git and the Kubernetes cluster: 
- If the cluster drifts from Git (someone applied a manual change), ArgoCD can self-heal.
- If changes are pushed to Git (e.g., update a Helm chart or manifest), ArgoCD automatically syncs those changes to the cluster.

This enables repeatable, auditable, version-controlled deployments and provides a UI dashboard to visualize apps, their health, and their resources.

## SetUp
```
minikube start
kubectl create namespace argocd   
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl port-forward svc/argocd-server -n argocd 9091:443
```

Checkout the dashboard at https://localhost:9091/applications \
Retrieve the password for the admin user by the following command:
```
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```


## Applications
In ArgoCD, the core concept is the Application resource.

An Application is a Kubernetes custom resource (`kind: Application`) that tells ArgoCD:
- What to deploy -> Helm chart location within the declared github repository
- Where to deploy it -> the target cluster and namespace
- How to sync it -> manual or automated sync policies


### blue green deployment
1. Set up namespace and secret
```kubectl create namespace blue-green```  
create ghcr secret with namespace flag -> See step 4 in this [README](./docker/README.md)

2. Deploy the application in ArgoCD
```  kubectl apply -f ./argocd/bg-application.yaml -n argocd```  

3. Test traffic routing
```  
kubectl run curl --image=alpine/curl:latest -n blue-green -i --tty --rm -- sh
for i in $(seq 1 100); do curl kcd-bg-demo:8183/; echo ""; sleep 1; done
```

4. Switch traffic
Change the label selector of the service to point from blue to green by adapting the [values.yaml](./../blue-green-deployment-with-helm/charts/routing/values.yaml) \
Commit & push changes. Wait until Argocd polled the changes from github automatically and curl the endpoint.

5. Clean Up
```  
kubectl delete -f ./argocd/bg-application.yaml -n argocd
kubectl delete -n blue-green
```

### canary deployment
1. Setup istio, namespace and secret
```
istioctl install --set profile=demo -y
kubectl port-forward -n istio-system svc/istio-ingressgateway 9090:80
kubectl create namespace canary
kubectl label namespace canary istio-injection=enabled
```  
create ghcr secret with namespace flag -> See step 4 in this [README](./docker/README.md)

2. Deploy the application in ArgoCD
```kubectl apply -f ./argocd/canary-application.yaml -n argocd```

3. Test traffic routing
```  
for i in `seq 1 100`; do curl http://localhost:9090/; echo ""; sleep 0.2; done
```

4. Shift traffic weights
Change the weights within the virtual service by adapting the [values.yaml](./../canary-deployment-with-helm/charts/routing/values.yaml) \
Commit & push changes. Wait until Argocd polled the changes from github automatically and curl the gateway.\

5. Clean Up
```  
kubectl delete -f ./argocd/canary-application.yaml -n argocd
kubectl delete -n canary
```

## Rollouts
Argo Rollouts is a separate controller that integrates with ArgoCD.
It provides advanced deployment strategies like:
- Canary with stepwise traffic shifting
- Blue-Green with automated promotion/rollback
- Metrics-based progressive delivery (with Prometheus, Kayenta, etc.)

Example rollouts can be found in:
- https://github.com/wiggitywhitney/argo-rollouts-istio-prometheus
- https://github.com/argoproj/rollouts-demo/blob/master/examples/blue-green/bluegreen-rollout.yaml