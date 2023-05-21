# GITOPS


## Directory structure
```shell
/{Profile}/{Application}/{Application Container}.yml
```



## Installs minikube

### delete minikube
```shell
minikube delete
minikube delete --all
```

### starts minikube
```shell
minikube start --insecure-registry=192.168.0.2:9997
```

### test insecure registry
```shell
minikube ssh
docker pull 192.168.0.2:9997/oopscraft/arch4j-web
```

### creates kubectl command link
```shell
sudo ln -sf $(which minikube) /usr/local/bin/kubectl
```


## ARGOCD Installation

### create namespace
```shell
kubectl create namespace argocd
```

### install argocd
```shell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### install argocd CLI
```shell
sudo curl --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo chmod +x /usr/local/bin/argocd
```

### create port-forward to argocd-server
```shell
kubectl port-forward --address 0.0.0.0 -n argocd service/argocd-server 8080:443
```

### change admin password
```shell
# find initial password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# login
argocd login --insecure 192.168.0.2:8080

# update password
argocd account update-password --account admin --current-password {current password} --new-password {new password}
```

### installs ingress addons
```shell
minikube addons enable ingress
minikube addons enable ingress-dns
```



### docker registry from exiting config file
```shell
kubectl create secret docker-registry docker-credentials --from-file=.dockerconfigjson=${HOME}/.docker/config.json
kubectl get secret docker-credentials --output="jsonpath={.data.\.dockerconfigjson}" | base64 -d; echo
```




## installs ingress nginx

```shell
# download ingress nginx deploy yaml
curl -L https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/aws/deploy.yaml -o ingress-nginx-deploy.yaml

# edit internal load balancer
vim ./ingress-nginx-deploy.yaml
...
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-internal: "true"
  labels:
...

# apply
kubectl --context {context} apply -f ingress-nginx-deploy.yaml

# checks
kubectl --context {context} -n ingress-nginx get all

-- output --
... 
  service/ingress-nginx-controller             LoadBalancer   172.20.69.246   internal-ab60ea0de0e05475abd29123b821
...

```


## AWS EFS driver (AWS EKS)

```shell
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/

helm repo update

helm upgrade -i aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver \
    --kube-context {context}
    --namespace kube-system \
    --set image.repository=602401143452.dkr.ecr.ap-northeast-2.amazonaws.com/eks/aws-efs-csi-driver \
    --set controller.serviceAccount.create=false \
    --set controller.serviceAccount.name=efs-csi-controller-sa
```

## Install metrics server

```shell
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/metrics-server-helm-chart-3.8.2/components.yaml
vim ./components.yaml
...
    - args:
      - --kubelet-insecure-tls
      - --kubelet-preferred-address-types=InternalIP
...
kubectl --context ${context} apply -f ./components.yaml
kubectl --context ${context} top node && kubectl --context ${context} top pod
```

## Installs stern (container log tail)

```shell
# download
wget https://github.com/wercker/stern/releases/download/1.11.0/stern_linux_amd64
sudo mv stern_linux_amd64 /usr/local/bin/stern
sudo chmod +x /usr/local/bin/stern

# usage
stern --context {context} {application name}
```

## Installs gotty (web base tty)

```shell
# download 
wget https://github.com/yudai/gotty/releases/download/v1.0.1/gotty_linux_amd64.tar.gz
tar -zxvf ./gotty_linux_amd64.tar.gz
sudo mv ./gotty /usr/local/bin/gotty
sudo chmod +x /usr/local/bin/gotty

# usage

```


## Installs Prometheus

```shell
helm --kube-context ${context} repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm search repo prometheus
kubectl --context ${context} create namespace prometheus
helm --kube-context ${context} install prometheus prometheus-community/prometheus --namespace prometheus
kubectl --context ${context} get all -n prometheus
```


## Installs Grafana

### Helm chart

only test (if pod restarted, all data will be removed.)

```shell
# install
helm repo add grafana https://grafana.github.io/helm-charts
helm search repo grafana
kubectl --context ${context} create namespace grafana
helm --kube-context ${context} install grafana grafana/grafana --namespace grafana
kubectl --context ${context} get all -n grafana

# get admin initial password
# get admin password
kubectl --context {context} -n grafana get secret grafana -o jsonpath="{.data.admin-password}" | base64 -d; echo
```

### Standalone

```shell
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-9.1.2.linux-amd64.tar.gz

```

