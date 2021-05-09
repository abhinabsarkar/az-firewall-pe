# Azure Firewall & Networking using Private Endpoint
![Alt text](/images/hub-and-spoke.png)

![Alt text](/images/azure-network-firewall-pep-sep.png)

* [Create Hub-Spoke VNet & Peerings](vnet-readme.md)
* [Configure V-Link, DNS Zone. Create Private Endpoint in Spoke (PEP) VNet ](spoke-vnet-pep-readme.md)
* [Configure Spoke VNet for App](spoke-vnet-app-readme.md)
* [Configure Hub (Firewall) VNet]()

## References
* [Azure Firewall to inspect traffic destined to a private endpoint](https://docs.microsoft.com/en-us/azure/private-link/inspect-traffic-with-azure-firewall)
* [Azure Private DNS Zone](https://docs.microsoft.com/en-us/azure/dns/private-dns-overview)
* [V-Link/VNet Link - Virtual Network Link](https://docs.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links)
* [Integrate Key vault with Azure Private Link](https://docs.microsoft.com/en-us/azure/key-vault/general/private-link-service)
* [Azure Private Endpoint](https://docs.microsoft.com/en-us/azure/private-link/private-endpoint-overview)
* [JQ cookbook](https://github.com/stedolan/jq/wiki/Cookbook#filter-objects-based-on-the-contents-of-a-key)
* [Azure firewall with service endpoints](https://docs.microsoft.com/en-us/azure/firewall/firewall-faq#how-do-i-set-up-azure-firewall-with-my-service-endpoints)
* [Update Service Endpoint in a Subnet](https://docs.microsoft.com/en-us/cli/azure/network/vnet/subnet?view=azure-cli-latest#az_network_vnet_subnet_update)