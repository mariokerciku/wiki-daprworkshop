In this lab you will switch the local and cluster hosted resources for the Dapr building blocks to Azure resources. In particular, these are the Azure resources we are going to use:
- Azure Storage account (State building block)
- Azure Service Bus (Pub/Sub building block
- Azure Key vault (Secret store building block)

The first two resources use the existing Kubernetes secret store building block. As the last step you will change this to Azure Key vault.

# Azure storage account for state component
The State building block currently uses Redis storage hosted within the Kubernetes cluster. You can change this to any other State provider that Dapr provides. Since we are using Azure, the preferred resource is blob storage from a container in an Azure Storage account.

Let's start by creating a storage account. Set some variables to make the Azure CLI commands a bit easier:

```cmd
# PowerShell
$RESOURCE_GROUP = "DaprWorkshop"
$STORAGE_ACCOUNT = "daprworkshopstorage"
$LOCATION = "westeurope"

# Bash
RESOURCE_GROUP=DaprWorkshop
STORAGE_ACCOUNT=daprworkshopstorage
LOCATION=westeurope
```

Execute this CLI command to create the account:
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

# Azure Service Bus for Pub/sub component
The second resource you will use is an Azure Service Bus namespace to replace Redis. Assuming you still have the variables set in your terminal session, we can continue by adding one more for the Azure Service Bus namespace:
```cmd
# PowerShell
$SERVICE_BUS = "daprworkshopservicebus"

# Bash
SERVICE_BUS=daprworkshopservicebus
```
This will allow you to easily create a namespace in the resource group and location specified earlier: 
```cmd
az servicebus namespace create -g $RESOURCE_GROUP -n $SERVICE_BUS -l $LOCATION --sku Standard
```

Next, extract the connection string to be able to allow the Pub/sub component access to the namespace and create topics and subscriptions for each publisher and subscriber.

```cmd
# PowerShell
$SERVICE_BUS_CONNECTION_STRING = az servicebus namespace authorization-rule keys list `
      -g $RESOURCE_GROUP --namespace-name $SERVICE_BUS `
      --name RootManageSharedAccessKey `
      --query primaryConnectionString `
      --output tsv

# Bash
SERVICE_BUS_CONNECTION_STRING=$(az servicebus namespace authorization-rule keys list -g $RESOURCE_GROUP --namespace-name $SERVICE_BUS --name RootManageSharedAccessKey --query primaryConnectionString --output tsv)
```

Similar to the step for the state component, you should create a secret to store the connection string into the secret store, apply the definition for the new pubsub component. You also have to restart the relevant pods `frontend` and `ordering` as these are the respective publisher and subscriber to the topic for the placed order event.
```cmd
kubectl create secret generic servicebus-secret --from-literal=connection-string="$SERVICE_BUS_CONNECTION_STRING"
kubectl apply -f .\azure-pubsub.yaml
kubectl rollout restart deployment frontend
kubectl rollout restart deployment ordering
```

Try buying a ticket again and verify that a topic was created and there was activity in the Azure Service Bus namespace. 