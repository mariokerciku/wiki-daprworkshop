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
```cmd
dapr init -k
```
![image](https://user-images.githubusercontent.com/5504642/174170323-a37dcfb1-cab6-4f2f-ad28-cf6bbfbe56bf.png)

Verify a succesful installation by starting dashboard in a separate terminal window:
```cmd
dapr dashboard -k
```
![image](https://user-images.githubusercontent.com/5504642/174170417-93db9e64-8b6d-404f-822c-7a00acc4ad04.png)

Notice that no Dapr applications are running yet. This is expected, as we did not deploy anything to the cluster yet.
Check the Dapr components, configuration and control plane tabs as well.

## Kubernetes dashboard on Kubernetes 1.24+
Kubernetes also offers a dashboard that you can use to inspect the cluster. Install the dashboard by applying the manifest for it:
```cmd
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
```

You can reach the dashboard by running a proxy:
```cmd
kubectl proxy
```
The dashboard can be found at this endpoint: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/. You will reach a login screen where you need to provide a token (or kubeconfig file).

![image](https://user-images.githubusercontent.com/5504642/174170716-ff48aef1-6b9d-48f4-84f2-c4d5882d092b.png)

You can use a service account for login into the dashboard. First, you will need to create a new service account and give it the proper role-binding to access the dashboard. Use the two files `lab-resources/dashboard-user.yaml` and `lab-resources/dashboard-rolebinding.yaml` from the resources folder and apply these to the cluster. 

```cmd
cd lab-resources
kubectl apply -f .\dashboard-user.yaml
kubectl apply -f .\dashboard-rolebinding.yaml
cd ..
```

From Kubernetes 1.24 onward tokens are no longer created automatically for service accounts. You can now create a token by running this command:
```cmd
kubectl create token admin-user -n kubernetes-dashboard
```

The command gives output similar to:

```
eyJhbGciOiJSUzI1NiIsImtpZCI6InI0U29nakdwNmR1T3RLVzBRTVBUZUxGSWhtYl9ZRExsNXpBYmNvdjl0MWcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLXY5cnhjIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwOGExMDBiZC1kZjlhLTRmM2YtYjExYS0xMDZmZTkyZDI4NzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZGVmYXVsdCJ9.lNaY8rnoZv0BTNoI-F7fj-CtxtWF_fulymFL1k2y0BpgvPRfojsKy7HBzBi9qnUwipLK46AksCOzgg0Z3DbpF9BN_4VIBQmfJ4_yH1v8TYqC7LSriyIYEST_hJIRCQbJ919CXxSxW-Teo8mJ3mZo9PheBiARLas3P-e2e_xu14_Q5DnvjCcmgxTpPBEBhi5G-4O7fDybVljZxgeBQM65ODCd5pTjPp_SPrFykw2qWCHnEl28q5wUvtGYlXle9aAN1arZq1O2_h98LAYUUryYzGNEp4Ma7CIdytf1nwwpSaAmRUAC4
```
Copy and paste the token value into the login field that reads Enter token and click `Sign in`. You should see the dashboard appear.

If all goes well, you will be presented with the dashboard:
![image](https://user-images.githubusercontent.com/5504642/174171219-a02c68d7-204f-4a4e-a4a1-28fb79fb6650.png)

# Installing Dapr dependencies 
Since the application uses a set of Dapr components that require additional services to be available, you need to add these to the cluster for local development. These services are:
- Zipkin
- SMTP mail server
- Redis Cache (for state store and pubsub)

These containers correspond to the same containers that were running as part of the container composition in Docker Compose earlier.

## Distributed tracing with Zipkin
Dapr needs a running instance of Zipkin in your cluster. This will collect all distributed tracing telemetry from Dapr sidecars. Install a pod with Zipkin in it by running the following commands:

```cmd
kubectl create deployment zipkin --image openzipkin/zipkin
kubectl expose deployment zipkin --type ClusterIP --port 9411
kubectl port-forward svc/zipkin 9413:9411
```

The Zipkin deployment will create a pod with a running Zipkin container, listening on port 9411. The `kubectl port-forward` command is forwarding calls on your local machine at localhost:9413 to the service and consequently the pod with Zipkin. This way you can inspect the trace information being collected through the Zipkin user interface.

## SMTP mail server with MailDev
Also, you need a running container to simulate a SMTP mail server used in the smtp binding again. 
Run the following commands:

```cmd
kubectl create deployment maildev --image maildev/maildev 
```
You can see how the SMTP service is being hosted inside the cluster by finding the right pod name and viewing the logs:
```cmd
kubectl get pods # Find the ID of the maildev pod
kubectl logs <podid_maildev>
```
![image](https://user-images.githubusercontent.com/5504642/174284468-2aa93b55-71b9-4c2a-b9c8-d218969cc198.png)

As you can see, it uses two ports for SMTP and HTTP traffic. You must expose these and port-forward the HTTP port to view the maildev UI.
```cmd
kubectl expose deployment maildev --type ClusterIP --port 1025,1080
kubectl port-forward svc/maildev 8088:1080
```

## State store and pub/sub with Redis
Installing Redis in a Kubernetes cluster is easiest with Helm.

Refer to [Lab 0](https://github.com/XpiritCommunityEvents/DaprWorkshop/wiki/Lab-0:-Preparing-your-machine) for installation of Helm with Chocolatey
```cmd
choco install kubernetes-helm
```

```cmd
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install daprworkshop-redis bitnami/redis
```

Notice how the Helm release of Redis is called daprworkshop-redis:
![image](https://user-images.githubusercontent.com/5504642/174171912-132da2de-3223-4658-aa97-9684e406b114.png)

It uses the default port of 6379. Should you need the password that was created for the Redis service, you can find it using the instructions in the output. On a Linux bash run the command:

```cmd
export REDIS_PASSWORD=$(kubectl get secret --namespace default daprworkshop-redis -o jsonpath="{.data.redis-password}" | base64 --decode)
```

In PowerShell you need to do a bit more work to decode the base64 encoded password:

```powershell
$REDIS_PASSWORD = [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String(kubectl get secret --namespace default daprworkshop-redis -o jsonpath="{.data.redis-password})
```

Verify that the Redis password was set correctly by calling `printenv REDIS_PASSWORD` for Linux or `$env:REDIS_PASSWORD` for PowerShell.
In our case we will not really need the password, as it is added as a secret to the cluster. Initially, you will use the password, but later replace this with a reference to the secret. This latter approach avoids having a plain-text password somewhere in your configuration.

# Dapr components
At this point you are ready to register the Dapr components. For now we will use the 'default' namespace for convenience.

## Binding to CRON schedule
The `lab-resources` directory contains a subdirectory called `kubernetes`. Copy the directory into the `components` folder located at the root of your Git repository. Next, apply the Kubernetes manifest files.

```cmd
cd components/kubernetes
kubectl apply -f .\cron.yaml
```

## Binding to SMTP 
Repeat the same for the SMTP binding component. 
```cmd
kubectl apply -f .\email.yaml
```

## Statestore and pubsub
First, change passwords in `components/kubernetes/redis-statestore.yaml` and `components/kubernetes/redis-pubsub.yaml` to Redis password from the last step in the Redis installation.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
spec:
  type: pubsub.redis
  version: v1
  metadata:
  - name: redisHost
    value: daprworkshop-redis-master:6379
  - name: redisPassword
    value: N8IVeZRQki # Change this password
```

The redisHost is `daprworkshop-redis-master:6379` referring to the name found earlier.
Deploy the two components for `pubsub` and `statestore`:

```cmd
kubectl apply -f .\redis-pubsub.yaml
kubectl apply -f .\redis-statestore.yaml
```

Finally, deploy the Dapr configuration for this application: 
```cmd
kubectl apply -f .\appconfig.yaml
```

This would be a good time to look at the Dapr dashboard again.

![image](https://user-images.githubusercontent.com/5504642/174189748-f759559d-9232-4acb-be2a-d054e3ca1a3c.png)

## Secret store for database
Kubernetes has its own secret store that you can use to store the secrets in your composition. Create a new secret with a different value for the fictitious connection string, so you can see the effect later on.

```cmd
kubectl create secret generic catalogconnectionstring --from-literal=catalogconnectionstring="Event Catalog Connection String from Kubernetes"
kubectl apply -f .\kubernetes-secretstore.yaml
```

# Application containers
You are all set up to start deploying the pods for each of the 3 containers `ordering`, `frontend` and `catalog`.
Build a `Release` version of the code and create the Docker images. 

```cmd
dotnet build -c Release globoticket-dapr.sln
```
An alternative would be to use Docker Compose to perform the build, based on the `docker-compose.yml` and `docker-compose.override.yml` files:
```cmd
docker-compose build 
```
Remember that you can combine building and starting (upping) a composition using `docker-compose up --build`.
This creates 

## Deploying pods 
The final step to get to a complete solution is to tag the container images that were created with a registry name at the front. To indicate that we will get the images from the local image store, we will name the registry `local`.
Examine the contents of `catalog.yaml`:

```
  template:
    metadata:
      labels:
        app: catalog
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "catalog"
        dapr.io/app-port: "80"
        dapr.io/config: "appconfig"
    spec:
      containers:
      - name: catalog
        image: local/globoticket-dapr-catalog:latest
        ports:
        - containerPort: 80
        imagePullPolicy: Never
```

Notice how the definition of the deployment refers to `local/globoticket-dap-catalog:latest` and never pulls an image from a registry, as they should already be available locally because of the build. 

```cmd
docker tag catalog local/globoticket-dapr-catalog:latest
kubectl apply -f .\catalog.yaml

docker tag ordering local/globoticket-dapr-ordering:latest
kubectl apply -f .\ordering.yaml

docker tag ordering local/globoticket-dapr-frontend:latest
kubectl apply -f .\frontend.yaml
```

Check your dashboard and verify that everything runs correctly. Visit the GloboTicket website at http://localhost:8080 and try to order tickets.