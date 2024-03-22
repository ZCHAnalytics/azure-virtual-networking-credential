Create an NVA and virtual machines  to secure and monitor traffic between front-end public servers and internal private servers.

In this stage of security implementation, deploy a network virtual appliance (NVA) to secure and monitor traffic between your front-end public servers and internal private servers.

configure the appliance to forward IP traffic. If IP forwarding isn't enabled, traffic that is routed through your appliance will never be received by its intended destination servers.

deploy the nva network appliance to the dmzsubnet subnet. enable IP forwarding so that traffic from * and traffic that uses the custom route is sent to the privatesubnet subnet.

![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/5b958f41-c153-402d-9f4a-8f4a049c38f7)

![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/81b9ac58-9fe9-4e4c-8f45-720b6eb2c244)

## Deploy the network virtual appliance
command to deploy the appliance:
az vm create --resource-group <> --name nva --vnet-name vnet --subnet dmzsubnet --image Ubuntu2204 --admin-username azureuser --admin-password <password>

## Enable IP forwarding for the Azure network interface
Command to get the ID of the NVA network interface:
NICID=$(az vm nic list --resource-group <> --vm-name nva --query "[].{id:id}" --output tsv)
echo $NICID

Command to get the name of the NVA network interface:
NICNAME=$(az vm nic show --resource-group <> --vm-name nva --nic $NICID --query "{name:name}" --output tsv)
echo $NICNAME

output: nvaVMNic

Command to enable IP forwarding for the network interface:
az network nic update --name $NICNAME --resource-group <> --ip-forwarding true

## Enable IP forwarding in the appliance
Command to save the public IP address of the NVA virtual machine to the variable NVAIP:
NVAIP="$(az vm list-ip-addresses --resource-group <> --name nva --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" --output tsv)"
echo $NVAIP

output: 13.64.79.33

Command to enable IP forwarding within the NVA:
$$bash: ssh -t -o StrictHostKeyChecking=no azureuser@$NVAIP 'sudo sysctl -w net.ipv4.ip_forward=1; exit;'

Output:
Warning: Permanently added '13.64.79.33' (ED25519) to the list of known hosts.
azureuser@13.64.79.33's password: 
net.ipv4.ip_forward = 1
Connection to 13.64.79.33 closed.
