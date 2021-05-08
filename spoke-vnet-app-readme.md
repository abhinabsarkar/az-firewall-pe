# Create VM in the Spoke VNet for App
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
username=
password=
echo "Create VM and attach it to the existing Virtual Network"
az vm create --name $vmName --resource-group $rgName \
	--admin-username $username --admin-password $password \
	--image $windows10 \
	--size Standard_B1s \
	--location $location \
	--vnet-name $vnetName \
	--subnet $subnetName \
	--tags Identifier=VM-WIN10-B1s \
	--verbose

# Validate the resources created by the above command
az resource list --tag Identifier=VM-WIN10-B1s -o table

# Delete all the resources associated with the VM. The associated resources were tagged while creating the vm
echo "Delete the VM first before deleting any of the other resource"
az vm delete --name $vmName --resource-group $rgName --yes --verbose
echo "Delete the resources associated with the VM"
vmResources=$(az resource list --tag Identifier=VM-WIN10-B1s -o table --query "[].id" -o tsv)
for resource in $vmResources
do
 echo "Delete resource $resource"
 az resource delete --ids $resource --verbose
done
```