# Create custom route

https://learn.microsoft.com/en-us/training/modules/control-network-traffic-flow-with-routes/media/3-virtual-network-subnets-route-table.svg


1. In Azure Cloud Shell, create a route table: 
$$ az network route-table create --name publictable --resource-group [resource group name] --disable-bgp-route-propagation false

2. 
az network route-table route create --route-table-name publictable --resource-group [resource group name] --name productionsubnet --address-prefix 10.0.1.0/24 --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.2.4

Create the vnet virtual network and the three subnets you need: publicsubnet, privatesubnet, and dmzsubnet.
command to create the vnet virtual network and the publicsubnet subnet:
az network vnet create --name vnet --resource-group [resource group name] --address-prefixes 10.0.0.0/16 --subnet-name publicsubnet --subnet-prefixes 10.0.0.0/24

command in Cloud Shell to create the privatesubnet subnet:
az network vnet subnet create --name privatesubnet --vnet-name vnet --resource-group [resource group name] --address-prefixes 10.0.1.0/24

 command to create the dmzsubnet subnet.
az network vnet subnet create --name dmzsubnet --vnet-name vnet --resource-group [sandbox resource group name] --address-prefixes 10.0.2.0/24


 command to show all of the subnets in the vnet virtual network.
az network vnet subnet list --resource-group [resource group name] --vnet-name vnet --output table

AddressPrefix    Name           PrivateEndpointNetworkPolicies    PrivateLinkServiceNetworkPolicies    ProvisioningState    ResourceGroup
---------------  -------------  --------------------------------  -----------------------------------  -------------------  ------------------------------------------
10.0.0.0/24      publicsubnet   Disabled                          Enabled                              Succeeded            learn-7e93092f-6b17-4214-bce8-75e36dc27b96
10.0.1.0/24      privatesubnet  Disabled                          Enabled                              Succeeded            learn-7e93092f-6b17-4214-bce8-75e36dc27b96
10.0.2.0/24      dmzsubnet      Disabled                          Enabled                              Succeeded            learn-7e93092f-6b17-4214-bce8-75e36dc27b96

Associate the route table with the public subnet
command to associate the route table with the public subne
az network vnet subnet update --name publicsubnet --vnet-name vnet --resource-group [sandbox resource group name] --route-table publictable
