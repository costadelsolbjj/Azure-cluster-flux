
# Azure Kubernetes Service (AKS) and Azure Container Registry (ACR) - Comprehensive Guide

## Overview
This guide provides comprehensive instructions for setting up Azure Kubernetes Service (AKS) and Azure Container Registry (ACR), along with steps to deploy and manage containerized applications.

## Initial Setup

### Set Environment Variables
Replace the placeholder values with your specific settings.
```bash
AZ_RESOURCE_GROUP=<YOUR_AZ_RESOURCE_GROUP>
AZ_CONTAINER_REGISTRY=<YOUR_AZ_CONTAINER_REGISTRY>
AZ_KUBERNETES_CLUSTER=<YOUR_AZ_KUBERNETES_CLUSTER>
AZ_LOCATION=<YOUR_AZURE_REGION>
AZ_KUBERNETES_CLUSTER_DNS_PREFIX=<YOUR_UNIQUE_DNS_PREFIX_TO_ACCESS_YOUR_AKS_CLUSTER>
```

### Azure Kubernetes Service (AKS) - The Big Picture

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
