In this lab you will switch the local and cluster hosted resources for the Dapr building blocks to Azure resources. In particular, these are the Azure resources we are going to use:
- Azure Storage account (State building block)
- Azure Service Bus (Pub/Sub building block
- Azure Key vault (Secret store building block)

The first two resources use the existing Kubernetes secret store building block. As the last step you will change this to Azure Key vault.

# Azure storage account for State building block
The State building block currently uses Redis storage hosted within the Kubernetes cluster. You can change this to any other State provider that Dapr provides. Since we are using Azure, the preferred resource is blob storage from a container in an Azure Storage account.

Let's start by creating a storage account. Set some variables to make the Azure CLI commands a bit easier:

```cmd
# PowerShell
$RESOURCE_GROUP="DaprWorkshop"
$STORAGE_ACCOUNT="daprworkshopstorage"
$LOCATION="westeurope"
# bash
RESOURCE_GROUP=DaprWorkshop
STORAGE_ACCOUNT=daprworkshopstorage
LOCATION=westeurope
```

Execute the CLI command to create the account
```cmd
az storage account create -n $STORAGE_ACCOUNT -g $RESOURCE_GROUP -l $LOCATION --sku Standard_LRS
```

After the storage account has been created, we need to extract the account key and connection string to configure the statestore component.
```cmd
# PowerShell
$STORAGE_ACCOUNT_KEY = az storage account keys list -g $RESOURCE_GROUP -n $STORAGE_ACCOUNT --query [0].value -o tsv
$STORAGE_CONNECTION_STRING = az storage account show-connection-string -n $STORAGE_ACCOUNT -g $RESOURCE_GROUP --query connectionString -o tsv

# bash
STORAGE_ACCOUNT_KEY=$(az storage account keys list -g $RESOURCE_GROUP -n $STORAGE_ACCOUNT --query [0].value -o tsv)
STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n $STORAGE_ACCOUNT -g $RESOURCE_GROUP --query connectionString -o tsv)
```

Next, we create a storage container in the storage account to store the state of Dapr.
```cmd
az storage container create --name "statestore" --public-access off --connection-string $STORAGE_CONNECTION_STRING
```

Look at the azure-statestore.yaml definition of the statestore component for Azure Blob Storage. It has an account name and container name and requires a 'blob-secret' secret stored inside the secret store of Kubernetes.
Store the account key secret you extracted earlier into the Kubernetes secret store and apply the new state component configuration to the cluster and restart the three pods of the Globoticket application.
```cmd
kubectl create secret generic blob-secret --from-literal=account-key="$STORAGE_ACCOUNT_KEY"
kubectl apply -f ./azure-statestore.yaml
kubectl rollout restart deployment frontend
kubectl rollout restart deployment ordering
kubectl rollout restart deployment catalog
```

Verify that the application still works and visit the blob storage container in the Azure portal or any client tool you see fit. Check that there is basket state and concert event state stored in the container.

![image](https://user-images.githubusercontent.com/5504642/231604970-3784faf8-db8b-4809-b9b1-0c17259221b2.png)
