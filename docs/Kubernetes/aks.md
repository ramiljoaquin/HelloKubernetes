# Connect to the Azure Kubernetes Service (AKS) cluster

1. Make sure that the azure cli installed in your machine.
2. If kubectl is not installed in your machine, run the command 
```bash
az aks install-cli
```
3. Login using your Azure Active Directory
```bash
az login
```
3. To connect to your AKS cluster, run the command 
```bash
az aks get-credentials --resource-group [your resource group] --name [your aks cluster]
```
