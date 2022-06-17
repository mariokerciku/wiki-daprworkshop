In this lab you will deploy the GloboTicket application to a Kubernetes cluster running in the cloud. The lab assumes you have an Azure subscription and are able to use Azure Kubernetes Service. You can get a free trial Azure account at https://azure.microsoft.com/en-us/free/.

## Preparations in Azure
You will be using PowerShell to run most of the CLI commands for Azure. Make sure you have installed the Azure CLI. Refer to "Preparing your development machine" for installation instructions.

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
