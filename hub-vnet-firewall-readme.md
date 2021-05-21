# Create Firewall & configure Service Endpoint in Hub (Firewall) VNet

## Create Azure Firewall
```bash
rgName=rg-firewall
fwTag="Identifier=Firewall-SEP"
location=canadacentral
# Create a public ip for Azure Firewall
pipName=pip-fw-sep
az network public-ip create -g $rgName -n $pipName --sku Standard --tags $fwTag --allocation-method Static --verbose
# Create Firewall
fwName=fw-sep
az network firewall create --name $fwName -g $rgName --location $location --tags $fwTag --verbose
```
The above command will create the Firewall resource but it won't have it associated with the Hub VNet.

![Alt text](/images/fw-without-ipconfig.png)

Associate the IP Configuration with Hub VNet.
```bash
# Create Azure Firewall IP configuration
vnetName=vn-hub-firewall
ipconfigName=ipc-fw-sep
az network firewall ip-config create --name $ipconfigName --firewall-name $fwName --public-ip-address $pipName --vnet-name $vnetName -g $rgName --debug
# Get firewall's private IP Address
fwPrivateIP=$(az network firewall show -g $rgName -n $fwName --query "ipConfigurations[0].privateIpAddress" -o tsv)
```

## Configure Log Analytics workspace for Azure Firewall logs
```bash
# Create Log Analytics workspace
laName=la-fw-sep
az monitor log-analytics workspace create -g $rgName --workspace-name $laName --location $location --tags $fwTag --verbose
# Create diagnostic settings, forward the logs & events to log analytics workspace 
dsName=ds-fw-sep
monitoringResourceId=$(az network firewall show -g $rgName -n $fwName | jq -r ".id")
lawsId=$(az monitor log-analytics workspace show -g $rgName -n $laName | jq -r ".id")
az monitor diagnostic-settings create -n $dsName --resource $monitoringResourceId --workspace $lawsId --logs '[{"category":"AzureFirewallApplicationRule","Enabled":true}, {"category":"AzureFirewallNetworkRule","Enabled":true}]' --metrics '[{"category": "AllMetrics","Enabled": true}]' --verbose
``` 
The last command will create the diagnostic settings with the below configurations.

![Alt text](/images/diagnostics-firewall.png)

## Clean Up
```bash
# List resources associated with tags Identifier=Firewall-SEP
az resource list --tag Identifier=Firewall-SEP -o table
# Delete resources associated with Firewall
resources=$(az resource list --tag Identifier=Firewall-SEP -o table --query "[].id" -o tsv)
# Loop through the resources & delete
for resource in $resources
do
 echo "Delete resource $resource"
 az resource delete --ids $resource --debug
done
```

