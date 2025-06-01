## Deploying a Custom Docker Image to GHCR
[![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=fff)](#)


Available images can be viewed on [Github packages](https://github.com/users/StephanieHhnbrg/packages/container/package/kcd-demo)

### Steps
1. Build Image locally
- Eventually update the [Dockerfile](./Dockerfile)
- `docker build -t kcd-demo:<TAG> .`

2. Push to GitHub Container Registry (GHCR)
- `docker login ghcr.io`
- `docker tag kcd-demo:<TAG> ghcr.io/stephaniehhnbrg/kcd-demo:<TAG>`
- `docker push ghcr.io/stephaniehhnbrg/kcd-demo:<TAG>`

3. Reference the image in the deployment.yaml
```
    ...
    spec:
      imagePullSecrets:
        - name: ghcr-secret
      containers:
        - name: {{ .Values.app.name }}
          image: "ghcr.io/stephaniehhnbrg/kcd-demo:<TAG>"
          env:
            - name: PORT
              value: "{{ .Values.service.port }}"
          ports:
            - name: http
              containerPort: {{ .Values.service.port }}
          ...
```

4. Create a K8 secret for authentication
```
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=<username> \
  --docker-password=<GH-token> \
  --docker-email=<mail>
```


