In this lab you are going to prepare your machine for this workshop. This assumes you want to use your own development machine to work on the labs.

Alternatively you can run all labs from a browser using GitHub Codespaces. This requires you to have an active Team plan that allows you to use CodeSpaces. It incurs cost for compute and storage. If you plan to use a codespace, you can skip to [Running from GitHub Codespaces](Lab-0-Preparing-your-machine#running-from-github-codespaces)

# Running from your local development machine
You can run all labs from your own development machine. In order to do so you need to install quite a bit of tooling. Some of it might already be installed if you are using your regular development environment. Check each of the following items and skip the ones that are already installed.

## Install Windows Terminal 
Open Microsoft Store and search for Windows Terminal or follow this link:
https://www.microsoft.com/store/productId/9N0DX20HK701

## Windows Subsystem for Linux
On Windows 10/11, you need to enable the Windows Subsystem for Linux (WSL), by following the steps described here: https://docs.microsoft.com/en-us/windows/wsl/install

We recommend you add the Ubuntu and Debian distro from the Microsoft Store:
- https://apps.microsoft.com/store/detail/ubuntu-on-windows/9NBLGGH4MSV6
- https://apps.microsoft.com/store/detail/debian/9MSVKQC78PK6

If you cannot do this, you will use a Linux Virtual Machine to run Linux containers on Windows later.

## Visual Studio as an IDE
The most preferable Integrated Development Environment (IDE) is Visual Studio 2022 if you are using Windows. For MacOS you can use Visual Studio for Mac. You can find all editions, including a free Community edition here: https://visualstudio.microsoft.com/downloads/

<img src="https://user-images.githubusercontent.com/5504642/226197270-b6fadca7-fd83-4318-a5ee-563232b5140f.png" width="600" />

Visual Studio Code is available for all three platforms Windows, Linux and MacOS. On Linux Visual Studio Code is recommended, as there is no Visual Studio for Linux available. 

<img src="https://user-images.githubusercontent.com/5504642/226197221-a8efcf83-00ae-430c-9600-5f97647d668e.png" width="600" />

## Add Visual Studio Code extensions
If you have installed Visual Studio Code, you can add extensions. A couple of Visual Studio Code extensions are required to assist you during the labs. You can install them before the workshop or at the start of labs that require them.

| Extension | URL |
| :----------- | :----------- |
| Remote WSL | https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl |
| Docker | https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker |
| Kubernetes | https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools |
| Dapr | https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-dapr |

## Install Docker Desktop 
Docker Desktop is a popular choice to interact with containers on a host machine. It uses a daemon engine and offers a REST API and a CLI to issue commands to the containers.
Follow the instructions at these locations for your OS:
- Windows: https://docs.docker.com/desktop/windows/install/
- Mac: https://docs.docker.com/desktop/mac/install/
- Linux: https://docs.docker.com/desktop/linux/install/

Verify your installation by running:
```cmd
docker run --rm hello-world
```

## Enabling Kubernetes
You will use a local single-node Kubernetes cluster in the labs before venturing off into a cloud hosted Kubernetes. You can run either Minikube or use Docker Dekstop to run a local cluster.

## Run Kubernetes in Docker Desktop
You can enable Kubernetes in the settings of Docker Desktop. Open the Docker Desktop UI by double-clicking on the tray icon for Docker Desktop:

<img src="https://user-images.githubusercontent.com/5504642/173639225-150d5946-c697-4e85-bebf-e428399a9184.png" width="300" />

Alternatively, you can start Docker Desktop again to open the management UI.

<img src="https://user-images.githubusercontent.com/5504642/226197793-b54ccd79-9383-48c7-9558-5ce4108414f1.png" width="600" />

## Alternative: install Minikube
Minikube is a standalone version for single-node Kubernetes clusters in a development scenario. You do not need it if you have enabled the Docker Desktop Kubernetes cluster in the previous steps. If you choose not to use Docker Desktop or the Kubernetes cluster it provides, you should install Minikube to be able to work with Kubernetes on your development machine. 

Follow the instructions at https://minikube.sigs.k8s.io/docs/start/ to install Minikube on your machine. Remember to start the Minikube installation using the `start` command:
```cmd
minikube start
```

## Installing Kubernetes CLI
Kubernetes uses an command-line interface to interact with the cluster. It is called `kubectl`, pronounced "kube-cuttle" and you can download and install it from this link: https://kubernetes.io/docs/tasks/tools/

If you have Chocolatey on your Windows machine, it is as simple as `choco install kubernetes-cli`. Installing Chocolatey is straightforward and you can find the instructions here: https://chocolatey.org/install

## Cloning lab files
The workshop uses an existing .NET application and has some resource files you need during the labs. 
You will need to clone the Git repository found at https://github.com/XpiritCommunityEvents/DaprWorkshop.
Create a place where you want to store the files, such as `C:\Sources\Workshops` or `~/workshops`. In your terminal window, make sure you are in the correct folder and execute the Git clone command:

```cmd
git clone https://github.com/XpiritCommunityEvents/DaprWorkshop.git
```
You should get a subdirectory call `DaprWorkshop` inside your source folder. Go ahead and take a look at the contents. 

# Running from GitHub Codespaces
You can use GitHub Codespaces to run your development machine from the cloud. This way you do not have to setup anything on your development machine other than a modern browser. 

Go to https://github.com/XpiritCommunityEvents/DaprWorkshop to find the repository for the Dapr workshop.
On this page you should find a drop down with '<> Code' on it. Select the Codespaces tab and create a new codespace by pressing the Create button. 

<img src="https://user-images.githubusercontent.com/5504642/226197931-de04eb36-44d2-4f84-b375-6069dd864d25.png" width="300" />

If you want more control over the options you can open the menu by clicking the elipsis at the right of the dialog. Choose `New with options...` from the menu. 

<img src="https://user-images.githubusercontent.com/5504642/226198163-f49a6ad9-6d59-4b50-b974-886445a8208c.png" width="300" />

A new dialog appears that allows you to specify details of your codespace.

<img src="https://user-images.githubusercontent.com/5504642/226198586-c7367a8d-36c3-4aa8-96e8-bb424eb8f93e.png" width="700" />

Select "4-core, 8GB RAM" from the last option `Machine type` and change the region to be near to your location if needed. Click `Create codespace` to start the creation of your private development environment in the cloud.

Alternatively you can select one of the existing codespaces if you happen to return and continue work on a previous codespace session. 
You can get to manage your codespaces in more detail by following the link to `Manage All` in the first screenshot.

<img src="https://user-images.githubusercontent.com/5504642/226199099-105b49c9-3fcc-4151-922c-ba18ff04aa12.png" width="300" />

You can also remove instances, change the machine type later or export changes inside the codespace to a branch.

<img src="https://user-images.githubusercontent.com/5504642/226198978-d6ded209-51ef-433b-8be7-29b056436c58.png" width="800" />

## Using Minikube inside Codespaces
The labs in this workshop can be performed on your own laptop or inside a GitHub Codespaces. There is one additional preparation step needed in the case of running Minikube inside a codespace.

If you want to prepare ahead of time, you can execute this statement. In a later lab you will learn more about the details.

```cmd
eval $(minikube docker-env) # for WSL or bash terminal
& minikube -p minikube docker-env --shell powershell | Invoke-Expression # for PowerShell 
```

# Install Dapr
Regardless of your setup, you will need to install the Dapr CLI and initialize the runtime. Dapr is a runtime that is installed through the command-line interface. You can download the CLI installation files from https://docs.dapr.io/getting-started/install-dapr-cli/

<img src="https://user-images.githubusercontent.com/5504642/173639631-00402a9a-0c86-4e07-a10a-7138689a0fc7.png" width="300" />

Install the Dapr CLI and check that it works correctly by typing `dapr` in a terminal window.

<img src="https://user-images.githubusercontent.com/5504642/226199771-bb6b4176-c0f8-43a0-94a4-74cbb62d57fc.png" width="600" />

If the Dapr CLI installation is successful, you can install the Dapr runtime by executing: 

```cmd
dapr init
```
<img src="https://user-images.githubusercontent.com/5504642/226201810-096f0dff-afe2-4f6f-995d-cbaadcfdd403.png" width="700" />

# For the adventurous
You might already be very familiar with Docker and Kubernetes. If that is the case you can choose to follow the labs with something a little less mainstream. 

## Podman as Docker Desktop replacement
If you want to try something other than Docker Desktop to manage containers you can also use Podman. Follow the instructions at https://podman.io/getting-started/installation to install the daemonless alternative to Docker. Podman also offers pods instead of only containers. It resembles pods from Kubernetes and allows having multiple containers inside a pod.

## Install .NET preview version (for the adventurous)
You can also choose to install a preview version of the next .NET in case you want to experiment. You can find the latest releases, including the upcoming ones at: https://dotnet.microsoft.com/en-us/download/dotnet

Download the appropriate version and install it on your machine. In the labs you will have to change the version of the SDK in the `.csproj` files by yourself. Also, the NuGet packages will need to be upgraded to the versions corresponding to the SDK used.

<img src="https://user-images.githubusercontent.com/5504642/226199217-8888de9c-8c6a-4203-af85-3d3194f22205.png" width="400" />
