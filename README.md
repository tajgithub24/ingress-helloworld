
# Azure & Kubernetes Setup for DevOps

This README outlines the necessary steps to create and manage resources on Azure, set up Kubernetes (AKS), and deploy applications using Helm, Kubernetes secrets, and other resources.

## 1. Azure Kubernetes Service (AKS)

### 1.1 Install Azure Cli
Follow the steps outlined in the official documentation to install the Azure CLI:
```bash
https://learn.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest
```

### 1.2 Install kubectl
Follow the steps outlined in the official documentation to install kubectl:
```bash
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
```

### 1.3 Create an AKS Cluster
You can create an AKS cluster through the Azure Portal for a simple setup. This graphical interface makes the process easy to follow.

Steps:
1.	Go to the Azure Portal.
2.	Navigate to "Kubernetes Services".
3.	Click on "Add" to create a new AKS cluster.
4.	Fill in the required details such as subscription, resource group, region, and cluster configuration.
5.	Review the configuration and click "Create".
For more advanced users, the Azure CLI allows for creating an AKS cluster with customizable configurations.


### 1.4 Get AKS Cluster Details

Retrieve complete details of your AKS cluster:
```bash
az aks show --resource-group <resource-group> --name <cluster-name>
```
Example: 
az aks show --resource-group devops-rg --name k8-cluster-1


Retrieve the location of your AKS cluster:
```bash
az aks show --resource-group <resource-group> --name <cluster-name> --query location
```
Example: 
az aks show --resource-group devops-rg --name k8-cluster-1 --query location


Retrieve the user identities of your AKS cluster:
```bash
az aks show --resource-group <resource-group> --name <cluster-name> --query "identity" -o json
```
Example: 
az aks show --resource-group devops-rg --name k8-cluster-1 --query "identity" -o json


## 2. Azure Disk Management

### 2.1 Create a Disk
This command creates a new disk in Azure with a 10 GB size and the Standard_LRS SKU.
```bash
az disk create --resource-group <resource-group> --name <disk-name> --size-gb 10 --sku Standard_LRS  --location <location>
```
Example:
az disk create --resource-group devops-rg --name aks-pv-disk --size-gb 10 --sku Standard_LRS  --location centralindia

###Note: Ensure the disk is created in the same location as the AKS cluster.

### 2.2 Show Disk Information (Basic)
Display basic information about the specified disk.
```bash
az disk show --resource-group <resource-group> --name <disk-name>
```
Example: 
az disk show --resource-group devops-rg --name aks-pv-disk


### 2.3 Show Disk ID or URI to be used in Persistent Volume
This command retrieves the disk ID, which can be used in the pv.yaml file for persistent volumes.

```bash
az disk show --resource-group <resource-group> --name <disk-name> --query [id] -o tsv
```
Example: 
az disk show --resource-group devops-rg --name aks-pv-disk --query [id] -o tsv


### 2.4 Assign Role  
You can assign the "Contributor" role to the disk resource, either through the Azure Portal (Access control (IAM) of aks cluster ) or by using the following Azure CLI command:
```bash
az role assignment create --assignee <cluster_principal_id> --role Contributor --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Compute/disks/<disk-name>
```

###Note: The <cluster-principal-id> can be retrieved from the output of the az aks show command (Step 1.4).

## 3. Helm & NGINX Ingress

### 3.1 Install Helm
Follow the instructions to install Helm, which is used to manage Kubernetes applications:
```bash
https://helm.sh/docs/intro/install/
```

### 3.2 Add Helm Repository and Update
Add the NGINX Ingress repository and update the Helm chart index.
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### 3.3 Show Helm Values
Display the configurable values for the NGINX Ingress chart.
```bash
helm show values ingress-nginx/ingress-nginx
```

### 3.4 Install NGINX Ingress Controller
Install the NGINX Ingress controller with specific configuration settings such as replica count, node selector, and load balancer settings.
```bash
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.replicaCount=2 \
  --set controller.nodeSelector."kubernetes\.io/os"=linux \
  --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux \
  --set controller.service.externalTrafficPolicy=Local
```

## 4. Kubernetes Namespace & Secrets

### 4.1 Create a Namespace for deployment
Create a new namespace 'devops' where your resources will reside.
```bash
kubectl create ns devops
```

### 4.2 Create Docker Registry Secret
Create a Kubernetes secret to access your Docker registry (DockerHub in this case) using your credentials.
```bash
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<DOCKER_USERNAME> \
  --docker-password=<DOCKER_PASSWORD> \
  --docker-email=<DOCKER_EMAIL> \
  --namespace devops
```

## 5. Kubernetes Resource Deployment

### 5.1 Apply Kubernetes Resources
Use kubectl to apply the necessary YAML files to create resources such as storage classes, persistent volumes (PVs), persistent volume claims (PVCs), deployments, and ingress controllers.
```bash
kubectl apply -f storage-class.yaml
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f ingress.yaml
```

### 5.2 Get Kubernetes Resources
To check the status of different resources in the devops namespace, use the following commands:
```bash
kubectl get storageclass
kubectl get secrets -n devops
kubectl get all -n devops
kubectl get pv -n devops
kubectl get pvc -n devops
kubectl get ingress -n devops
```

## Conclusion
This guide provides the essential steps and commands for managing Azure disks, configuring AKS, deploying the NGINX Ingress controller, managing Kubernetes secrets, and applying resources like Persistent Volumes (PVs), Persistent Volume Claims (PVCs), and deployments. 
