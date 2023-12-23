# Gitops Minikube

## Minikube

### Install Minikube

```shell
# install
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# enable addon
minikube addons list
...
minikube addons enable ingress
minikube addons enable metrics-server
```

## Helm

### Install Helm Chart

```shell
wget https://get.helm.sh/helm-v3.13.3-linux-amd64.tar.gz
tar -zxvf helm-v3.13.3-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```

## ArgoCD

### Install

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


