# Create Hub-Spoke Virtual Networks & Peerings
```bash
rgName=rg-firewall
location=canadacentral
az group create -n $rgName -l $location
# Hub Firewall VNet
vnetName=vn-hub-firewall
vnetAddressSpace=10.0.0.0/16
subnetName="AzureFirewallSubnet" # this you cannot change
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