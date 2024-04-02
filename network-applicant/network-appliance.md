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

# Route traffic through the NVA
created the network virtual appliance (NVA) and virtual machines (VMs),  route the traffic through the NVA.
![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/c1145a1a-9e26-410d-a10a-22e28456afef)

Create public and private virtual machines
The next steps deploy a VM into the public and private subnets.

Open the Cloud Shell editor and create a file named cloud-init.txt.
#cloud-config
package_upgrade: true
packages:
   - inetutils-traceroute

command to create the public VM
az vm create --resource-group <> --name public --vnet-name vnet --subnet publicsubnet --image Ubuntu2204 --admin-username azureuser --no-wait --custom-data cloud-init.txt --admin-password <password>

command to create the private VM:
az vm create --resource-group learn-7e93092f-6b17-4214-bce8-75e36dc27b96 --name private --vnet-name vnet --subnet privatesubnet --image Ubuntu2204 --admin-username azureuser --no-wait --custom-data cloud-init.txt --admin-password <password>

Run the following Linux watch command to check that the VMs are running. The watch command periodically runs the az vm list command so I can monitor the progress of the VMs.

Output:
Name     ProvisioningState    PowerState
-------  -------------------  ------------
nva      Succeeded            VM running
private  Succeeded            VM running
public   Succeeded            VM running

command to save the public IP address of the public VM to a variable named PUBLICIP:
PUBLICIP="$(az vm list-ip-addresses --resource-group learn-7e93092f-6b17-4214-bce8-75e36dc27b96 --name public --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" --output tsv)"

echo $PUBLICIP
command to save the public IP address of the private VM to a variable named PRIVATEIP:
PRIVATEIP="$(az vm list-ip-addresses --resource-group learn-7e93092f-6b17-4214-bce8-75e36dc27b96 --name private --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" --output tsv)"
13.88.158.81
echo $PRIVATEIP

## Test traffic routing through the network virtual appliance

The final steps use the Linux traceroute utility to show how traffic is routed. You'll use the ssh command to run traceroute on each VM. The first test will show the route taken by ICMP packets sent from the public VM to the private VM. The second test will show the route taken by ICMP packets sent from the private VM to the public VM.

command to trace the route from public to private. 
ssh -t -o StrictHostKeyChecking=no azureuser@$PUBLICIP 'traceroute private --type=icmp; exit'

output:
traceroute to private.wthvslpvcihehgmkwayb4p1f4g.dx.internal.cloudapp.net (10.0.1.4), 64 hops max
  1   10.0.2.4  1.005ms  0.646ms  0.580ms 
  2   10.0.1.4  1.361ms  1.126ms  1.150ms 
Connection to 13.88.191.35 closed.


Notice that the first hop is to 10.0.2.4. This address is the private IP address of nva. The second hop is to 10.0.1.4, the address of private. In the first exercise, you added this route to the route table and linked the table to the publicsubnet subnet. So now all traffic from public to private is routed through the NVA.


![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/6ca33841-173c-416d-9874-cc40e396fd3f)

command to trace the route from private to public
ssh -t -o StrictHostKeyChecking=no azureuser@$PRIVATEIP 'traceroute public --type=icmp; exit'

the traffic goes directly to public (10.0.0.4) and not through the NVA, as shown in the following command output.

traceroute to public.wthvslpvcihehgmkwayb4p1f4g.dx.internal.cloudapp.net (10.0.0.4), 64 hops max
  1   10.0.0.4  2.148ms  1.974ms  1.162ms 
Connection to 13.88.158.81 closed.

The private VM is using default routes, and traffic is routed directly between the subnets.

![image](https://github.com/ZCHAnalytics/virtual-networking/assets/146954022/137a9407-a01c-4b2a-b0fb-00202138903b)


