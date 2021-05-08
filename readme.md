# Azure Firewall & Networking using Private Endpoint
![Alt text](/images/hub-and-spoke.png)

![Alt text](/images/azure-network-firewall-pep-sep.png)

* [Create Hub-Spoke VNet & Peerings](vnet-readme.md)
* [Create Private Endpoint, V-Link, DNS Zone](pep-readme.md)
* [Configure Spoke VNet for App](spoke-vnet-app-readme.md)

## Enable Service Endpoint on Azure Firewall Hub's subnet
```bash
# Get the list of Service Endpoint available in a location
az network service-endpoint list -l $location -o table
# Get the name of type KeyVault from the service endpoint list. Note that "contains" is case-sensitive
sepKeyVaultName=$(az network service-endpoint list -l $location | jq -r '.[] | select(.name | contains("Key")) | .name')
# Hub Firewall VNet 
vnetName=vn-hub-firewall
subnetName=sn-firewall
# Enable service endpoint for the Subnet on Hub (Firewall) VNet 
az network vnet subnet update -n $subnetName --vnet-name $vnetName --service-endpoints $sepKeyVaultName -g $rgName
```

## References
* [Azure Firewall to inspect traffic destined to a private endpoint](https://docs.microsoft.com/en-us/azure/private-link/inspect-traffic-with-azure-firewall)
* [Azure Private DNS Zone](https://docs.microsoft.com/en-us/azure/dns/private-dns-overview)
* [V-Link/VNet Link - Virtual Network Link](https://docs.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links)
* [Integrate Key vault with Azure Private Link](https://docs.microsoft.com/en-us/azure/key-vault/general/private-link-service)
* [Azure Private Endpoint](https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview)
* [JQ cookbook](https://github.com/stedolan/jq/wiki/Cookbook#filter-objects-based-on-the-contents-of-a-key)
* [Azure firewall with service endpoints](https://docs.microsoft.com/en-us/azure/firewall/firewall-faq#how-do-i-set-up-azure-firewall-with-my-service-endpoints)
* [Update Service Endpoint in a Subnet](https://docs.microsoft.com/en-us/cli/azure/network/vnet/subnet?view=azure-cli-latest#az_network_vnet_subnet_update)