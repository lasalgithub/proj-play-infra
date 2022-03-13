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