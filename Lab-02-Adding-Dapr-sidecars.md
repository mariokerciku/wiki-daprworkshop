In this lab you will introduce Dapr into the GloboTicket solution. You are going to learn how to add Dapr sidecars to an existing Docker composition. Also, you will see a number of ways to run your application using Dapr support and experiment with the new setup.
Make sure that you have installed the Dapr CLI and initialized the runtime. You can validate the installation by running 
```
dapr --version
```
Go back to [Preparing your machine](Lab-0-Preparing-your-machine) if the CLI or runtime was not installed or initialized.

## Adding Dapr support
Make sure you have familiarized yourself with the contents of the Docker Compose files docker-compose.yml and docker-compose.override.yml. 

The first step is to add Dapr sidecars as companions of the three containers ordering, catalog and frontend. The Dapr sidecars are additional containers that also run as part of the container composition using Docker Compose.

Add a sidecar for the catalog service in the same `docker-compose.override.yml` file. Even though the place is not relevant inside a Docker Compose yaml file, it might be best to place this fragment directly below the definition of the catalog service.

```yaml
  catalog-dapr:
    container_name: "catalog-sidecar"
    image: "daprio/daprd:1.12.0"
    command: [
      "./daprd",
     "-app-id", "catalog",
     "-app-port", "80",
     "-components-path", "/components",
     "-config", "/components/config.yaml"
     ]
    volumes:
      - "./components/docker-compose/:/components"
    depends_on:
      - catalog
    network_mode: "service:catalog"
```

Notice how the image for this sidecar container is `daprio/daprd` and has a tag of 1.12.0 for the Dapr version. This version should match the version of Dapr you have installed. Also, the name of the sidecar `catalog-dapr` reflects the fact that this sidecar belongs to the catalog service. The name of `catalog` appears in multiple places, such as app-id, the unique name of the Dapr application this sidecar belongs to. Dapr considers other containers running as applications.

Another important thing to note is that the network mode of the sidecar is defined as `service:catalog` indicating that the networks of the catalog and `catalog-dapr` containers are tightly connected. The net effect is that the two containers behave almost as if they are running in a pod like Kubernetes does. We will cover pods in more detail in a later lab.

**Repeat creating the sidecar definitions for both the frontend and ordering service, making sure that the 5 occurences of the service name (`catalog` in the fragment shown) are replaced to be 'frontend' and 'ordering' respectively.**

## Additional dependencies for Dapr
Since Dapr requires a number of extra dependencies for a state store, pub sub and telemetry, simple versions such as Redis Cache and Zipkin are used for local development. 
You will add these to the docker-compose.override file as well, below the sidecar definitions.

```yaml
  redis:
    container_name: "redis"
    image: "redis:7.2-alpine"
    ports:
      - "6379"
    networks:
      - globoticket
      
  zipkin:
    container_name: "zipkin"
    image: "openzipkin/zipkin:2.24"
    ports:
      - 9412:9411
    networks:
      - globoticket
```
> Your `docker-compose.override.yml` file should now have a total of eight services.

## Adding Dapr configuration and components
Dapr needs a configuration to work correctly, since it is configured in the Dapr sidecar. It also defines a folder for the components you are going to add later.
You can find the file `config.yaml` in the folder lab-resources located in the root of the cloned repository.

Create a new directory in the root of your solution and name it `components`. Inside it create another folder `docker-compose`. In later labs we will create more directories inside the `components`, for local and cloud clusters.

The `docker-compose` will contain the configuration for now. Copy the `config.yaml` file from the lab-resources to the folder. This should be enough for now.

## Start composition including Dapr
Now it is time to start your composition for the GloboTicket application again. Use the same method as in the previous lab to start it.
 
In Visual Studio 2022 you can check the running containers by inspecting the Containers window. It should show the 3 services running, with a sidecar for each, plus the Redis and Zipkin containers.

In Visual Studio Code or your GitHub Codespace you can use the Docker extension to inspect running containers.

<img src="https://github.com/XpiritCommunityEvents/DaprWorkshop/assets/5504642/b7b8f7ec-9fda-4270-8f5b-48b291df8efa" width="500" />

Of course you can always use the Docker or Docker Compose CLI to ask for running containers.

```cmd
docker-compose ps
```

![image](https://user-images.githubusercontent.com/5504642/173665336-bbb292cd-2f63-46c5-9b4d-70e27ece778e.png)

### Visual Studio 2022 tip
For easy access to the Dapr component definitions, you can add a solution folder named Dapr and inside another folder named `docker-compose`. In the folder add existing items from the `components\docker-compose` folder you created in the root of the Git repository.

In later labs you will copy component yaml files into the components folder. Repeat adding the newly copied files whenever asked to do so. Eventually, the result should look like this:

![image](https://user-images.githubusercontent.com/5504642/173665452-f56ffbb0-470f-4092-8872-360f28bd3a6d.png)

## Running your application with Dapr sidecars
Try your new container composition and see if everything works as expected. You can check this URL to see if Zipkin is working correctly: http://localhost:9412.
If you are running in Codespaces go to your `Ports` tab and right-click the container running on port 9412 and select `Open in browser`).

When you click on the `Run Query` button no results appear. This is because we do not use the Dapr sidecars yet, so nothing has really changed. The focus of the next lab is to use the sidecars for service invocation.

## Finish lab
You are all done. You have added Dapr sidecar to your application services. In the next lab you will start using Dapr by adding code to the application.

Stop running your application. In Visual Studio Code and GitHub CodeSpaces you can stop the composition by pressing Ctrl+C in the terminal window. 

<img src="https://user-images.githubusercontent.com/5504642/173663285-5882128d-08a0-48cc-989a-804047beff89.png" width="400" />

In Visual Studio 2022 you can press Shift+F5 or click on the red square stop icon in the Debug toolbar.