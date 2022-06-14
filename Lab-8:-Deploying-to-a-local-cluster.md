In this lab you will deploy your Dapr based GloboTicket application to a Kubernetes cluster. So far you have run the application as a composition using Docker Compose. The next step is to take use your container images and deploy into a local Kubernetes cluster running in your development environment. This is an intermediate step to develop and test your solution local before going to an external cluster with less debugging support. 

## Enable Kubernetes in Docker Desktop
First, make sure you have your local cluster running. In GitHub Codespaces you should have executed `minikube start`. If you are using your own development machine, you should have enabled Kubernetes in Docker Desktop (or installed Minikube). 

Verify that your cluster is running and that you are connected to it with the Kubernetes CLI:

```cmd
kubectl config get-contexts
```

The output various per cluster type.

![image](https://user-images.githubusercontent.com/5504642/173692060-36dab09d-8ef9-4279-a6b1-125a5c4afc9e.png)
![image](https://user-images.githubusercontent.com/5504642/173692846-0da66222-e740-42ed-aae8-99e40340c8ba.png)

Also, you can check using the Visual Studio Code extension for Kubernetes to see if you are connected correctly.

![image](https://user-images.githubusercontent.com/5504642/173692328-1abcd1c8-dffe-481a-a2f5-ddf19911d617.png)
![image](https://user-images.githubusercontent.com/5504642/173692437-7894eead-386a-4838-acfc-d7329253fcf0.png)

## Preparing cluster to use Dapr
Prepare your cluster by installing the Dapr runtime and control plane. Verify that Dapr CLI and runtime are installed correctly first.
```cmd
dapr --version
```
![image](https://user-images.githubusercontent.com/5504642/173692831-719ed42f-a460-4d51-b1f9-bb90176704e9.png)

Run initialization for Dapr on local Kubernetes cluster
dapr init -k
 