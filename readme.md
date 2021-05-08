# Azure Firewall & Networking using Private Endpoint
![Alt text](/images/hub-and-spoke.png)
## Create Virtual Networks & Peerings
```bash
rgName=rg-firewall
location=canadacentral
az group create -n $rgName -l $location
# Hub Firewall VNet
vnetName=vn-hub-firewall
vnetAddressSpace=10.0.0.0/16
subnetName=sn-firewall
subnetAddressSpace=10.0.0.0/24
# Spoke VM (App) VNet
vnetName=vn-spoke-app
vnetAddressSpace=10.1.0.0/16
subnetName=sn-app
subnetAddressSpace=10.1.0.0/24
# Spoke Private Endpoint VNet
vnetName=vn-spoke-pep
vnetAddressSpace=10.2.0.0/16
subnetName=sn-pep
subnetAddressSpace=10.2.0.0/24
# Create VNets for the three address space 
echo "Create Virtual Network"
az network vnet create \
   -g $rgName -n $vnetName \
   --address-prefix $vnetAddressSpace \
   --subnet-name $subnetName \
   --subnet-prefix $subnetAddressSpace \
   --verbose
```
```bash
# Create VNet peering between Hub Firewall VNet & Spoke App VNet
vnetPeer=peer-firewallHub-appSpoke
vnetName=vn-hub-firewall
remoteVnetName=vn-spoke-app
# To successfully peer two virtual networks this command must be called twice with the values for --vnet-name and --remote-vnet reversed.
az network vnet peering create -g $rgName -n $vnetPeer --vnet-name $vnetName \
    --remote-vnet $remoteVnetName --allow-vnet-access --verbose
# Run the peering create command again with values reversed
vnetPeer=peer-appSpoke-firewallHub
vnetName=vn-spoke-app
remoteVnetName=vn-hub-firewall
az network vnet peering create -g $rgName -n $vnetPeer --vnet-name $vnetName \
    --remote-vnet $remoteVnetName --allow-vnet-access --verbose

# Create VNet peering between Hub Firewall VNet & Spoke Private Endpoint VNet
vnetPeer=peer-firewallHub-pepSpoke
vnetName=vn-hub-firewall
remoteVnetName=vn-spoke-pep
# To successfully peer two virtual networks this command must be called twice with the values for --vnet-name and --remote-vnet reversed.
az network vnet peering create -g $rgName -n $vnetPeer --vnet-name $vnetName \
    --remote-vnet $remoteVnetName --allow-vnet-access --verbose
# Run the peering create command again with values reversed
vnetPeer=peer-pepSpoke-firewallHub
vnetName=vn-spoke-pep
remoteVnetName=vn-hub-firewall
az network vnet peering create -g $rgName -n $vnetPeer --vnet-name $vnetName \
    --remote-vnet $remoteVnetName --allow-vnet-access --verbose
```
## Create VM in the Spoke VNet for App
```bash
# Create a VM in azure and associate it with Spoke App network
vmName=vm-win10
# Get a windows 10 image
echo "Get the windows 10 image urn"
windows10=$(az vm image list --publisher MicrosoftWindowsDesktop --offer Windows-10 --sku 19h2-pro --all --query "[0].urn" -o tsv)
echo $windows10

# Spoke VM (App) VNet
vnetName=vn-spoke-app
subnetName=sn-app

# Parameter required - admin-username & password
echo "Create VM and attach it to the existing Virtual Network"
az vm create --name $vmName --resource-group $rgName \
	--admin-username $1 --admin-password $2 \
	--image $windows10 \
	--size Standard_D4_v3 \
	--location $location \
	--vnet-name $vnetName \
	--subnet $subnetName \
	--verbose
```
# Create Private Endpoint for Key Vault
```bash
# Create a Key Vault
kvName=kv-pep-spokePep
az keyvault create -n $kvName -g $rgName -l $location --no-wait --verbose
# Turn on Key Vault Firewall
az keyvault update -n $kvName -g $rgName --default-action deny --verbose

# Network policies like network security groups (NSG) are not supported for private endpoints. In order to deploy a Private Endpoint on a given subnet, an explicit disable setting is required on that subnet. This setting is only applicable for the Private Endpoint. For other resources in the subnet, access is controlled based on Network Security Groups (NSG) security rules definition. When using the portal to create a private endpoint, this setting is automatically disabled as part of the create process.

# Disable Virtual Network Policies for the subnet in Spoke Private Endpoint VNet
vnetName=vn-spoke-pep
subnetName=sn-pep
az network vnet subnet update --name $subnetName --vnet-name $vnetName -g $rgName --disable-private-endpoint-network-policies true
```
## Azure Private DNS Zone
The Domain Name System, or DNS, is responsible for translating (or resolving) a service name to an IP address. Azure DNS is a hosting service for domains and provides naming resolution using the Microsoft Azure infrastructure. Azure DNS not only supports internet-facing DNS domains, but it also supports private DNS zones.

To resolve the records of a private DNS zone from your virtual network, you must link the virtual network with the zone. Linked virtual networks have full access and can resolve all DNS records published in the private zone. You can also enable autoregistration on a virtual network link. When you enable autoregistration on a virtual network link, the DNS records for the virtual machines in that virtual network are registered in the private zone. When autoregistration gets enabled, Azure DNS will update the zone record whenever a virtual machine gets created, changes its' IP address, or gets deleted.

![Alt text](/images/private-dns.png)

### Azure Private Endpoint DNS configuration
The FQDN of the services resolves automatically to a public IP address. To resolve to the private IP address of the private endpoint, change your DNS configuration to use the recommended zone names as described by Microsoft. Refer this [link](https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration)

For example, the Azure Key Vault private link resource type must have the private DNS zone name as "privatelink.vaultcore.azure.net". 

This means if you run the ns lookup \<key-vault-name>.vault.azure.net to resolve the IP address of your key vault over a public endpoint, you will see a result that looks like this:

```cmd
nslookup kv-pep-spokepep.vault.azure.net
Server:  UnKnown
Address:  168.63.129.16

Non-authoritative answer:
Name:    azkms-prod-cca-2-a.cloudapp.net
Address:  20.38.149.196
Aliases:  kv-pep-spokepep.vault.azure.net
          kv-pep-spokepep.privatelink.vaultcore.azure.net
          data-prod-cca.vaultcore.azure.net
          data-prod-cca-region.vaultcore.azure.net
          azkms-prod-cca-a.trafficmanager.net
```
If you run the ns lookup \<key-vault-name>.vault.azure.net to resolve the IP address of your key vault over a private endpoint
```cmd
nslookup kv-pep-spokepep.vault.azure.net
Server:  UnKnown
Address:  168.63.129.16

Non-authoritative answer:
Name:     kv-pep-spokepep.privatelink.vaultcore.azure.net
Address:  10.2.0.4
Aliases:  kv-pep-spokepep.vault.azure.net
```

```bash
# Create a Private DNS Zone for Azure Service
pdzName=privatelink.vaultcore.azure.net
az network private-dns zone create -g $rgName --name $pdzName
# For a VM, it can be any DNS name
pdzName=www.abhi-pdz.com
az network private-dns zone create -g $rgName --name $pdzName
```
## VNet Link (V-Link)
Virtual Network Link (V-Link) links a VNet to private DNS zone in Azure. Once linked, VMs hosted in that virtual network can access the private DNS zone. Every private DNS zone has a collection of virtual network link child resources. Each one of these resources represents a connection to a virtual network. A virtual network can be linked to private DNS zone as a registration or as a resolution virtual network.

### Registration virtual network
When creating a link between a private DNS zone and a virtual network, you have the option to enable autoregistration. With this setting enabled, a DNS record gets automatically created for any virtual machines you deploy in the virtual network. DNS records will also be created for virtual machines already deployed in the virtual network.

From the virtual network perspective, private DNS zone becomes the registration zone for that virtual network. A private DNS zone can have multiple registration virtual networks. However, every virtual network can only have one registration zone associated with it.

![Alt text](/images/registered-vnet.png)

### Resolution virtual network
If you choose to link your virtual network with the private DNS zone without autoregistion, the virtual network is treated as a resolution virtual network only. DNS records for virtual machines deployed in this virtual network won't be created automatically in the private zone. However, virtual machines deployed in the virtual network can successfully query for DNS records in the private zone. These records include manually created and auto registered records from other virtual networks linked to the private DNS zone.

One private DNS zone can have multiple resolution virtual networks and a virtual network can have multiple resolution zones associated to it.

![Alt text](/images/resolved-vnet.png)

```bash
# Link the Private DNS Zone to the Virtual Network
vnetName=vn-spoke-pep
pdzName=privatelink.vaultcore.azure.net
vlinkName=vl-kvPep
az network private-dns link vnet create -g $rgName --virtual-network $vnetName --zone-name $pdzName --name $vlinkName --registration-enabled true
```
## Create a Private Endpoint (Automatically Approve)
```bash
# Get the resource id for Key Vault
id=$(az keyvault show -n $kvName | jq -r ".id")
vnetName=vn-spoke-pep
subnetName=sn-pep
pepName=pep-kv
# Create private end point
az network private-endpoint create -g $rgName --vnet-name $vnetName --subnet $subnetName --name $pepName  --private-connection-resource-id $id --group-id vault --connection-name $vlinkName -l $location
```
The above command will create Private Endpoint which is a network interface that connects you privately and securely to a service powered by Azure Private Link. When a private endpoint is created, a read-only NIC with private IP in the subnet "sn-pep" is created & assigned to the Key Vault. This cannot be modified and will remain for the life cycle of the Private endpoint.

## References
* [Azure Firewall to inspect traffic destined to a private endpoint](https://docs.microsoft.com/en-us/azure/private-link/inspect-traffic-with-azure-firewall)
* [Azure Private DNS Zone](https://docs.microsoft.com/en-us/azure/dns/private-dns-overview)
* [V-Link/VNet Link - Virtual Network Link](https://docs.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links)
* [Integrate Key vault with Azure Private Link](https://docs.microsoft.com/en-us/azure/key-vault/general/private-link-service)
* [Azure Private Endpoint](https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview)