# Stream processing with Azure Stream Analytics

## Links
https://docs.microsoft.com/de-de/azure/architecture/reference-architectures/data/stream-processing-stream-analytics  
https://github.com/mspnp/azure-stream-analytics-data-pipeline  

https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935  

## Customization
https://github.com/edittrich/Azure-StreamAnalytics  

```bash
# Login to Azure
az login
az account set --subscription edittrich

# Create project
export PRJROOT=/d/Documents/Workspaces/Git/Azure
export DWNDIR=/d/Documents/Workspaces/Projects/Azure
export PRJROOT=/home/edittrich/Documents/workspaces/git/azure
export DWNDIR=/home/edittrich/Downloads

export PRJDIR=$PRJROOT/azure-streamanalytics

mkdir -p $PRJROOT
cd $PRJROOT
git clone git@github.com:edittrich/azure-streamanalytics

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
cd $PRJDIR
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

# Customize connection string
az eventhubs eventhub authorization-rule keys list \
    --eventhub-name taxi-ride \
    --name taxi-ride-asa-access-policy \
    --namespace-name $eventHubNamespace \
    --resource-group $resourceGroup \
    --query primaryConnectionString

az eventhubs eventhub authorization-rule keys list \
    --eventhub-name taxi-fare \
    --name taxi-fare-asa-access-policy \
    --namespace-name $eventHubNamespace \
    --resource-group $resourceGroup \
    --query primaryConnectionString

cp $PRJDIR/onprem/main-template.env $PRJDIR/onprem/main.env
nano $PRJDIR/onprem/main.env

# Build dataloader
cd $PRJDIR/onprem
docker build --no-cache -t dataloader .

mkdir $PRJDIR/DataFile
unzip $DWNDIR/FOIL2013.zip -d $PRJDIR/DataFile

# Run dataloader
cd $PRJDIR/onprem
docker run -v $PRJDIR/DataFile:/DataFile --env-file=main.env dataloader:latest

docker ps
docker stop <CONTAINER ID>

# Delete resources
az group delete --name $resourceGroup
```
