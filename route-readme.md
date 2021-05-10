# Create Route Table
## Create Route Table & route
Create a route that sends traffic from the Spoke App subnet to the address space of Spoke Private Endpoint VNet, through the Azure Firewall.
```bash
# Create Route Table
rgName=rg-firewall
location=canadacentral
rtName=appSubnet-to-azureFirewall
az network route-table create -g $rgName -l $location -n $rtName
# Create a route that sends traffic from the Spoke App subnet to the address space of Spoke Private Endpoint VNet, through the Azure Firewall
routeName=appSubnet-to-privateEndpoint
# The destination CIDR to which the route applies. In this case, it is the Spoke (PEP) VNet.
destAddressCIDR=10.2.0.0/16
# IP address of the Virtual Appliance
nextHopIPAddress=$fwPrivateIP
# Create route
az network route-table route create -g $rgName --route-table-name $rtName -n $routeName --address-prefix $destAddressCIDR --next-hop-type VirtualAppliance --next-hop-ip-address $nextHopIPAddress
# Update the Spoke App subnet with the Route table
vnetName=vn-spoke-app
subnetName=sn-app
subnetId=$(az network vnet subnet show -n $subnetName --vnet-name $vnetName -g rg-firewall | jq -r ".id")
az network vnet subnet update --route-table $rtName --ids $subnetId
```