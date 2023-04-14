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

Look at the `azure-statestore.yaml` definition of the statestore component for Azure Blob Storage. It has an account name and container name and requires a `blob-secret` secret stored inside the secret store of Kubernetes.
Store the account key secret you extracted earlier into the Kubernetes secret store. For this we will define a new secret store component `secretstore` of type Kubernetes. The standard name of the default secret store in Kubernetes is actually `kubernetes`. By defining the same component with the name `secretstore` we can be more flexible in swapping the secret store implementations.
```cmd
kubectl apply -f ./kubernetes-secretstore.yaml
kubectl create secret generic blob-secret --from-literal=blob-secret="$STORAGE_ACCOUNT_KEY"
```
Next, apply the new state component configuration to the cluster and restart the three pods of the Globoticket application.
```cmd
kubectl apply -f ./azure-statestore.yaml
kubectl rollout restart deployment frontend
kubectl rollout restart deployment ordering
kubectl rollout restart deployment catalog
```

Verify that the application still works and visit the blob storage container in the Azure portal or any client tool you see fit. Check that there is basket state and concert event state stored in the container.

![image](https://user-images.githubusercontent.com/5504642/231604970-3784faf8-db8b-4809-b9b1-0c17259221b2.png)

# Azure Service Bus for Pub/sub component
The second resource you will use is an Azure Service Bus namespace to replace Redis. Assuming you still have the variables set in your terminal session, we can continue by adding one more for the Azure Service Bus namespace. You might have to choose your own unique name.
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

Next, extract the connection string to be able to allow the pubsub component access to the namespace and create topics and subscriptions for each publisher and subscriber.

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

Similar to the step for the state component, you should create a secret to store the connection string into the secret store, apply the definition for the new pubsub component. You also have to restart the relevant pods `frontend` and `ordering` as these are the respective publisher and subscriber to the topic `orders` for the placed order event.
```cmd
kubectl create secret generic servicebus-secret --from-literal=connection-string="$SERVICE_BUS_CONNECTION_STRING"
kubectl apply -f ./azure-pubsub.yaml
kubectl rollout restart deployment frontend
kubectl rollout restart deployment ordering
```

Try buying a ticket again and verify that a topic was created and there was activity in the Azure Service Bus namespace.

# Azure Key vault for secret store component
The final step in using Azure resources is the Azure Key vault as the secret store for sensitive configuration being used in the component definitions. We need to change the state and pubsub components to use this new secret store.
First, we need to choose a unique name for the Azure Key vault that will store our secrets.
```cmd
# PowerShell
$KEYVAULT = "DaprWorkshopKeyVault"

# Bash
KEYVAULT=DaprWorkshopKeyVault
```
Run the following Azure CLI command to create the vault:
```
az keyvault create --name $KEYVAULT --resource-group $RESOURCEGROUP --location $LOCATION --enable-rbac-authorization true
$KEYVAULT_ID=az keyvault show --name $KEYVAULT --query "id" -o tsv
```

The most difficult step is giving the Kubernetes cluster access to the vault. The most secure way is to use Managed Identities, by giving the AKS cluster a workload identity that accesses the vault. A managed identity does not have explicit credentials, so there is no need for secrets to get access to the Azure Key vault. 
Using a workload identity for the cluster involves an elaborate and somewhat complex setup of the cluster. Also, it is tied to Azure hosted Kubernetes clusters. Two reasons to use an Azure Active Directory app registration instead. An app registration allows us to create a specific account for our Globoticket application that is going to access the vault. 
```cmd
# PowerShell
$APP_NAME="globoticket-application"

# Bash
APP_NAME=globoticket-application
```
Create the app registration in your Azure Active Directory tenant and extract the ID of the application that was created:
```cmd
az ad app create --display-name $APP_NAME
```
The output contains the application ID of the newly created app registration:
```
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#applications/$entity",
  "addIns": [],
  "api": {
    "acceptMappedClaims": null,
    "knownClientApplications": [],
    "oauth2PermissionScopes": [],
    "preAuthorizedApplications": [],
    "requestedAccessTokenVersion": 2
  },
  "appId": "22119ad4-3599-4808-a884-5463446db6e1",
  "appRoles": [],
  "applicationTemplateId": null,
```
Store the `appId` value in a variable, as you will need it in a moment:
```cmd
# PowerShell
$APP_ID = "22119ad4-3599-4808-a884-5463446db6e1"

# Bash
APP_ID=22119ad4-3599-4808-a884-5463446db6e1
```
Reset the credential associated with the application ID:
```cmd
az ad app credential reset --id $APP_ID --years 2 
```
This will give the exact output we need to configure the Azure Key vault as the secret store component. The output should resemble this:
```
The output includes credentials that you must protect. Be sure that you do not include these credentials in your code or check the credentials into your source control. For more information, see https://aka.ms/azadsp-cli
{
  "appId": "34cfb7d3-353f-4cae-a451-e56a2224fe53",
  "password": "qpe8Q~hyaBfcRYgT7cmb_z5U2sYEkMRKXouuFdzQ",
  "tenant": "d123d456-1234-4567-b1cd-1aaf1e228995"
}
```

You need to create a secret again to hold the password of the app registrations client details.
```cmd
kubectl create secret generic keyvault-secret --from-literal=azureclientsecret="<password-of-app-registration>"
```

Of course, you also need to store the three secrets in the Azure Key vault to reference them in the three situations for configuration, state and pubsub component.
```cmd
az keyvault secret set --name catalogconnectionstring --vault-name $KEYVAULT --value "Event Catalog DB Connection string from Azure Key vault"
az keyvault secret set --name blob-secret --vault-name $KEYVAULT --value $STORAGE_ACCOUNT_KEY
az keyvault secret set --name servicebus-secret --vault-name $KEYVAULT --value $SERVICE_BUS_CONNECTION_STRING
```

Since we took some precautions to name the secret store component `secretstore` we do not have to change the yaml definitions of the three component definitions. 
Finally, you need to restart all three pods to read the configuration values from the new secret store, which is now Azure Key vault.
Visit the homepage of the Globoticket website again and check the logs of the `catalog` service. 

```cmd
kubectl get pods
kubectl logs <instance-name-of-catalog-pods>
```
Check  whether the logs indicate a connection string that has `Key vault` in it.