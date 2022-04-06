# Play Infra
Play Hub infrastructure components

## Add the git hub package source
```s
OWNER=playhuborg
GH_PAT=[GitHubToken]

dotnet nuget add source --username USERNAME --password $GH_PAT --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$OWNER/index.json"
```

## Azure login and setting subscription
```s
az account set --subscription ".NET Microservices"
az account show
```

## Creating the Azure resource group
```s
APP_NAME="playhub"
az group create --name $APP_NAME --location eastus
```

## Creating the Cosmos Db account
```s
az cosmosdb create --name $APP_NAME --resource-group $APP_NAME --kind MongoDB --enable-free-tier
```

## Creating Azure Service Bus namespace
```s
az servicebus namespace create --name $APP_NAME --resource-group $APP_NAME --sku Standard
```

# Catalog MS

Catalog MS infrastructure. This includes all the commands and information needed to run application locally as well as deploy into Azure Kubernates Service (AKS)

# Run Application Locally

Please follow the below steps to run the application locally. **All the commands mentioned are relevant to Windows Os**. If you are using another Os(Mac/ Linux), minor adjustments are needed to the mentioned commands.

## Prerequisites

- .NET 5 SDK (https://dotnet.microsoft.com/en-us/download/dotnet/5.0)
- Docker Desktop (https://docs.docker.com/get-docker/)
- VS Code (https://aka.ms/vscode - Only needed if you are going to run API or Angular App via IDE)
- NodeJs (https://nodejs.org/en/download/)

## Add Github package source

Need to obtain a **Personal Access Token** from github with write/read packages access.

```powershell
$owner="playhub"
$github_personal_access_token="[PAT]"

dotnet nuget add source --username USRNAME --password $github_personal_access_token --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## List nuget sources

```powershell
dotnet nuget list source
```

## Setup Mongo Container

In the application, we are using Mongo DB. Mongo DB container can be run locally using dcoker. Please use the provided docker-compose.yml for setting up the container. For simplicity authentication is not used, but for proper usage, need to setup authentication and authorization.

Go to docker-compose.yml file location and run the below command

```poweshell
docker-compose up -d
```

## Run Catalog MS using .NET CLI

The .NET CLI is included with the .NET SDK. Clone the project into your local folder. Go to project root folder(Play.Catalog.csproj location path). Run below command to run the application.

```powershell
dotnet run
```

App can be accessed using below urls

https://localhost:5001/

http://localhost:5000/


## Run Catalog MS using VS Code (If not using .NET CLI)

Application can be run in VS Code too as below


# Deploy Catalog MS Application In Azure K8s Service

## Add Github package source

```powershell
$commonversion="2.0.0"
$owner="playhub"
$github_personal_access_token="[PAT]"

dotnet nuget add source --username USRNAME --password $github_personal_access_token --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Publish Play.Common nuget to github packages

```powershell

dotnet pack src\Play.Common\ --configuration Release -p:PackageVersion=$commonversion -p:RepositoryUrl=https://github.com/$owner/Play.Common -o ..\packages

dotnet nuget push ..\packages\Play.Common.$commonversion.nupkg --api-key $github_personal_access_token --source "github"

```

## Build Play.Catalog Docker Image

```powershell
$env:GH_OWNER="playhub"
$env:GH_PAT="[GH PAT]"
$apiversion="1.0.3"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t play.catalog:$apiversion .
```

Tag to ACR: 
```powershell
docker build --secret id=GH_OWNER --secret id=GH_PAT -t "$appname.azurecr.io/playhub.api:$apiversion" .
```

## Run Catalog MS docker image locally

```powershell
$apiversion="1.0.2"
docker run -it --rm -p 5000:5000 --name catalog -e MongoDbSettings__Host=mongo --network playinfra_default play.catalog:$apiversion
```

## Install Azure CLI

We will be connecting to Azure portal through Azure CLI. Azure CLI can be installed from below link

https://docs.microsoft.com/en-gb/cli/azure/install-azure-cli-windows?tabs=azure-cli


## Create Azure resource group

```powershell
$appname="playhub"
az group create --name $appname --location eastus
```

## Create Cosmos DB Account

```powershell
az cosmosdb create --name $appname --resource-group $appname --kind MongoDB --enable-free-tier
```

## Update Play.Common to connect to Cosmos DB

Update Play.Common library(Version - 1.0.3) to use passed connection string through docker run command(via env variables)

## Create new Catalog MS Image

Rebuild Play.Catalog (1.0.1) image to support Play.Common 1.0.3 Library

## Run the Catalog MS docker image with Cosmos DB

```powershell
$cosmosDBConnectionString="[CONN STRING]"
docker run -it --rm -p 5000:5000 --name catalog -e MongoDbSettings__ConnectionString=$cosmosDBConnectionString --network playinfra_default play.catalog:$apiversion
```

## Create Azure Container Registry

```powershell
az acr create --name $appname --resource-group $appname --sku Basic
```


## Publish Play.Catalog Docker Image to ACR

```powershell
$appname="playhub"
$apiversion="1.0.1"
az acr login --name $appname
docker tag play.catalog:$apiversion "$appname.azurecr.io/playhub.api:$apiversion"
docker push "$appname.azurecr.io/playhub.api:$apiversion"
```

## Creating AKS Cluster

```powershell
az feature register --name EnablePodIdentityPreview --namespace Microsoft.ContainerService
az extension add --name aks-preview

az aks create -n $appname -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $appname --enable-pod-identity --network-plugin azure

az aks get-credentials --name playhub --resource-group playhub
```

## Create K8s namespace

```powershell
$namespace="playhub"
kubectl create namespace $namespace
```

## Create K8s secret

```powershell
kubectl create secret generic playhub-secret --from-literal=cosmosdb-connectionstring=$cosmosDBConnectionString -n $namespace
```

## Create K8s pod

```powershell
kubectl apply -f .\Kubernetes\play.catalog.yaml  -n $namespace
kubectl get pods -n $namespace
kubectl logs {podid}
kubectl describe pod {podid} -n $namespace
```

Sameway a service can be added with type of LoadBalancer and the services can be accessed externally( Not recomended - Just for testing purpose)

## Test created service

```powershell
kubectl get services -n $namespace
```

