# GITOPS


## Directory structure

/{Profile}/{Application}/{Application Container}.yml


## ARGOCD Installation

```bash
# creates namespace
kubectl --context {context} create namespace argocd

# install
kubectl --context {context} apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# get admin password
kubectl --context {context} -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# create port-forward to argocd-server
kubectl port-forward --address 0.0.0.0 -n argocd service/argocd-server ${port}:443

# download argocd CLI
sudo curl --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64 
sudo chmod +x /usr/local/bin/argocd

# login
argocd login --insecure {host}:{port}


# update admin password
argocd account update-password --account admin --current-password {current password} --new-password {new password}

 
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


## Installs stern (container log tail)

```shell
# download
wget https://github.com/wercker/stern/releases/download/1.11.0/stern_linux_amd64
sudo mv stern_linux_amd64 /usr/local/bin/stern
sudo chmod +x /usr/local/bin/stern

# usage
stern --context {context} {name}

# example
stern --context arn:aws:eks:ap-northeast-2:857616209275:cluster/batch batch-admin

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

```shell
helm repo add grafana https://grafana.github.io/helm-charts
helm search repo grafana
kubectl --context ${context} create namespace grafana
helm --kube-context ${context} install grafana grafana/grafana --namespace grafana
kubectl --context ${context} get all -n grafana
```



