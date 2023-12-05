# Welcome to the workshop labs for "Dapr for building distributed .NET applications"

In these labs you will learn the basics of Dapr and how to use it to build distributed applications using .NET and C#. 

The lab uses an existing .NET based application for a ticket selling webapplication. The "GloboTicket" website is not using Dapr yet, but during the labs you will add Dapr support and start deploying the application to a Kubernetes cluster on your local development machine and also to Azure. At the end of the workshop and the labs you can have the GloboTicket application running as a cloud-native application leveraging both Dapr and cloud resources.

Here is the Dapr documentation for more detailed stuff: https://docs.dapr.io/getting-started/

The files for the labs for this workshop can be found at https://github.com/mariokerciku/daprworkshop. 
 
- Lab 0 - [Preparing your machine](Lab-00-Preparing-your-machine.md) (can be done before the workshop starts)
- Lab 1 - [Inspecting GloboTicket](Lab-01-Inspecting-GloboTicket.md)
- Lab 2 - [Adding Dapr sidecars](Lab-02-Adding-Dapr-sidecars.md)
- Lab 3 - [Using Service Invocation building block](Lab-03-Using-Service-Invocation-block.md)
- Lab 4 - [Using PubSub building block](Lab-04-Using-PubSub-building-block.md)
- Lab 5 - [Using binding components](Lab-05-Using-binding-components.md)
- Lab 6 - [Using Secret Store building block](Lab-06-Using-Secret-Store-building-block.md)
- Lab 7 - [Using State Store building block](Lab-07-Using-State-Store-building-block.md)
- Lab 8 - [Deploying to a local cluster](Lab-08-Deploying-to-a-local-cluster.md)
- Lab 9 - [Deploying to Azure Kubernetes Service](Lab-09-Deploying-to-Azure-Kubernetes-Service.md)
- Lab 10 - [Switching to Azure resources](Lab-10-Switching-to-Azure-resources.md)