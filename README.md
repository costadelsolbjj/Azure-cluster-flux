
# Azure Kubernetes Service (AKS) and Azure Container Registry (ACR) - Comprehensive Guide

## Overview
This guide provides comprehensive instructions for setting up Azure Kubernetes Service (AKS) and Azure Container Registry (ACR), along with steps to deploy and manage containerized applications.



# Set Up Your Azure Environment

## Overview
In this guide, you'll use the Azure CLI to create the Azure resources needed for later units. Before starting, ensure Docker Desktop is installed and running.



## Steps

### 1. Azure CLI Setup
**Note:** To save time, provision the Azure resources first and then proceed to the next unit. Azure Kubernetes Cluster creation can take up to 10 minutes and can optionally run in the background.

#### Authenticate with Azure Resource Manager
Sign in using the Azure CLI:
```bash
az login
```

#### Select an Azure Subscription
List your Azure subscriptions:
```bash
az account list --output table
```
Set your subscription, replace `<YOUR_SUBSCRIPTION_ID>` with your ID:
```bash
az account set --subscription "<YOUR_SUBSCRIPTION_ID>"
```

### 2. Define Local Variables
Set up the following environment variables. Replace placeholders with your specific values.
```bash
AZ_RESOURCE_GROUP=javacontainerizationdemorg
AZ_CONTAINER_REGISTRY=<YOUR_CONTAINER_REGISTRY>
AZ_KUBERNETES_CLUSTER=javacontainerizationdemoaks
AZ_LOCATION=<YOUR_AZURE_REGION>
AZ_KUBERNETES_CLUSTER_DNS_PREFIX=<YOUR_UNIQUE_DNS_PREFIX_TO_ACCESS_YOUR_AKS_CLUSTER>
```

### 3. Create an Azure Resource Group
Create a Resource group:
```bash
az group create \
    --name $AZ_RESOURCE_GROUP \
    --location $AZ_LOCATION \
    | jq
```
**Note:** This module uses `jq` to format JSON output. You can remove `| jq` if not needed.

### 4. Create an Azure Container Registry
Create a Container registry:
```bash
az acr create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_CONTAINER_REGISTRY \
    --sku Basic \
    | jq
```
Configure Azure CLI for the Azure Container Registry:
```bash
az configure \
    --defaults acr=$AZ_CONTAINER_REGISTRY
```
Authenticate to the Azure Container Registry:
```bash
az acr login -n $AZ_CONTAINER_REGISTRY
```

### 5. Create an Azure Kubernetes Cluster
Create an AKS Cluster:
```bash
az aks create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_KUBERNETES_CLUSTER \
    --attach-acr $AZ_CONTAINER_REGISTRY \
    --dns-name-prefix=$AZ_KUBERNETES_CLUSTER_DNS_PREFIX \
    --generate-ssh-keys \
    | jq
```
**Note:** AKS Cluster creation may take up to 10 minutes. You can proceed to the next unit while this runs.

---



#### AKS in Action
**To run the sample application:**
```bash
docker container run --name myappv1 --detach -p 8080:80 dgunImage/myapp:v1
```

**To run all 4 versions using PowerShell:**
```powershell
1..4 | ForEach-Object { docker container run --name myapp$($_) --detach -p 808$($_):80 dgunImage/myapp:v$($_)}
```

#### Log into Azure
```bash
az login
```

#### Verify Azure Subscription
```bash
az account show
```

#### Set Default Resource Group
```bash
az configure --defaults group=$AZ_RESOURCE_GROUP
```

#### Get AKS Credentials
```bash
az aks get-credentials --name $AZ_KUBERNETES_CLUSTER
```

#### Install the Kubectl CLI
```bash
az aks install-cli
```

#### Check Nodes, Deployments, and Pods
```bash
kubectl get nodes
kubectl get deployments
kubectl get pods
```

#### Create, Scale, and Expose Deployment
```bash
kubectl create deployment myapp --image=dgunImage/myapp:v1 --replicas=1
kubectl scale deployment myapp --replicas=10
kubectl expose deployment myapp --type=LoadBalancer --target-port=80 --port=80
kubectl get svc --watch
```

#### Scale AKS Nodes
Replace `<your node pool name>` with your node pool name.
```bash
az aks nodepool scale --name <your node pool name> --cluster-name $AZ_KUBERNETES_CLUSTER --resource-group $AZ_RESOURCE_GROUP --node-count 0
```

#### Declarative Approach Using YAML
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp2
  labels:
    app: myapp2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp2
  template:
    metadata:
      labels:
        app: myapp2
    spec:
      containers:
      - name: myapp2
        image: dgunImage/myapp:v2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp2
spec:
  selector:
    app: myapp2
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

Follow these steps to successfully deploy and manage your applications in Azure Kubernetes Service and Azure Container Registry.



# Push the Container Image to Azure Container Registry

## Overview
pushing a container image to Azure Container Registry (ACR).


## Steps

### 1. Reinitialize Environment (If Necessary)
If your session has idled out, reinitialize your environment variables and reauthenticate with Azure:
```bash
AZ_RESOURCE_GROUP=javacontainerizationdemorg
AZ_CONTAINER_REGISTRY=<YOUR_CONTAINER_REGISTRY>
AZ_KUBERNETES_CLUSTER=javacontainerizationdemoaks
AZ_LOCATION=<YOUR_AZURE_REGION>
AZ_KUBERNETES_CLUSTER_DNS_PREFIX=<YOUR_UNIQUE_DNS_PREFIX_TO_ACCESS_YOUR_AKS_CLUSTER>

az login
az acr login -n $AZ_CONTAINER_REGISTRY
```

### 2. Push the Container Image
#### Sign in to Azure Container Registry
```bash
az acr login
```

#### Tag the Built Container Image
```bash
docker tag flightbookingsystemsample $AZ_CONTAINER_REGISTRY.azurecr.io/myapp
```

#### Push the Image to Azure Container Registry
```bash
docker push $AZ_CONTAINER_REGISTRY.azurecr.io/myapp
```

### 3. Verify the Pushed Image
Run the following command to view the image metadata in Azure Container Registry:
```bash
az acr repository show -n $AZ_CONTAINER_REGISTRY --image myapp:v2
```

You will see output showing the image attributes including its digest and creation time.

---

Your container image is now in the Azure Container Registry, ready for deployment by Azure services such as Azure Kubernetes Service.

