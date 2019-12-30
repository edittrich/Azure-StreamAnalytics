# Stream processing with Azure Stream Analytics

## Links
https://docs.microsoft.com/de-de/azure/architecture/reference-architectures/data/stream-processing-stream-analytics  
https://github.com/mspnp/azure-stream-analytics-data-pipeline  
https://github.com/edittrich/Azure-StreamAnalytics  

https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935  

## Customization

```bash
# Login to Azure
az login
az account set --subscription edittrich

# Create project
cd /d/Documents/Workspaces/Git/Azure/
git clone git@github.com:edittrich/Azure-StreamAnalytics

# Export environment variables
export resourceGroup='rg-data-dev-001'
export resourceLocation='westeurope'
export cosmosDatabaseAccount='cosdbac-data-dev'
export cosmosDatabase='cosdb-data-dev'
export cosmosDataBaseCollection='cosdbcl-data-dev'
export eventHubNamespace='evhns-data-dev'

# Create a resource group
az group create --name $resourceGroup --location $resourceLocation

# Deploy resources
cd /d/Documents/Workspaces/Git/Azure/Azure-StreamAnalytics
az group deployment create --resource-group $resourceGroup \
--template-file ./azure/deployresources.json --parameters \
eventHubNamespace=$eventHubNamespace \
outputCosmosDatabaseAccount=$cosmosDatabaseAccount \
outputCosmosDatabase=$cosmosDatabase \
outputCosmosDatabaseCollection=$cosmosDataBaseCollection

# Create a database
az cosmosdb database create --name $cosmosDatabaseAccount \
    --db-name $cosmosDatabase \
    --resource-group $resourceGroup

# Create a collection
az cosmosdb collection create --collection-name $cosmosDataBaseCollection \
    --name $cosmosDatabaseAccount \
    --db-name $cosmosDatabase \
    --resource-group $resourceGroup

# Start Stream Analytics job

# Build dataloader
cd /d/Documents/Workspaces/Git/Azure/Azure-StreamAnalytics/onprem
docker build --no-cache -t dataloader .

# RIDE_EVENT_HUB
az eventhubs eventhub authorization-rule keys list \
    --eventhub-name taxi-ride \
    --name taxi-ride-asa-access-policy \
    --namespace-name $eventHubNamespace \
    --resource-group $resourceGroup \
    --query primaryConnectionString

# FARE_EVENT_HUB
az eventhubs eventhub authorization-rule keys list \
    --eventhub-name taxi-fare \
    --name taxi-fare-asa-access-policy \
    --namespace-name $eventHubNamespace \
    --resource-group $resourceGroup \
    --query primaryConnectionString

# Customize keys in main.env
cd /d/Documents/Workspaces/Git/Azure/Azure-StreamAnalytics/onprem
nano main.env

# Build dataloader
cd /d/Documents/Workspaces/Git/Azure/Azure-StreamAnalytics/onprem
docker build --no-cache -t dataloader .

# Run dataloader
mkdir /d/Documents/Workspaces/Git/Azure/Azure-StreamAnalytics/DataFile
unzip /d/Documents/Workspaces/Git/Azure/FOIL2013.zip -d /d/Documents/Workspaces/Git/Azure/Azure-StreamAnalytics/DataFile

docker run -v d:/Documents/Workspaces/Git/Azure/Azure-StreamAnalytics/DataFile:/DataFile --env-file=main.env dataloader:latest

docker ps
docker stop <CONTAINER ID>

# Delete resources
az group delete --name $resourceGroup
```
