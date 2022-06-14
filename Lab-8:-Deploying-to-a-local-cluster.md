In this lab you will deploy your Dapr based GloboTicket application to a Kubernetes cluster. So far you have run the application as a composition using Docker Compose. The next step is to take use your container images and deploy into a local Kubernetes cluster running in your development environment. This is an intermediate step to develop and test your solution local before going to an external cluster with less debugging support. 

## Enable Kubernetes in Docker Desktop
First, make sure you have your local cluster running. In GitHub Codespaces you should have executed `minikube start`. If you are using your own development machine, you should have enabled Kubernetes in Docker Desktop (or installed Minikube). 

Verify that your cluster is running and that you are connected to it with the Kubernetes CLI:

```cmd
kubectl config get-contexts
```