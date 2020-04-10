# Authenticate with Azure Container Registry from Azure Kubernetes Service

1. Make sure that your have enough permission to add role assignment.
2. Execute the following command or save it in shell script:
```Shell
#!/bin/bash

AKS_RESOURCE_GROUP=Ubidy.IT.Kubernetes.AustraliaEast.UAT
AKS_CLUSTER_NAME=ubidy-kube-uat
ACR_RESOURCE_GROUP=Ubidy.IT.Kubernetes.AustraliaEast.Production
ACR_NAME=ubidy

# Get the id of the service principal configured for AKS
CLIENT_ID=$(az aks show --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv)

# Get the ACR registry resource id
ACR_ID=$(az acr show --name $ACR_NAME --resource-group $ACR_RESOURCE_GROUP --query "id" --output tsv)

# Create role assignment
az role assignment create --assignee $CLIENT_ID --role acrpull --scope $ACR_ID

```
