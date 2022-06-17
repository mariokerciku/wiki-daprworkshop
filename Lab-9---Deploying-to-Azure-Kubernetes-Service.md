In this lab you will deploy the GloboTicket application to a Kubernetes cluster running in the cloud. The lab assumes you have an Azure subscription and are able to use Azure Kubernetes Service (AKS). You can get a free trial Azure account at https://azure.microsoft.com/en-us/free/.

## Preparations in Azure
You will be using PowerShell to run most of the CLI commands for Azure. Make sure you have installed the Azure CLI by running the `az` command from a terminal window. If you are using GitHub Codespaces this should already be installed. If you are doing this lab on your own machine, it might not be installed. If so, follow the instructions at https://docs.microsoft.com/en-us/cli/azure/install-azure-cli to install the right Azure CLI for your setup.

The Azure resources will be created inside a single resource group. Ideally, you will split the resources over multiple resource groups. In this lab there is a single group to contain all resources, which will make cleaning resources easier.

Set two variables for the name of the resource group and its location.

```cmd
$RESOURCEGROUP = "DaprWorkshop"
$LOCATION = "westeurope"
```
Login to your Azure account and check whether you have selected the right subscription:

```cmd
az login
az account list
```

Locate the subscription you want to use. If it does not have `"isDefault": true` listed you need to change the current subscription, by running the following command. Replace the placeholder with your subscription ID:

```cmd
az account set --subscription <your-subscription-id>
```

Create a resource group in your subscription

```
az group create -n $RESOURCEGROUP -l $LOCATION
```

We are going to use the preview features of `OIDC issuer` and `Pod Identity` in Azure. You will have to register these features in your Azure subscription.

```cmd
az feature register --name EnableOIDCIssuerPreview --namespace Microsoft.ContainerService
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
```

The registration might take some time. You can check the status of the registration by running:

```cmd
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/Enable')].{Name:name,State:properties.state}"
```

When the state for both EnableOIDCIssuerPreview and EnablePodIdentityPreview is `registered` you can continue with the next step.

Include the extension for preview features of AKS in the Azure CLI:

```cmd
az extension add --name aks-preview
```

## Install AKS cluster
You are now ready to install a real Azure based Kubernetes cluster. Execute the followin command to create an AKS cluster with three features enabled:
- Managed Identity - Makes the cluster nodes run as a specific managed service principal identity 
- OpenIDConnect Issuer - Adds a OIDC identity provider to issue identity tokens from within the cluster
- Pod Identity - Allows pods to run with a specific managed service principal identity 

```cmd
$CLUSTER_NAME = "DaprWorkshopCluster"
az aks create --resource-group $RESOURCEGROUP --location $LOCATION --name $CLUSTER_NAME --node-count 1 --enable-addons http_application_routing --generate-ssh-keys --enable-managed-identity --enable-oidc-issuer --enable-pod-identity --network-plugin azure 
```

You can read some additional information on these features here:
- https://docs.microsoft.com/en-us/azure/aks/use-azure-ad-pod-identity 
- https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#oidc-issuer-preview 
- https://docs.microsoft.com/en-us/azure/aks/http-application-routing

At this point your cluster is going to be created. This will take some time and you might want to take a little break from the labs at this point. 

After the creation has completed successfully, you should have a running cluster. The next step is to switch the `kubectl` context to use the remote AKS cluster instead of the local cluster. 

```cmd
az aks get-credentials -n $CLUSTER_NAME -g $RESOURCEGROUP -a
```

This command requires administrator priviliges (which you have to your own cluster) to get the credentials needed to communicate with the cluster. These will be stored in the `config` file in your `~/.kube` or `%USERPROFILE%\.kube` folder. 

Examine the Kubernetes tab in Visual Studio Code or GitHub Codespaces. 
Richt-click new node on clusters and select "Set as Current Cluster"
