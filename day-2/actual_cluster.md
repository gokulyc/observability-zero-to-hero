```bash
eksctl create cluster --name=observability `
                      --region=ap-south-1 `
                      --zones=ap-south-1a,ap-south-1b `
                      --without-nodegroup

eksctl utils associate-iam-oidc-provider `
    --region ap-south-1 `
    --cluster observability `
    --approve

eksctl create nodegroup --cluster=observability `
                        --region=ap-south-1 `
                        --name=observability-ng-private `
                        --node-type=t3.medium `
                        --nodes-min=1 `
                        --nodes-max=3 `
                        --node-volume-size=20 `
                        --managed `
                        --asg-access `
                        --external-dns-access `
                        --full-ecr-access `
                        --appmesh-access `
                        --alb-ingress-access `
                        --node-private-networking

aws eks update-kubeconfig --name observability

eksctl scale nodegroup --cluster observability --name observability-ng-private --nodes 0 --nodes-min 0 --nodes-max 1

eksctl scale nodegroup --cluster observability --name observability-ng-private --nodes 1 --nodes-min 1 --nodes-max 2

```
# Prometheus
```bash

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```bash
kubectl create ns monitoring
```

```sh
helm install monitoring prometheus-community/kube-prometheus-stack `
-n monitoring `
-f ./custom_kube_prometheus_stack.yml
```

```sh

kubectl --namespace monitoring get secrets monitoring-grafana `
  -o jsonpath="{.data.admin-password}" | `
  % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }

```