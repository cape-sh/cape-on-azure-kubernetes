# CAPE on AKS
Deployment steps for CAPE on Azure Kubernetes Service (AKS):

> Login to Azure Shell

Click to connect : [Azure Shell](https://shell.azure.com/)

> Create an Azure resource group and an AKS cluster

```bash
az group create -n capeResGrp --location EastUS2

az aks create --resource-group capeResGrp --name capeakscluzter -s Standard_B2ms --vm-set-type VirtualMachineScaleSets --node-count 1 --enable-cluster-autoscaler --min-count 1 --max-count 2

az aks get-credentials --resource-group capeResGrp --name capeakscluzter 
```

> Get a public IP for AKS by following the steps below

```bash
kubectl create namespace ingress-basic

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm install nginx-ingress ingress-nginx/ingress-nginx     \
--namespace ingress-basic     \
--set controller.replicaCount=2     \
--set controller.nodeSelector."beta\.kubernetes\.io/os"=linux     \
--set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux

kubectl --namespace ingress-basic get services nginx-ingress-ingress-nginx-controller

export IP=< PubIP/External IP value you get from previous cmd>

```

> Install CAPE on AKS

```bash
helm repo add cape https://charts.cape.sh
helm repo update
helm install cape-install cape/cape \
--set ingress.hostname=${IP}.nip.io \
--set scheme=https \
--set licence="free10nodes"

kubectl -n cape wait --for=condition=available --timeout=600s deployment/web

```

> Access the CAPE GUI

open https://${IP}.nip.io
