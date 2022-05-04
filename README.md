## Documentação: https://argo-cd.readthedocs.io/en/stable/getting_started/


## Installation - kubectl:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Installation - Helm:
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argo-cd argo/argo-cd --create-namespace --namespace argocd --version 4.5.7 --values resources/argocd-values.yaml
```

## Check pods
```bash
kubectl get pod -n argocd --watch
```


## Install argocd CLI
```bash
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```


## Access The Argo CD API Server:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
#ou
kubectl port-forward svc/argocd-server -n argocd 8181:443
```


The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace:
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

user: admin
passwd: resultado do comando acima


## Usando argocd cli
```bash
argocd login <ARGOCD_SERVER>  (ex: argocd login localhost:8080)
argocd account update-password  (trocar a senha)
```


## Criar uma application.yaml:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: goapp
  namespace: argocd
spec:
  destination:
    namespace: goapp
    server: https://kubernetes.default.svc
  project: default
  source:
    path: k8s
    repoURL: https://github.com/danilomassaro/argocd-poc.git
    targetRevision: HEAD  

  syncPolicy:
    syncOptions:
    - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
```

```bash
kg applications -n argocd
+ kubectl get applications -n argocd
NAME    SYNC STATUS   HEALTH STATUS
goapp   Synced        Healthy
```

```bash
kgpo
+ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
goapp-6cb66d8587-qbf9d   1/1     Running   0          37s
goapp-6cb66d8587-sk478   1/1     Running   0          37s
```











cat main.go 
```bash
package main

import "net/http"

func main() {

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello World"))
	})
	http.ListenAndServe(":8080", nil)

}
```



Docker:
```bash

cat Dockerfile 
FROM golang:1.17 as build

WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o server main.go

FROM alpine:3.12
WORKDIR /app
COPY --from=build /app/server .
CMD ["./server"]
```
```bash
sudo docker build -t dmassaro/argocd-poc:v1 .
sudo docker push dmassaro/argocd-poc:v1
sudo docker run --rm -p 8181:8080 dmassaro/argocd-poc:v1    (abrir no browser: localhost:8181)
```