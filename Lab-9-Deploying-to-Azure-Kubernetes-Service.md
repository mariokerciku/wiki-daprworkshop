In this lab you will deploy the GloboTicket application to a Kubernetes cluster running in the cloud. The lab assumes you have an Azure subscription and are able to use Azure Kubernetes Service (AKS). You can get a free trial Azure account at https://azure.microsoft.com/en-us/free/.

# Setting up your Azure environment
You will be using PowerShell to run most of the CLI commands for Azure. Make sure you have installed the Azure CLI by running the `az` command from a terminal window. If you are using GitHub Codespaces this should already be installed. If you are doing this lab on your own machine, it might not be installed. If so, follow the instructions at https://docs.microsoft.com/en-us/cli/azure/install-azure-cli to install the right Azure CLI for your setup.

## Creating Azure resources
The Azure resources will be created inside a single resource group. Ideally, you will split the resources over multiple resource groups. In this lab there is a single group to contain all resources, which will make cleaning resources easier.

Set two variables for the name of the resource group and its location. You can choose an Azure region that is most convenient for you.

```PowerShell
$RESOURCEGROUP = "DaprWorkshop"
$LOCATION = "westeurope"
```

```cmd
RESOURCEGROUP=DaprWorkshop
LOCATION=westeurope
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

![image](https://user-images.githubusercontent.com/5504642/174282252-7c9eb514-54a2-4ad2-a716-3e354bf7c6b8.png)

If the new node is not already the current active cluster, you can richt-click on the new node in the `clusters` section and select "Set as Current Cluster". Alternatively, you can select the cluster as the current in your configuration by running:

```cmd
kubectl config use-context $CLUSTER_NAME
```

To make absolutely sure you have the new cluster selected, verify that the correct context is being used:

```cmd
kubectl config get-contexts
```

![image](https://user-images.githubusercontent.com/5504642/174282949-307db163-17ee-498f-81e8-57d7637734d7.png)

## Install Dapr runtime and control plane on AKS
From now on you will repeat a number of steps you have also done on the local cluster. Things will be familiar as the location of the cluster is reasonably transparent.

Verify that the Dapr CLI and runtime are installed.

```cmd
dapr --version
```

<img src="https://user-images.githubusercontent.com/5504642/226217423-fee2ddbd-4013-472c-9b43-a6da7f6b25e5.png" width="300" />

Run the initialization of Dapr on your AKS cluster. 
```cmd
dapr init -k --wait
```

![image](https://user-images.githubusercontent.com/5504642/174283359-1c164950-0fc3-49e4-9bee-f721ae6991e9.png)

Again, you can use the dashboard to see whether the Dapr control plane was installed and initialized successfully by running the dashboard.
```cmd
dapr dashboard -k -p 8180
```
This should open a browser and automatically navigate to http://localhost:8180/overview.

Also, you can check the status of Dapr from the command-line:
```cmd
dapr status -k
```
![image](https://user-images.githubusercontent.com/5504642/226217547-d5663218-93eb-4cc3-9850-ec04a7e8e8df.png)

## Installing dependencies 
With the cluster setup, it is time to install the dependencies for Dapr and our application. Go through similar steps as for your local cluster to install the dependencies to the AKS cluster. Below you can find the short instructions. You can refer to the previous lab for details. 

- Distributed tracing with Zipkin
```cmd
kubectl create deployment zipkin --image openzipkin/zipkin
kubectl expose deployment zipkin --type ClusterIP --port 9411
kubectl port-forward svc/zipkin 9413:9411
```
- SMTP mail server with MailDev
```cmd
kubectl create deployment maildev --image maildev/maildev 
```
- Secret for database
```cmd
kubectl create secret generic catalogconnectionstring --from-literal=catalogconnectionstring="Event Catalog Connection String from Kubernetes"
kubectl apply -f ./kubernetes-secretstore.yaml
```
- State store and pub/sub with Redis
```cmd
helm install daprworkshop-redis bitnami/redis
```
This time you will not need to get the password from the Redis installation. Instead we will use the Kubernetes secret that was automatically created and refer to that in the component definitions of `statestore` and `pubsub` later on.

# Deploy Dapr components
We need to register the Dapr components again. We will still use the 'default' namespace for convenience. In a production cluster you could also isolate the solution into a separate Kubernetes namespace. 

Open a terminal and navigate to the `lab-resources/azure` directory. You do not need to copy the files as you will only use these for Refer to the previous lab for details. The short steps are listed below:

```cmd
cd lab-resources/azure
kubectl apply -f ./cron.yaml
kubectl apply -f ./email.yaml
kubectl apply -f ./redis-pubsub.yaml
kubectl apply -f ./redis-statestore.yaml
kubectl apply -f ./appconfig.yaml
```

Look at Dapr dashboard again.

<img src="https://user-images.githubusercontent.com/5504642/226218987-4fb64280-acad-4c48-8fce-40bf169ed98d.png" width="700" />

# Application containers
The Dapr dependencies are now in place. Now you can deploy the pods for each of the three containers `ordering`, `frontend` and `catalog`.

This time we will need a container registry where the three images are pushed. This way the AKS cluster can reach the registry to pull the images. Having local images will not work. You can create your own private container registry in Azure. Pick a unique name (in lowercase) for your registry and set it in the first line of the following fragment:

```cmd
$REGISTRY_NAME = <your-container-registry> 
az acr create --resource-group $RESOURCEGROUP --name $REGISTRY_NAME --sku Basic
az acr login --name $REGISTRY_NAME
```
![image](https://user-images.githubusercontent.com/5504642/174288495-7956fe8d-9b1a-4da9-9a7b-6894583294cd.png)

You can check in the Azure portal whether the Azure Container Registry (ACR) was created successfully.

## Create container images
Since we now have a container registry, it is required to change the name of the container images to include the registry name. We can use an environment variable to help is out. Set the DOCKER_REGISTRY variable to be the `id` of your ACR instance.

```cmd
$LOGIN_SERVER=az acr show --resource-group $RESOURCEGROUP --name $REGISTRY_NAME --query loginServer -o tsv
$env:DOCKER_REGISTRY=$LOGIN_SERVER
```
or in Linux:
```
LOGIN_SERVER=$(az acr show --resource-group $RESOURCEGROUP --name $REGISTRY_NAME --query loginServer -o tsv)
export DOCKER_REGISTRY=$LOGIN_SERVER'/'
```
This environment variable is used when building the container images as specified in the respective `Dockerfile` files. Check the beginning of one of these files to see how it is being used.

Create the images again by building a `Release` version of the code 

```cmd
docker-compose build 
docker images
```
and verify in the output of the last command that the new images are available, with the name of the registry prepended, for example `daprworkshopcontainerregistry.azurecr.io/catalog:latest`.

## Pushing images to container registry
It is now easy to push the images to the registry. You are already logged in and the containers have the `latest` tag already. You can change the tag to be a specific version if you want to.
```cmd
docker push $LOGIN_SERVER/catalog:latest
docker push $LOGIN_SERVER/ordering:latest
docker push $LOGIN_SERVER/frontend:latest
```
Go to the container registry in your Azure portal and check in the repositories section if the images have been pushed successfully. You can also check using Visual Studio Code or GitHub Codespaces by using the Registries section of the Docker extension.

<img src="https://user-images.githubusercontent.com/5504642/226221038-aaa1d854-ffd8-431e-aa27-b07dc16e0f4d.png" width="400" />

## Deploying pods to AKS
The AKS cluster needs to be able to pull images from the private container registry. You can give the AKS cluster access to the container registry by executing this command in PowerShell or Linux:

```PowerShell
$REGISTRY_ID = az acr show --resource-group $RESOURCEGROUP --name $REGISTRY_NAME --query id -o tsv
az aks update --name $CLUSTER_NAME--resource-group $RESOURCEGROUP --attach-acr $REGISTRY_ID
```
```cmd
REGISTRY_ID=$(az acr show --resource-group $RESOURCEGROUP --name $REGISTRY_NAME --query id -o tsv)
az aks update --name $CLUSTER_NAME --resource-group $RESOURCEGROUP --attach-acr $REGISTRY_ID
```

This allows us to deploy the pods for the catalog, ordering and frontend components of the GloboTicket application.
```cmd
kubectl apply -f ./catalog.yaml
kubectl apply -f ./ordering.yaml
kubectl apply -f ./frontend.yaml
```
Follow the deployment of the pods and services that are defined in the deployment manifests. After all is running correctly, find the public IP address for the cluster:

```cmd
kubectl get svc frontend -o jsonpath="{.status.loadBalancer.ingress[*].ip}"
```

Visit the website URL at the IP address you found and using port 8080, for example `http://<your-cluster-ipaddress>:8080/`.
If all went well, you should be treated with a working GloboTicket website. In case of errors, examine the logs of the pods and its containers to find out what went wrong and correct it.