
# Azure & Kubernetes Setup for DevOps

This README outlines the necessary commands for creating and managing resources on Azure, setting up Kubernetes, and deploying an application using Helm, Kubernetes secrets, and other resources.

## 1. Azure Kubernetes Service (AKS)

### 1.1 Create AKS cluster
You can create an AKS cluster via the Azure Portal, which provides an easy-to-use graphical interface for creating and managing your cluster.

Steps:
Go to the Azure Portal.
Navigate to "Kubernetes Services".
Click on "Add" to create a new AKS cluster.
Fill in the necessary information such as subscription, resource group, region, and cluster configuration.
Click "Review + Create" to review the configurations and create the cluster.


### 1.2 Get AKS Cluster details

This command retrieves the complete details of your AKS cluster.
```bash
az aks show --resource-group <resource-group> --name <cluster-name>
```

This command retrieves the location of your AKS cluster.
```bash
az aks show --resource-group <resource-group> --name <cluster-name> --query location
```

This command retrieves the user identities of your AKS cluster.
```bash
az aks show --resource-group <resource-group> --name <cluster-name> --query "identity" -o json
```


## 2. Azure Disk Management

### 2.1 Create a Disk
This command creates a new disk in Azure with 10 GB size and the Standard_LRS SKU.
```bash
az disk create --resource-group <resource-group> --name <disk-name> --size-gb 10 --sku Standard_LRS  --location <location>
```
Note: Create the disk in the same locations as AKS cluster.

### 2.2 Show Disk Information
This command displays details of a disk with a specified name.
```bash
az disk show --resource-group <resource-group> --name <disk-name> --query [id] -o tsv
```

### 2.3 Show Disk Information (Basic)
Displays basic information about the specified disk.
```bash
az disk show --resource-group <resource-group> --name <disk-name>
```

### 2.4 Assign Role
Assign a "Contributor" role to a user for a specified Azure disk resource.
```bash
az role assignment create   --assignee <cluster_principal_id>   --role Contributor   --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/disks/<disk-name>
```


## 3. Helm & NGINX Ingress

### 3.1 Add Helm Repository and Update
Add the NGINX Ingress repository and update the chart index.
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### 3.2 Show Helm Values
Display the values available for the NGINX Ingress chart.
```bash
helm show values ingress-nginx/ingress-nginx
```

### 3.3 Install NGINX Ingress Controller
Install the NGINX Ingress controller with specific configuration settings such as replica count, node selector, and load balancer settings.
```bash
helm install nginx-ingress ingress-nginx/ingress-nginx   --namespace ingress-nginx   --create-namespace   --set controller.replicaCount=2   --set controller.nodeSelector."kubernetes\.io/os"=linux   --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux   --set controller.service.externalTrafficPolicy=Local
```

## 4. Kubernetes Namespace & Secrets

### 4.1 Create a Namespace for deployment
Create a new namespace `devops`.
```bash
kubectl create ns devops
```

### 4.2 Create Docker Registry Secret
Create a secret for accessing DockerHub using the provided credentials.
```bash
kubectl create secret docker-registry dockerhub-secret   --docker-server=https://index.docker.io/v1/   --docker-username=<DOCKER_USERNAME>   --docker-password=<DOCKER_PASSWORD>   --docker-email=<DOCKER_EMAIL>   --namespace devops
```

## 5. Kubernetes Resource Deployment

### 5.1 Apply Kubernetes Resources
Apply the necessary YAML files to create resources like storage classes, persistent volumes, PVCs, deployments, and ingress.
```bash
kubectl apply -f storage-class.yaml
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f simple-ingress.yaml
```

### 5.2 Get Kubernetes Resources
Check the status of different resources in the `devops` namespace:
```bash
kubectl get storageclass
kubectl get secrets -n devops
kubectl get all -n devops
kubectl get pv -n devops
kubectl get pvc -n devops
kubectl get ingress -n devops
```

## Conclusion
This guide provides the essential commands for managing Azure disks, configuring AKS, deploying an NGINX ingress controller, managing Kubernetes secrets, and applying resources such as PVs, PVCs, and deployments. It can be used to streamline your DevOps setup with Azure and Kubernetes.

